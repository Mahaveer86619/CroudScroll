# Recommendation

`internal/ranking` is pure: candidates with their stats plus a room context in, ranked picks with explanations out. No database or network access, so strategies are unit-tested and replayed against recorded sessions.

```go
type Strategy interface {
    Rank(ctx RoomContext, candidates []Candidate, rng *rand.Rand) []Pick // Pick has Candidate + Explanation
}
```

The host panel shows the explanation for the current and on-deck items ("why this?"): each score component and its weight.

## Signals

Normalised to 0..1 and smoothed so a few votes can't dominate.

| Signal | Definition |
|---|---|
| Stay rate | `(stay + a) / (stay + skip + a + b)`, prior from the room's overall stay rate |
| Reaction score | (👍 + ❤️ + 😂 − 👎) / impressions, same smoothing |
| Dwell | time on screen before a crowd skip relative to the room median; timeouts count as full |
| Demand | number of people who submitted the same item (duplicate submissions boost, see [submissions.md](submissions.md)) |
| Freshness | exponential decay from approval time, half-life 24 h |
| Repeat penalty | excluded if shown this session; decaying penalty if shown in recent sessions |
| Diversity | penalty when recent items share a submitter, source or tag |
| Submitter reputation | smoothed stay rate of the submitter's past items, small weight |

Room-level stats and global stats are kept separately; room stats take over once they have enough impressions (default 10).

## Modes

| Mode | Behaviour |
|---|---|
| Balanced (default) | weighted sum of all signals, then softmax sampling from the top 10 so it isn't predictable |
| Democracy | heavy stay rate and reactions |
| Fresh | heavy freshness; never-shown items first |
| Rabbit hole | similarity to the current item's tags (embeddings later) |
| Chaos | near-uniform random, slight popularity tilt |
| Underdogs | items with few impressions, inverse popularity |
| Host picks | no automatic picks; the host fills the on-deck list |

Weights live in room config, not code constants.

## Fairness constraints

Applied after scoring, before filling the on-deck list:
- At most one item per submitter in any 5 consecutive items.
- At most 3 consecutive items from the same source.

## Encores

Crowd favourites come back, rarely.
- Eligible: ≥ 20 impressions, stay rate and ❤️ share both in the room's top 5%, not shown in the last 7 days.
- At most one per 25 items, chosen with a small probability rather than on a schedule.
- Labelled `🔁 Encore`. Votes during an encore count again, so a favourite people are tired of drops out.
