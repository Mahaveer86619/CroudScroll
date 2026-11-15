# CrowdScroll

## Audience-powered, real-time doom-scrolling

**Tagline:** The audience controls the feed.

**Project type:** Full-stack web application · Real-time collaboration ·
Content discovery · Recommendation systems

**Primary use case:** Interactive entertainment for livestreamers and
online communities.

------------------------------------------------------------------------

## 1. Executive summary

CrowdScroll is a collaborative content-discovery platform where a
community collectively decides what appears in a shared, continuously
scrolling feed.

People submit reels, memes, images, videos and posts. CrowdScroll
collects these submissions, checks them for safety, and places approved
content into a room's feed. While a post is displayed, viewers react
using simple emoji reactions and collectively vote to **Stay** or
**Skip**. A live Stay/Skip bar at the bottom of the content shows the
room's current decision.

The interface is deliberately content-first: the post takes centre
stage, the live chat provides the conversation, and emoji reactions
provide lightweight feedback without filling the screen with buttons and
controls.

CrowdScroll is designed primarily for livestream entertainment. A
streamer creates a room, shares its link with viewers, and displays the
CrowdScroll website in their stream using a browser source or screen
capture. Viewers can participate from their own devices. CrowdScroll
handles the shared feed, chat, reactions, voting, moderation and
recommendation logic; the existing streaming platform handles the actual
broadcast.

The same platform can also be used by friends and communities without a
livestream.

## 2. Product vision

Build a shared internet rabbit hole where the community---not a
conventional individualised social-media algorithm---has meaningful
control over what everyone sees next.

CrowdScroll should feel:

-   **Collective:** everyone in a room participates in the same content
    experience.
-   **Effortless:** viewers can react with one tap or emoji, without
    navigating complex controls.
-   **Entertaining:** the audience can influence the feed and see its
    collective opinion change live.
-   **Content-first:** the UI gives most of its space to the reel, meme
    or post and the conversation around it.
-   **Safe:** content is checked before it reaches the public feed, with
    reporting and moderator controls available throughout a session.
-   **Flexible:** the host can choose how much control belongs to the
    audience and how the recommendation algorithm behaves.

## 3. Target users and primary scenarios

### A. Streamers and their audiences --- primary

A streamer starts a CrowdScroll room and shares its URL with viewers.
The streamer opens the room's full-screen feed in OBS or another
streaming setup. Viewers join the room on their phones or computers.

Viewers can: - Watch the same post at roughly the same time. - React
with emojis. - Vote to Stay or Skip. - Comment in the live chat. -
Submit new content to the room. - Report content they believe is unsafe.

The streamer can: - Start, pause or resume the shared feed. - Change the
recommendation mode. - Skip or pin a post. - Override the audience's
decision. - Moderate submissions and participants. - Pause the room or
end the session.

### B. Friends and online communities --- secondary

A community creates a room for a casual shared scrolling session.
Members submit content and join whenever the room is active. The room
can be used without a streamer or external livestream.

### C. Community content discovery --- future

A persistent community can maintain a shared content pool, build a
history of popular submissions and start themed sessions using its
existing content.

## 4. Core user experience

### 4.1 Content submission

Users can submit content directly through the CrowdScroll website.

Supported submission types should include: - External URLs, such as
supported Instagram, YouTube and other social-media links. - Image
uploads. - Video uploads. - Text or link posts, where appropriate.

Each submission should retain useful attribution, such as its submitter,
source URL, submission time, room or community, and moderation status.

**MVP recommendation:** Start with website-native submissions. Add a
Discord bot later as an additional ingestion channel. The bot should
send submissions through the same backend pipeline as website
submissions.

External social-media URLs should initially be treated as references. Do
not assume every platform permits downloading or embedding every post.
Use supported embeds when available and provide a safe link-out
fallback.

### 4.2 Shared scrolling feed

-   A room has one shared feed state.
-   The currently selected content item is visible to all participants.
-   Feed progression is driven by the room's voting policy, host actions
    and recommendation engine.
-   The feed should be smooth, responsive and visually focused on the
    content.
-   Participants who join late should receive the current room state.
-   Reconnecting participants should recover the latest state without
    creating duplicate votes or comments.

The experience should resemble a simple vertical social feed, but with a
shared room state rather than an independent feed for every viewer.

