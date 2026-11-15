# Feed

Each live room has one feed, run by the room actor (see [architecture.md](architecture.md)). Only the server decides what's current and when it ends.

## States

```
idle ──start──> playing ──end condition──> advancing ──> playing
                  │  ▲                         │
               pause resume               pool empty
                  ▼  │                         ▼
                paused                       starved ──item approved──> playing
```

- `advancing` takes slot 1 of the on-deck list, writes the transition to Postgres, then broadcasts `feed.item.changed`.
- `starved`: nothing eligible. The Stage shows a "submit something" screen with the room link and QR code.
- `paused` freezes the timer; votes are kept.

## End conditions

An item ends at the first of:

1. **Crowd skip**: votes ≥ max(3, 20% of active participants) and skip share ≥ threshold (default 60%), held for 2 s so the bar can't flicker the feed.
2. **Time**: `ends_at` = start + duration (or 12 s for images/text), capped at the room's max (default 60 s).
3. **Host**: skip, remove, or pin something else to play now.
4. **Playback failure** on the Stage (see [playback.md](playback.md)).

A Stay majority when time runs out extends a capped item once by 15 s (for long clips cut by the cap); never more than once.

## Skip guards

- No vote-skip in the first 3 s + stream delay, so the crowd sees an item before killing it. Host skip is always instant.
- **Stream delay** (Stage mode): viewers watching on Twitch see the Stage 3–10 s late. The host sets the delay (default 5 s when streaming); the server shifts the skip guard and keeps accepting votes for that long after an item ends, counting them toward the ended item's stats, not the next one's decision.

## On-deck list

The actor keeps 3 items lined up, broadcast as `feed.next`, so clients preload and a burst of skips never waits for a pick.

- Slot 1 is locked once announced; clients have already loaded it. It only changes if it becomes ineligible or the host pins.
- Slots 2–3 are re-ranked whenever votes change meaningfully (debounced to every 5 s).
- An item that is removed, reported past the threshold or marked `play_error` is dropped and refilled.
- A host pin goes into slot 1 and pushes the others down.

## Host controls

pause, resume, skip, pin (play next), play now, remove (also from the pool), override (cancel a crowd skip and keep the item), extend, change thresholds and mode live.

All host actions are written to `room_events`.

## Votes and reactions

- One Stay/Skip choice and one reaction per participant per item, changeable while it's current.
- Tallies are kept in memory by the actor and upserted to Postgres in batches every 250 ms.
- Updates are broadcast coalesced, at most 5 per second per room.
- "Active participants" means connected and has sent anything in the last 5 minutes, so idle tabs don't make thresholds unreachable.

## Recovery

On startup or when a room actor is created, it loads from Postgres: the open feed item and its start time, its votes and reactions, the on-deck list, and room config. If the open item's time has already run out, it advances. Clients reconnect and receive a full `room.state` snapshot.
