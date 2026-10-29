# Contract: Subscriptions API

Base URL: configured per environment, `http://localhost:5151/api/` by default. All bodies are JSON
(`Content-Type: application/json`). Property names are camelCase.

## Types

**Subscription**

```json
{ "url": "https://devblogs.microsoft.com/dotnet/feed/" }
```

## GET /api/subscriptions

Returns all subscriptions in the order they were added.

| Status | Body | When |
|--------|------|------|
| 200 OK | `Subscription[]` | Always. An empty list is `[]`. |

## POST /api/subscriptions

Adds a subscription.

**Request body**

```json
{ "url": "  https://devblogs.microsoft.com/dotnet/feed/  " }
```

| Status | Body | When |
|--------|------|------|
| 201 Created | `Subscription` (trimmed URL); `Location` header points at the collection | The URL was new and has been added. |
| 200 OK | `Subscription` (the existing entry) | The URL (after trimming) is already subscribed. The list is unchanged. |
| 400 Bad Request | Problem details | `url` is missing, empty, whitespace only, or longer than 2048 characters after trimming. |

## Cross-cutting rules

- CORS allows only the configured frontend origins; no wildcard.
- The API never makes outbound requests for these endpoints and never interprets the URL.
- Error bodies use the standard problem-details shape and MUST NOT include stack traces.

## Contract test cases

1. GET on a fresh server returns `200` and `[]`.
2. POST a new URL returns `201`; a following GET contains it.
3. POST two different URLs; GET returns them in the order added.
4. POST the same URL twice; the second returns `200` and GET contains one entry.
5. POST a URL with surrounding whitespace; the stored value is trimmed.
6. POST an empty, whitespace-only, missing, or over-length `url` returns `400`; the list is unchanged.
