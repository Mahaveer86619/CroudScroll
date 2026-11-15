# Hosting

## No GPU needed

The load is WebSockets, Postgres and a headless browser for probes. Moderation uses a hosted text API and a small image model that runs on CPU in a few hundred ms, which is fine for a background queue. A rented GPU would cost more than everything else combined and sit idle. If heavier models are needed later, call a pay-per-second serverless GPU (Modal, RunPod) from the worker behind the same moderator interface.

## Production layout

One VPS running Docker Compose:

| Service | Notes |
|---|---|
| caddy | TLS, reverse proxy, WebSocket upgrade |
| web | Next.js standalone |
| api | Go; room actors |
| worker | Go; River jobs |
| probe | Playwright + Chromium, internal only, 1–2 tabs |
| classifier | Python + ONNX Runtime, internal only |
| postgres | local volume, nightly `pg_dump` to R2 |
| redis | no persistence |

Size: 4 vCPU, 8 GB RAM (Chromium and the classifier are the big users). Hetzner/Netcup/OVH class, roughly €8–15/month.

Uploads and posters live in Cloudflare R2, served from `media.<domain>`.

## Domain

- DNS on Cloudflare, proxied (WebSockets work through it). App on the apex or `crowdscroll.<domain>`, media on `media.<domain>`.
- SSL mode Full (strict); Caddy has its own certificate.
- Twitch clip embeds need our domain in their `parent` parameter, so set the final domain early.

## Before the VPS

The minipc can run the same compose file behind a Cloudflare Tunnel for the private beta, with no open ports and no bill. One caveat: Instagram and TikTok are less likely to block a home IP than a datacenter one, so probe results may differ once moved; re-run the provider spike from the VPS.

## Deploys

GitHub Actions builds images to GHCR on a tag; the server runs `docker compose pull && docker compose up -d`. Migrations run as a one-shot container before the api starts.

## Monthly cost

| Item | Cost |
|---|---|
| VPS | €8–15 |
| R2 | free to 10 GB, then about $0.015/GB, no egress fees |
| Text moderation | free |
| Domain | already owned |
