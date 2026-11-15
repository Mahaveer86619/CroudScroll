# Realtime protocol

One WebSocket per tab: `wss://<host>/ws?room=<slug>`. Viewers authenticate with the session cookie; the Stage with its stage token.

## Envelope

```json
{ "type": "feed.vote", "id": "<client uuid>", "seq": 1042, "data": { } }
```

- `id`: client-generated on client messages and echoed in acks, so retries are idempotent.
- `seq`: set by the server on broadcasts. A gap means the client asks for a fresh snapshot.

## Events

| Event | Direction | Notes |
|---|---|---|
| `room.join` | C→S | first message; reply is `room.state` |
| `room.state` | S→C | snapshot: mode, config, current item + `started_at`/`ends_at`, on-deck list, tallies, own vote and reaction, last 50 chat messages, `seq` |
| `room.presence.updated` | S→C | count + capped name list, at most 1/s |
| `clock.ping` / `clock.pong` | both | clock offset estimate |
| `feed.item.changed` | S→C | new current item, `started_at`, `ends_at`, previous item's end reason |
| `feed.next` | S→C | on-deck list (3); slot 1 locked |
| `feed.state` | S→C | paused, resumed, starved, extended |
| `feed.vote` | C→S | `{ item_id, choice: "stay" \| "skip" \| null }` |
| `feed.vote.updated` | S→C | totals, at most 5/s |
| `feed.reaction` | C→S | `{ item_id, emoji: "👍" \| "👎" \| "❤️" \| "😂" \| null }` |
| `feed.reaction.updated` | S→C | totals, at most 5/s |
| `player.event` | C→S | Stage (any client in Together mode): `ready`, `started`, `ended`, `error`, with `item_id` and duration |
| `chat.message` | both | server adds author, time, optional `item_id`; links become submissions |
| `chat.deleted` | S→C | moderator removal |
| `submission.feedback` | S→C | submitter only: accepted, in review, queue band, or rejection code + message |
| `submission.created` | S→C | hosts and mods |
| `moderation.updated` | S→C | submitter and mods |
| `report` | C→S | `{ item_id \| message_id, reason }` |
| `host.command` | C→S | pause, resume, skip, pin, play_now, remove, override, extend, set_config; role checked |
| `error` | S→C | `{ code, message, ref_id }` |

## Rules

- Votes, reactions and player events carry `item_id`; ones for an item that isn't current are dropped (except late votes inside the stream-delay window, see [feed.md](feed.md)).
- Clients never pick items or end them. They render `ends_at` and report player events.
- `player.event` from non-Stage clients in Stage mode is ignored; in Together mode failures are aggregated (see [playback.md](playback.md)).