### 4.3 Emoji-only crowd reactions

Keep the primary reaction UI minimal. Instead of a row of labelled Like,
Dislike, Heart and Laugh buttons, show a compact set of familiar emoji
reactions.

Initial reaction set: - 👍 Like - 👎 Dislike - ❤️ Heart - 😂 Laugh

The exact set should be configurable later, but the MVP should use a
small, consistent set.

Interaction principles: - A viewer can select one reaction for a content
item at a time. - Selecting a different emoji updates their reaction
rather than creating an unlimited number of votes. - The UI can show
compact live counts or a subtle reaction burst without obscuring the
content. - Reactions update in real time for everyone in the room. -
Reactions are separate from the Stay/Skip decision. - Reaction counts
and ratios can inform recommendations, but should not independently
decide whether the current post stays on screen.

The reaction UI should be unobtrusive. The content and live chat should
remain the main focus.

### 4.4 Stay / Skip audience vote

A simple, live-updating Stay/Skip bar should sit at the bottom of the
displayed content.

The bar represents the room's collective decision: - **Stay:** viewers
want the current content to remain or continue. - **Skip:** viewers want
the room to move on.

Design goals: - Display the split as a horizontal bar, with clear but
restrained visual distinction. - Update the proportions live as votes
arrive. - Show the total number of votes, if useful, without crowding
the interface. - Let each participant change their vote while the item
is active. - Count at most one current Stay/Skip vote per participant
per content item per room. - Apply a configurable decision threshold and
voting window. - Allow the host to override the result.

The bar should remain visible at the bottom of the content, not become a
large modal or separate poll screen.

**Suggested initial policy:** Once a minimum number or percentage of
active participants have voted, skip when the Skip share crosses a
configurable threshold. Otherwise keep the post until a maximum display
duration or host action. The exact threshold and timeout should be
configurable and tested with real users.

Do not make the system wait indefinitely for votes. A room needs a
predictable fallback when participation is low.

### 4.5 Live chat

Live chat is the main conversational layer.

-   Messages are associated with the room and can optionally reference
    the currently displayed content.
-   Messages arrive in real time.
-   The chat should remain readable without covering important parts of
    the content.
-   The host and moderators can remove messages or restrict
    participants.
-   Basic anti-spam, rate limits and reporting are required.

The initial release should provide CrowdScroll's own web chat. Twitch or
YouTube live-chat integrations can be added later.

## 5. Recommendation engine

The recommendation engine is the central differentiator. It should
combine room-level decisions with content engagement signals to select
what appears next.

### 5.1 Separate two kinds of feedback

**Immediate room decision**

Stay/Skip votes decide whether the current item remains or the room
moves on. This is a live decision for the current room.

**Longer-term content feedback**

Emoji reactions, watch duration, comments, shares, skips and other
engagement signals help estimate which content is likely to interest a
room or community in future sessions.

These signals should not be conflated. A funny post may receive many
laughing reactions but still be skipped because the audience has seen
enough of it. A controversial post may receive many comments without
being broadly liked.

### 5.2 Candidate ranking signals

Potential signals include: - Stay/Skip ratio. - Like, dislike, heart and
laugh reaction counts. - Watch duration and completion rate, where
measurable. - Early exits and skip timing. - Comment activity, with
safeguards against rewarding arguments or spam. - Shares or saves, if
supported. - Content freshness. - Category or topic preferences. -
Previous exposure and duplicate history. - Room-specific preferences. -
Submission diversity, so one user or category does not dominate.

A first version can use a transparent weighted scoring model. The
weights should be treated as experimental parameters, not as proven
optimal values.

Example conceptual score:

`content_score = room_interest + reaction_score + watch_score + freshness_bonus + diversity_bonus - repetition_penalty`

Use normalised signals, minimum sample sizes, time decay and safeguards
against a small number of votes dominating the ranking. Keep
room-specific feedback distinct from global content popularity.

### 5.3 Recommendation modes

The host or room can choose a feed mode:

1.  **Democracy:** Prioritise content with strong positive Stay/Skip
    results and audience reactions.
2.  **Chaos:** Select more unpredictably, with some popularity
    weighting.
