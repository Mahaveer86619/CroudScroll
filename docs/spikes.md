# Spikes

Anything marked uncertain in these docs is tried as a throwaway mock flow first. Only what the mock proves gets built into the real app, and the doc it came from is updated with the result.

A spike lives in `spikes/<name>/`, is time-boxed, and ends with a short results section appended here. Spike code is never merged into `server/` or `web/`; it's rewritten properly.

| Spike | Question | Decides | Status |
|---|---|---|---|
| Provider players | For YouTube, Twitch clips, Reddit, TikTok, Instagram and X: in OBS and Chrome on Android, does it autoplay muted, unmute from code, report start/end/duration, seek, and fail detectably? | supported source list in [playback.md](playback.md) | open |
| Probe from a datacenter | Do TikTok and Instagram embeds still load when the probe runs on the VPS instead of the minipc? | whether those sources can be gated automatically | open |
| Stage + Remote flow | With 5–10 people on phones and one Stage in OBS, is the Remote enough without seeing the clip on it? Do people vote? | Remote layout, whether it shows a muted mirror | open |
| Stream delay voting | With a real Twitch delay, does the skip guard plus late-vote window feel fair? | defaults in [feed.md](feed.md) | open |
| Skip thresholds | Does 60% skip with a minimum of max(3, 20%) move the feed at a good pace in small and large rooms? | default room config | open |
| Phone preloading | Memory and data use of 1–2 hidden players on a mid-range Android; does a double skip feel instant enough? | preload tiers | open |
| Chat submissions | Do people find pasting links in chat natural, and do the rejection messages lead to better resubmissions? | submit UX, message copy | open |
| Together mode sync | Drift and late-join behaviour across 5 devices on mixed networks | drift threshold, seek rules | open |
| Encores | Do replays of favourites feel like a treat or a rerun? | encore rate and thresholds | open |

## Results

None yet.
