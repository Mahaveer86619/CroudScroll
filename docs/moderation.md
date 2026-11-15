# Moderation

Rule: nothing reaches a public room without an approved review. In private (invite-only) rooms, passing automated checks or a host approval is enough.

## Automated checks

Run after the playability probe, so only playable items are checked.

| Input | Check |
|---|---|
| Title, caption, chat messages | hosted text moderation API (OpenAI's moderation endpoint is free) |
| Poster frame, images | small NSFW/violence image classifier, ONNX Runtime on CPU in a sidecar |
| Uploaded video | frames sampled every 2 s with ffmpeg, same classifier |
| External video | poster frame from the probe plus up to 5 frames captured during the probe |

Both classifiers sit behind interfaces (`TextModerator`, `ImageModerator`) so providers can be swapped. Scores fall into three bands: approve, review, reject. Room owners can pick a stricter preset.

For external videos we only see a few frames, so in public rooms they go to review unless the submitter is trusted (a history of approved items with no upheld reports).

## Review queue

In the host panel: item player, frames, scores, submitter history, approve/reject with a reason. Every decision, automated or human, is a row in `moderation_reviews`.

## Reports and removal

- Any viewer can report the current item or a chat message. The reporter stops seeing that item immediately (Together mode) and gets a confirmation.
- Three reports (configurable) from distinct accounts pull an item off the Stage and the on-deck list pending review.
- Removed items leave the pool everywhere, including future sessions.
- Repeat offenders: mute, ban from room, submission cooldown.

## Legal basics before public rooms

- Uploaded media is served through Cloudflare with its CSAM scanning tool on.
- Terms, privacy note, and a takedown contact address.
- Public rooms stay off until the review queue and reporting work; start with invite-only rooms.
- Guests have no email; keep personal data minimal.