3.  **Rabbit Hole:** Prefer content related to the current post's topic.
4.  **Fresh Internet:** Give newly submitted content more opportunities.
5.  **Balanced Feed:** Mix popular, fresh, diverse and less-exposed
    submissions.
6.  **Minority Rules:** Give lower-ranked or less common submissions a
    chance.
7.  **Host's Choice:** Let the host select, pin or advance content
    manually.

The recommendation engine should be replaceable and testable. Keep the
ranking logic separate from the room and transport layers.

## 6. Content moderation and safety

Safety is a core requirement. Unreviewed submissions should never be
inserted directly into a public room feed.

### 6.1 Moderation pipeline

1.  Receive a URL, image, video or text submission.
2.  Validate the input and identify its source/type.
3.  Extract only metadata and media that can be accessed lawfully and
    reliably.
4.  Run applicable text, image and video safety checks.
5.  Assign a moderation status.
6.  Add approved content to the eligible pool.
7.  Send uncertain content to a moderator review queue.
8.  Reject prohibited or clearly harmful content.
9.  Retain a moderation decision and reason for auditability.

### 6.2 Categories to detect or review

-   Graphic violence and gore.
-   Sexual or explicit content.
-   Hate speech and targeted harassment.
-   Self-harm or suicide-related material.
-   Dangerous challenges or encouragement of harm.
-   Disturbing or potentially triggering material.
-   Spam, scams and malicious links.
-   Other categories configured by the community.

### 6.3 Moderation outcomes

-   **Approved:** Eligible for recommendation and display.
-   **Needs review:** Held out of the feed until a moderator decides.
-   **Rejected:** Not eligible for display.
-   **Removed:** Previously approved content removed after a report or
    later review.

AI moderation is a screening layer, not a guarantee. For external links
that cannot be inspected reliably, keep them pending review or require a
trusted moderator to approve them. Never infer that a post is safe from
its URL or title alone.

### 6.4 User and room safety controls

-   Report a post or comment.
-   Hide or skip content immediately for the reporting user.
-   Moderator review queue.
-   Remove content from active and future feeds.
-   Block or restrict repeat offenders.
-   Rate-limit submissions, reactions, votes and chat messages.
-   Maintain moderation audit logs.
-   Allow room owners to configure stricter content policies.
-   Provide clear feedback when content is rejected.

## 7. Streamer and room controls

The host should have a compact control panel, separate from the
viewer-focused feed.

Host controls: - Create, configure and end a room. - Set room visibility
and joining permissions. - Start, pause and resume the feed. - Select
the recommendation mode. - Set Stay/Skip thresholds and display
timeouts. - Pin, skip or remove the current item. - Override an audience
decision. - Approve or reject pending submissions. - Restrict or remove
participants. - Enable or disable submissions during a session. - View
basic room activity and moderation status.

Provide a clean full-screen or presentation mode for OBS/browser
capture. The stream view should show the content, subtle Stay/Skip bar
and optional chat overlay, without exposing administrative controls.

## 8. Product scope and priorities

### MVP --- prove the core experience

Build one complete, usable room experience before adding integrations.

-   User authentication.
-   Create and join rooms through shareable links.
-   Website-native URL and media submissions.
-   Basic submission queue and moderation status.
-   Shared vertical scrolling feed.
-   Room-level WebSocket synchronisation.
-   Emoji reactions: 👍 👎 ❤️ 😂.
-   Live Stay/Skip bar at the bottom of each post.
-   Live room chat.
-   Basic host controls: pause, resume, skip and pin.
-   Basic moderation: report, approve, reject and remove.
-   A simple, transparent recommendation policy.

### V1 --- make it useful for streamers and communities

-   Configurable recommendation modes.
-   Improved engagement scoring.
-   Moderator dashboard and audit history.
-   Better content metadata and supported embeds.
-   Room-level content categories and preferences.
-   Full-screen OBS-friendly display.
-   Better reconnect and late-join handling.
-   Rate limits and anti-abuse protections.
-   Basic room analytics.

### Later integrations and enhancements

-   Discord bot for content submissions.
-   Twitch chat integration.
-   YouTube Live Chat integration.
-   Community-wide content pools.
-   More advanced topic/category detection.
-   Personal or room-level recommendation experiments.
-   Content leaderboards and weekly highlights.
-   Additional moderation providers and review workflows.

