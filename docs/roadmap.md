# Roadmap

Each milestone ends with something demoable. Resume bullets only describe finished milestones.

## M0 — Skeleton and spikes (1–2 weeks)
- Dev compose (Postgres, Redis, MinIO), goose, sqlc, Echo with health check, Next.js shell, CI.
- `FeedPlayer` for uploads and YouTube.
- **Spikes first** (see [spikes.md](spikes.md)): provider players in OBS and on Android, then the Stage + Remote mock flow. Results fix the supported source list and the Remote layout before M1 builds them.

## M1 — Stage mode with host-added clips (2 weeks)
- Guest join, Discord/Twitch login, rooms, roles, stage token.
- Room actor, WebSocket gateway, snapshot on join/reconnect, clock sync.
- Stage page and Remote page; host adds links from the panel (host approval only, no worker yet).
- Chat.

## M2 — The crowd decides (2 weeks)
- Stay/Skip and reactions; feed state machine with thresholds, timers, skip guards, stream delay, host controls.
- On-deck list of 3 with tiered preloading on the Stage.
- Stage `player.event` handling and `play_error` recovery.
- Restart test: kill the api mid-item; the Stage and Remotes resume on the same item with the same tallies.
- State machine unit tests with a fake clock.

## M3 — Submissions, playability gate, moderation (2–3 weeks)
- Chat submissions and uploads (transcode to R2).
- River jobs: metadata, headless probe, classifier, review queue, reports.
- Private rejection feedback, duplicate boosts, rate limits, provider health.

## M4 — Recommendation (1–2 weeks)
- Stats roll-up, strategies and modes, fairness constraints, encores, "why this?" panel.
- Replay tests on recorded sessions.

## M5 — Together mode and launch (2 weeks)
- Synced playback on every client, drift correction, per-device failure handling.
- Deploy: minipc behind a Tunnel, then the VPS on the domain.
- k6 load test: rooms × sockets, broadcast latency p95, memory per connection.
- A real session with friends and one small streamer.

## Later
Discord bot and Twitch chat bridge for submissions, tag embeddings for Rabbit hole, community pools across sessions, weekly highlights.

## Resume entry (fill in as built)

**CrowdScroll** — *audience-steered shared video feed for streams and watch parties (Go, Next.js, PostgreSQL, Redis, WebSockets)*
- M2: Rooms where viewers vote Stay/Skip and react live while a stream plays the feed. One goroutine owns each room, so concurrent votes change state without locks or double counting.
- M2: Room state rebuilds from Postgres after a restart, and reconnecting clients resume from a snapshot.
- M3: Submissions are dropped into chat and checked by a job queue that loads each clip in a headless browser to prove it plays, then screens it for safety. Rejected submitters get the reason within seconds.
- M4: Pluggable ranking strategies over smoothed vote, reaction and watch-time signals, with fairness limits and an explanation for every pick.
- M5: Live at <domain>; load-tested to X concurrent sockets at Y ms p95 broadcast latency on a €N/month VPS.
