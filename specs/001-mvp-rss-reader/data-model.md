# Data Model: MVP RSS Reader

## Subscription

A feed the user follows.

| Field | Type | Rules |
|-------|------|-------|
| `url` | string | Required. Trimmed of leading and trailing whitespace. Non-empty after trimming. At most 2048 characters. Identity of the subscription: compared case-sensitively as an exact string. No format validation (FR-009). |

There is no separate ID in the MVP. The URL is the identity (spec: Key Entities).

## Subscription List

An ordered collection of subscriptions, held in memory by the backend.

- **Order**: insertion order (FR-008).
- **Uniqueness**: no two entries have the same `url` (FR-006).
- **Lifetime**: lives as long as the backend process. It is empty on every start (accepted MVP limit).
- **Concurrency**: the duplicate check and insert happen as one atomic step.

## State transitions

```text
(not subscribed) --add(url)--> (subscribed)
(subscribed)     --add(url)--> (subscribed)   # no-op, existing entry returned
```

Removal is out of scope for this feature.

## Validation summary

| Input | Result |
|-------|--------|
| `"  https://example.com/feed  "` | Stored as `https://example.com/feed` |
| `""` or whitespace only | Rejected (400) |
| Longer than 2048 characters after trim | Rejected (400) |
| Already present | Existing entry returned (200), list unchanged |
| Anything else | Added (201) |