Avoid building a proprietary livestreaming service. Let established
platforms broadcast the stream; CrowdScroll supplies the interactive
shared feed.

## 9. Technical architecture

Start with a modular monolith and background workers. Do not split the
project into microservices before scale or operational needs justify it.

### Suggested stack

  -----------------------------------------------------------------------
  Layer                   Technology              Responsibility
  ----------------------- ----------------------- -----------------------
  Web frontend            Next.js + TypeScript    Feed, rooms, chat, host
                                                  UI

  Backend                 Go + Echo               APIs, authentication,
                                                  room logic

  Real-time transport     WebSockets              Feed state, votes,
                                                  reactions, chat

  Primary database        PostgreSQL              Users, rooms,
                                                  submissions, votes,
                                                  moderation

  Cache and event         Redis                   Live aggregates,
  distribution                                    Pub/Sub, transient room
                                                  state

  Background workers      Go                      Ingestion, metadata,
                                                  moderation, ranking

  Media storage           S3-compatible object    Uploaded images and
                          storage                 videos

  Deployment              Docker Compose          Reproducible local and
                          initially               hosted environment
  -----------------------------------------------------------------------

### Logical components

**API and room service** - Authentication and authorisation. - Room
lifecycle and membership. - Submission and moderation APIs. - Host
controls. - Persistent room configuration.

**Real-time room gateway** - WebSocket connections. - Room presence. -
Chat delivery. - Reaction updates. - Stay/Skip aggregation and
broadcasts. - Feed state synchronisation.

**Content ingestion** - URL validation. - Source identification. -
Duplicate detection. - Metadata extraction. - Upload handling.

**Moderation workers** - Text and media checks. - Moderation status
updates. - Review queue creation. - Removal and re-check workflows.

**Recommendation engine** - Candidate selection. - Room-specific
ranking. - Diversity and repetition controls. - Feed-mode strategies. -
Next-item selection.

### Architecture principles

-   PostgreSQL is the source of truth for durable records.
-   Redis is used for transient state and event distribution, not as the
    only record of votes or moderation decisions.
-   Use idempotent event handling where retries are possible.
-   Validate room membership and permissions on every relevant action.
-   Make feed state recoverable after reconnects.
-   Keep moderation and recommendation decisions observable and
    auditable.

## 10. Initial data model

A starting set of entities:

  Entity             Purpose
  ------------------ --------------------------------------------------------
  User               Account and profile
  Room               Shared scrolling session and configuration
  RoomMember         User membership, role and permissions
  Submission         Submitted URL or uploaded media
  ContentItem        Normalised content eligible for a feed
  ModerationReview   Safety checks, decisions and reasons
  FeedSession        Current feed state and session history
  FeedItem           Content shown in a session and its ordering
  StaySkipVote       A user's current decision for a feed item
  Reaction           A user's emoji reaction to a feed item
  Comment            Room chat message, optionally linked to a feed item
  ContentReport      User reports and review status
  RoomEvent          Durable or diagnostic record of important room actions

Important constraints: - A user should have at most one active Stay/Skip
vote per feed item per room. - A user should have at most one selected
emoji reaction per feed item per room, unless the product later
deliberately supports multiple reactions. - Store changes as updates or
event history where auditability is useful. - Do not treat every
WebSocket message as a durable database record. - Keep the original
source URL and submitter attribution where appropriate.

## 11. Real-time event concepts

Possible WebSocket events:

  -------------------------------------------------------------------------
  Event                     Direction               Purpose
  ------------------------- ----------------------- -----------------------
  `room.join`               Client → Server         Join a room

  `room.state`              Server → Client         Initial room and feed
                                                    state

  `feed.item.changed`       Server → Client         New current content

  `feed.vote`               Client → Server         Submit or update
                                                    Stay/Skip vote

  `feed.vote.updated`       Server → Clients        Broadcast current vote
                                                    totals

  `feed.reaction`           Client → Server         Add or change emoji
                                                    reaction

  `feed.reaction.updated`   Server → Clients        Broadcast reaction
                                                    totals

  `chat.message`            Both                    Send or receive room
                                                    messages

  `submission.created`      Server → Clients        Notify room of a new
                                                    submission

  `moderation.updated`      Server → Clients        Notify relevant users
                                                    or moderators

  `host.feed.control`       Client → Server         Pause, resume, skip or
                                                    pin

  `room.presence.updated`   Server → Clients        Update participant
                                                    presence
  -------------------------------------------------------------------------

