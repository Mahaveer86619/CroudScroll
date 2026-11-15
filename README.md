# CrowdScroll

A shared video feed that a crowd fills and steers. People drop clips into a room's chat, a recommender lines up what plays next, and everyone votes Stay or Skip, reacts and chats while it plays. Everything plays inside CrowdScroll: on a Stage screen captured by OBS (Stage mode) or on every participant's screen in sync (Together mode).

Status: planning. Start with [docs/README.md](docs/README.md); milestones are in [docs/roadmap.md](docs/roadmap.md).

## Layout

```
server/            Go module (Echo API, WebSocket gateway, background worker)
  cmd/api/         HTTP + WebSocket process
  cmd/worker/      submission checks, probe, moderation, stats jobs
  internal/
    auth/          guest + OAuth sessions, roles
    room/          room lifecycle, membership, host controls
    realtime/      WebSocket gateway, per-room hub, presence, fan-out
    feed/          feed state machine, on-deck list, tallies (pure, fake clock)
    ranking/       recommendation strategies (pure, no I/O)
    submission/    chat/API submissions, inline checks
    ingest/        provider resolvers, metadata, upload transcode, dedup hashes
    probe/         client for the headless playability probe
    moderation/    moderators, review queue, reports, audit
    media/         S3/R2 object storage
    ratelimit/     Redis-backed token buckets
    store/         sqlc-generated Postgres access
  migrations/      goose SQL migrations
  queries/         sqlc query files
web/               Next.js + TypeScript (Stage, Remote, Together view, host panel, /probe page)
sidecars/probe/    Playwright + Chromium service that plays a clip to prove it works
sidecars/classifier/  ONNX Runtime image safety classifier (CPU)
deploy/            production compose file, Caddyfile, backup script
loadtest/          k6 WebSocket scenarios
docs/              design docs, indexed from docs/README.md
```

## Local development

```
docker compose -f docker-compose.dev.yml up -d   # postgres, redis, minio
```

## Contributing

`main` is protected by convention: every change goes through a pull request.
