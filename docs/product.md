# Product

CrowdScroll is a shared feed that a crowd fills and steers. People drop clips into a room, a recommender picks what plays next, and everyone votes Stay or Skip, reacts and chats while it plays.

**The website is the feed.** Every item plays inside CrowdScroll. There are no "open on Instagram" cards or outbound links in the feed. If we can't play something reliably in our own player, we don't accept it, and the submitter is told why.

## Room modes

A room runs in one of two modes, chosen by the host.

### Stage mode (streamers, a TV at a party)

- One **Stage** screen plays the feed with sound: `/r/<slug>/stage`, added to OBS as a browser source or opened full screen on a TV. It shows the clip, the Stay/Skip bar, reaction bursts and an optional chat overlay. No controls.
- Everyone else opens the **Remote** on their phone or laptop: what's playing now (poster, title, time left), Stay/Skip, the four emoji reactions, chat and submit. They watch the clip on the stream or the TV, not on the Remote.
- Only one device has to play the clip, so autoplay with sound, provider quirks and sync all reduce to one well-controlled browser.

### Together mode (friends online, no stream)

- Every participant's page plays the feed in sync, with the same vote bar, reactions and chat beside it.
- Clips autoplay muted; one tap unmutes for the rest of the session.

A host can switch modes between sessions. Both modes share the same server, feed logic and data.

## Roles

| Role | Can |
|---|---|
| Host | everything: mode, feed controls, thresholds, moderation, roles, end room |
| Moderator | approve/reject submissions, remove items and messages, mute/ban |
| Viewer | vote, react, chat, submit, report |
| Stage | a display only; joins with a secret stage link, no account |

Viewers can join as guests with just a display name, or sign in with Discord or Twitch. Asking a stream audience to make an account first kills participation.

## Core loop

1. Someone pastes a link (or uploads) in chat.
2. It's checked: supported, not a duplicate, actually plays in our player, safe. Bad submissions get a private reason within seconds.
3. Approved items join the room's pool. The recommender keeps the next three lined up.
4. The current item plays. Votes and reactions update live. Enough Skip votes, a timeout or the host moves the feed on.
5. Every item's votes, reactions and watch time feed back into ranking for later.

## Fixed decisions

1. Everything plays in CrowdScroll. No link-outs in the feed.
2. Stay/Skip decides what happens now; emoji reactions are longer-term signal. They are separate.
3. Moderation before exposure: nothing unreviewed reaches a public room.
4. The host can always pause, skip, pin and override the crowd.
5. No custom livestreaming. The Stage is captured by OBS; Twitch/YouTube do the broadcast.
6. Website-native submissions first. Discord and Twitch chat bridges come later through the same pipeline.
7. Modular Go monolith plus a worker. No microservices.
8. External videos are played through their platforms' official players, not downloaded and re-hosted. Only user uploads live on our storage.

## Not in scope for now

Native mobile apps, our own streaming, monetisation, personal per-user feeds outside rooms.