The server must validate each event, enforce permissions and rate
limits, and determine the authoritative room state. Clients should not
decide the winning vote or select the next feed item independently.

## 12. Key engineering challenges

The project should demonstrate real engineering depth in:

1.  **Shared state synchronisation:** Everyone should see the same
    current item and room decision, even with latency or reconnects.
2.  **Concurrent voting:** Handle many users voting or changing votes
    without double counting.
3.  **Real-time engagement:** Broadcast reactions and vote changes
    efficiently.
4.  **Recommendation design:** Combine room decisions, engagement,
    freshness and diversity without creating a simplistic popularity
    contest.
5.  **Moderation pipeline:** Keep unreviewed or rejected content out of
    the feed.
6.  **Abuse prevention:** Protect against spam, vote manipulation,
    repeated submissions and chat flooding.
7.  **External media limitations:** Handle unsupported embeds,
    unavailable posts, deleted links and inaccessible metadata
    gracefully.
8.  **Recoverability:** Reconstruct a room's current state after server
    restarts or participant reconnects.

## 13. Success criteria for the first prototype

A useful first demonstration should prove that:

-   Multiple users can join one room.
-   All participants see the same current content.
-   A user can submit content and see its moderation status.
-   Approved content can enter the shared feed.
-   Emoji reactions update live for everyone.
-   Stay/Skip votes update the bottom bar live.
-   The configured voting policy can advance the feed.
-   Chat messages appear in real time.
-   The host can pause, skip and override the feed.
-   A late-joining or reconnecting participant receives the correct
    current state.
-   Reported or rejected content can be removed from the eligible feed.

Start with a small test group and measure whether people enjoy
participating, submitting content and influencing what appears next.

## 14. Resume positioning

**CrowdScroll --- Audience-Driven Real-Time Content Discovery
Platform**\
*Go, Next.js, WebSockets, Redis, PostgreSQL*

Potential resume bullets, to be updated as features are implemented:

-   Built a collaborative content discovery platform that aggregates
    community-submitted media into synchronised, audience-controlled
    scrolling sessions.
-   Engineered real-time room infrastructure supporting live chat, emoji
    reactions, concurrent Stay/Skip voting and synchronised feed
    progression.
-   Designed configurable recommendation strategies combining collective
    audience feedback, engagement signals, freshness and content
    diversity.
-   Implemented asynchronous content ingestion and moderation workflows
    to screen user submissions and prevent unsafe content from entering
    shared feeds.

------------------------------------------------------------------------

## 15. Product decisions to keep fixed

1.  **Content-first UI:** The content is the main attraction. Use
    emoji-only reactions and a compact Stay/Skip bar instead of a
    crowded control panel.
2.  **Stay/Skip is collective:** It determines the current room's
    immediate feed progression.
3.  **Emoji reactions are lightweight feedback:** They help measure
    audience sentiment and inform recommendations, but do not replace
    the Stay/Skip vote.
4.  **Chat is central to the experience:** The conversation around a
    post is part of the entertainment.
5.  **Moderation before exposure:** Unreviewed content must not
    automatically enter a public feed.
6.  **Streamer control remains available:** The host can pause, skip,
    pin and override.
7.  **No custom livestreaming infrastructure in the MVP:** Use
    OBS/browser capture and existing streaming platforms.
8.  **Website-native submission first:** Discord and streaming-platform
    integrations come later.
9.  **Modular monolith first:** Build a maintainable Go backend and
    extract services only when justified.
10. **The recommendation engine is audience-driven:** The crowd
    influences the feed, while safety and host controls remain
    authoritative.

## Final product definition

CrowdScroll is a content-first, audience-powered doom-scrolling platform
built for livestreamers and online communities. Members submit memes,
reels, videos and posts through the website. Submissions pass through a
moderation pipeline before becoming eligible for shared feeds. During a
session, viewers participate through live chat, lightweight emoji
reactions and a real-time Stay/Skip vote bar. Their collective decisions
control immediate feed progression, while accumulated engagement signals
inform configurable recommendation algorithms.

**The audience supplies the content, reacts to it, and decides what
everyone sees next.**
