# Submissions

Viewers submit by pasting a link in room chat, `/submit <url>`, or the upload button on the Remote. The chat message is the submission.

## Check order

Cheapest first. A failing check stops the pipeline and the submitter hears why.

| Step | Where | Time | Rejects with |
|---|---|---|---|
| 1. Parse, normalise to `(provider, id)`, check source is supported | api, inline | µs | `unsupported_site`, `not_a_post` |
| 2. Submitter and room limits | api, Redis | µs | `rate_limited`, `submissions_closed`, `banned` |
| 3. Duplicates | api, Postgres | ms | `duplicate_recent` (or boosts, below) |
| 4. Provider metadata (oEmbed, YouTube Data API, upload sniffing) | worker | ~300 ms | `private_or_deleted`, `embedding_disabled`, `age_restricted`, `region_blocked`, `too_long`, `provider_degraded` (held, not rejected) |
| 5. Playability probe (see [playback.md](playback.md)) | worker | 2–10 s | `wont_play` |
| 6. Moderation (see [moderation.md](moderation.md)) | worker | 1–10 s | `flagged`, or held for review |

Steps 1–3 run in the request; the rest are River jobs. Uploads are transcoded between steps 4 and 5.

## Feedback in chat

- Accepted: a public line, `🎬 <name> added a clip`, so submitting is visible.
- In review: private, `Waiting for a moderator`.
- Rejected: private, with the reason and what to do instead.

| Code | Message |
|---|---|
| `unsupported_site` | We can play uploads, YouTube, Twitch clips, Reddit, Imgur, Giphy and Streamable. |
| `not_a_post` | That looks like a profile or page, not a single post. |
| `private_or_deleted` | That post is private or was removed. |
| `embedding_disabled` | The creator doesn't allow it to be played on other sites. Try another upload of the same clip. |
| `age_restricted` | Age-restricted posts can't play here. |
| `wont_play` | We couldn't get it to play. Try a different link to the same clip. |
| `too_long` | This room takes clips up to N minutes. |
| `duplicate_recent` | Already shown 12 minutes ago. |
| `flagged` | This doesn't fit the room's rules. |
| `rate_limited` | You can submit again in 40 s. |

Codes are stored on the submission; the host panel shows the common ones.

## Duplicates

Identity is `(provider, id)` after normalisation (tracking parameters stripped, URL variants unified). Uploads use a perceptual hash of sampled frames, which also catches the same clip re-uploaded.

- Already in the pool: not a rejection. The new person is added as a co-submitter and the item gains demand. Reply: `Already queued, your vote bumped it up 👍`.
- Shown this session: `duplicate_recent`.
- Shown in an earlier session: accepted; the repeat penalty handles it.

## Where am I in the queue

The order changes with votes, so submitters see a band (`up next`, `soon`, `in the pool`) rather than a number, updated when it changes.
