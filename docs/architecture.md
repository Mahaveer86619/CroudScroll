# Architecture

Modular monolith: one Go module with two binaries, a Next.js app, and two small sidecars.

```
 Stage (OBS) ─┐
 Remotes ─────┼─HTTPS/WSS─> Caddy ──> web (Next.js)
 Together ────┘                  └──> api (Go, Echo)  ── Postgres (source of truth, job queue)
                                        ▲      │      └─ Redis (pub/sub, rate limits)
                                        │      ▼
                                       worker (Go) ──> probe (headless Chromium)
                                                   ──> classifier (ONNX, CPU)
                                                   ──> R2 (uploads, posters)
```

## Stack

| Concern | Choice | Why |
|---|---|---|
| HTTP | Echo | middleware for auth, rate limits |
| WebSocket | `github.com/coder/websocket` | context-aware, maintained |
| DB | pgx + sqlc, goose migrations | typed SQL, no ORM |
| Jobs | River (Postgres-backed) | enqueue in the same transaction as the submission insert |
| Pub/sub | Redis | worker → api notifications; lets rooms spread over several api instances later |
| Frontend | Next.js + TypeScript, Tailwind | Remote, Stage, Together view, host panel, probe page |
| Probe | Playwright + Chromium, driven by the worker | runs the real player to prove playability |
| Classifier | Python + ONNX Runtime | small image model on CPU |
| Media | S3 API: MinIO locally, Cloudflare R2 in prod | no egress fees |

## Server packages

```
internal/
  auth/        guest sessions, Discord/Twitch OAuth, stage tokens, roles
  room/        room lifecycle, membership, config, host commands
  realtime/    WebSocket gateway, connection send buffers, presence, clock ping
  feed/        feed state machine, on-deck list, vote/reaction tallies (pure + fake clock)
  ranking/     strategies (pure)
  submission/  chat/API submission entry, checks 1–3
  ingest/      provider resolvers, metadata, upload transcode, dedup hashes
  probe/       client for the headless player probe
  moderation/  moderators, review queue, reports, audit
  media/       object storage
  ratelimit/   Redis token buckets
  store/       sqlc output
```

`feed` and `ranking` have no I/O; the room actor wires them to the store and the gateway. That's what makes the state machine and strategies testable with a fake clock.

## Room actors

Each live room is owned by one goroutine in the api process. Votes, reactions, chat, host commands, Stage player events, timer ticks and worker notifications all arrive as messages on its channel, so room state has a single writer and no locks.

Every state change bumps a room `seq` and is broadcast. Each connection has a bounded send buffer; a client that can't keep up is disconnected and reconnects into a fresh snapshot instead of slowing the room.

Actors start on first join and stop after 10 minutes with no connections.

## Durability

- Postgres is the record. Redis holds nothing that can't be rebuilt.
- Feed transitions and chat messages are written before they're broadcast.
- Votes and reactions are upserted in 250 ms batches; a crash loses at most that window, and tallies are rebuilt from Postgres on restart.
- Client messages carry an id, so retries after a reconnect don't double-post or double-vote.

## Scaling stance

One api process on one VPS is the target. If a load test shows otherwise, rooms get pinned to api instances by room id with Redis pub/sub for cross-instance notifications. Not before.
