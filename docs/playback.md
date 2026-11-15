# Playback

Rule: if it can't play in our player, it doesn't enter the feed. This is enforced at submission time by the playability gate, not discovered when the room is watching.

## Sources

A source is supported only if its player gives us what the feed needs: muted autoplay, programmatic unmute, a reliable "started" signal, and ideally an "ended" signal, duration and seek.

| Source | Player | Autoplay | Ended / duration | Seek | Status |
|---|---|---|---|---|---|
| Uploads (video, image, GIF) | our `<video>` / `<img>`, files on R2 | yes | yes | yes | MVP |
| YouTube, Shorts | IFrame Player API | muted, yes | yes | yes | MVP |
| Twitch clips | clip embed (needs `parent=<our domain>`) | muted, yes | partial | no | MVP |
| Reddit video | `v.redd.it` HLS via hls.js | yes | yes | yes | MVP |
| Imgur, Giphy, direct mp4/gif/jpg | `<video>` / `<img>` | yes | yes | yes | MVP |
| Streamable | embed / direct mp4 | yes | yes | yes | MVP |
| TikTok | embed player v1, postMessage API | to verify | to verify | to verify | M0 spike decides |
| Instagram reels | official embed iframe | to verify (may need a click) | no | no | M0 spike decides |
| X / Twitter video | widgets.js | likely no | no | no | probably excluded |

Anything not on the list is rejected at submission with a list of what works. "To verify" rows are settled by the M0 spike: open each in the Stage page inside OBS and in Chrome on Android, and record what actually happens. A source that needs a click to start is not supported in either mode; we don't make the host click every clip.

Images and text posts are first-class: shown natively for a fixed time (default 12 s).

## Player interface

One React component per source, all behind the same interface, so the feed never branches on provider:

```ts
interface FeedPlayer {
  load(item: FeedItem, opts: { offsetMs: number; muted: boolean; preload: boolean }): void
  play(): void
  setMuted(m: boolean): void
  destroy(): void
  caps: { seek: boolean; ended: boolean; duration: boolean }
  on(ev: 'ready' | 'started' | 'ended' | 'error', fn: (info?: PlayerInfo) => void): void
}
```

The same components power the Stage, Together mode and the submission probe, so "passed the probe" means "plays in the feed".

## Playability gate

The worker runs one long-lived headless Chromium with 1–2 tabs. For each submission it opens an internal `/probe` page that mounts the real `FeedPlayer`, waits for `started`, records duration, and grabs a poster frame. Failure or a 10 s timeout rejects the submission with `wont_play`. Results are cached per `(provider, id)` for 24 h, and items are re-probed when they have been idle for a week, since posts get deleted.

If one provider starts failing across the board (an API change, or Instagram blocking our server's IP), the worker marks the provider `degraded`: new submissions from it are held with a message saying so, and on-deck items from it are replaced. The host panel shows provider health.

## Who decides time

- **Stage mode**: the Stage is the source of truth for playback. It sends `ready`, `started`, `ended` and `error` to the server. The server sets `ends_at` from the start time plus duration (capped by the room's max display time), or ends the item on `ended`.
- **Together mode**: the server sets `ends_at` from the probed duration. Clients follow it.

Clients estimate their clock offset with a ping/pong at connect and every 30 s, so countdowns and seeks use server time.

## Sync in Together mode

Each client seeks to `now - started_at` on load for seekable sources and corrects drift above 1.5 s. Non-seekable sources start from the beginning for a late joiner; the shared timer still ends the item for everyone at the same moment.

## Preloading

The server keeps 3 items on deck (see [feed.md](feed.md)). Clients that play the feed warm them in tiers to limit memory and data:

| Slot | Stage / desktop | Phone (Together mode) |
|---|---|---|
| 1 | player mounted hidden, muted, buffered | player mounted hidden, muted, buffered |
| 2 | player mounted hidden, muted, buffered | poster + `preconnect` |
| 3 | poster + `preconnect` | poster |

On `feed.item.changed` the client swaps in slot 1, destroys the old player and promotes the next slot. Remotes in Stage mode only prefetch posters.

## When playback fails anyway

There is no link-out fallback, so failures are handled inside the feed:

- **Stage fails** (error or no `started` within 5 s): the server ends the item with `end_reason = play_error`, advances to slot 1 (already loaded, so the stream barely notices), and sends the item back to the probe. Two failures remove it from the pool.
- **One viewer's player fails in Together mode**: retry once. If it still fails, that viewer sees the item's poster frame with the vote bar, chat and timer, plus a small "couldn't load on your device" note, and rejoins playback on the next item. The room isn't affected.
- **Many viewers report failures for the same item**: treated like a Stage failure.

## Why we don't download external videos

Re-hosting Instagram, TikTok or YouTube videos would make playback uniform, but it breaks those platforms' terms and puts other people's copyrighted videos on our storage under our domain, which is the largest legal risk a public site like this can take. The playability gate gets most of the reliability benefit without it. Uploads are the exception: they're the submitter's files, transcoded to H.264 MP4 (faststart) plus a poster, and served from R2.
