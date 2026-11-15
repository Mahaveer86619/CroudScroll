# Data model

| Table | Key columns | Constraints |
|---|---|---|
| `users` | id, display_name, is_guest, oauth_provider, oauth_subject, trust_level | unique (oauth_provider, oauth_subject) |
| `rooms` | id, slug, owner_id, mode (stage/together), visibility, config jsonb, stage_token_hash, status | unique slug |
| `room_members` | room_id, user_id, role, muted_until, banned | pk (room_id, user_id) |
| `content_items` | id, provider, provider_id, media_kind, title, duration_ms, poster_key, upload_key, phash, tags text[], probe_status, last_probed_at | unique (provider, provider_id) |
| `submissions` | id, room_id, content_item_id, submitter_id, chat_message_id, status, reject_code, created_at | one row per person per item per room: unique (room_id, content_item_id, submitter_id) |
| `room_pool` | room_id, content_item_id, review_status, added_at, removed_at | pk (room_id, content_item_id); only `approved` and not removed is eligible |
| `moderation_reviews` | id, room_id, content_item_id, source (auto/human), decision, scores jsonb, reason, reviewer_id, at | append-only |
| `feed_sessions` | id, room_id, mode, strategy, started_at, ended_at | one open session per room |
| `feed_items` | id, session_id, content_item_id, position, started_at, ends_at, ended_at, end_reason, is_encore | end_reason: crowd_skip, timeout, ended, host_skip, removed, play_error |
| `on_deck` | session_id, slot, content_item_id, locked | pk (session_id, slot) |
| `stay_skip_votes` | feed_item_id, user_id, choice, late, updated_at | pk (feed_item_id, user_id) |
| `reactions` | feed_item_id, user_id, emoji, updated_at | pk (feed_item_id, user_id) |
| `chat_messages` | id, room_id, user_id, feed_item_id, body, client_id, deleted_at | unique (user_id, client_id) |
| `reports` | id, reporter_id, content_item_id or chat_message_id, room_id, reason, status | unique (reporter_id, target) |
| `content_stats` | content_item_id, room_id (null = global), impressions, stays, skips, reactions jsonb, dwell_ms_sum, last_shown_at | pk (content_item_id, room_id) |
| `room_events` | id, room_id, kind, actor_id, payload jsonb, at | host and moderation actions only |

Notes:
- `content_items` is global, so a clip probed once is reusable across rooms; whether it's allowed in a room lives in `room_pool`.
- Submitting a duplicate inserts another `submissions` row; demand is the count of submitters per pool item.
- Votes and reactions are upserts holding the current choice. Add an append-only vote history table only if analysis needs it.
- `content_stats` is updated by the worker when a feed item ends, not on every vote.
