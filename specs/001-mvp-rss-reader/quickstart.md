# Quickstart: Validating the MVP

A run-and-verify guide. API details are in [contracts/subscriptions-api.md](contracts/subscriptions-api.md) and the data rules in [data-model.md](data-model.md).

## Prerequisites

- .NET 10 SDK. Check with `dotnet --version`, which must report 10.x. The locally installed 3.1.402 is not sufficient.
- A browser. No other services are needed.

## 1. Configuration check

Ports and URLs must agree in all four places (constitution, Principle II):

| Item | Location | Expected |
|------|----------|----------|
| Backend port | `backend/RSSFeedReader.Api/Properties/launchSettings.json` | `http://localhost:5151` |
| Frontend port | `frontend/RSSFeedReader.UI/Properties/launchSettings.json` | `http://localhost:5213` |
| API base URL | `frontend/RSSFeedReader.UI/wwwroot/appsettings.json` | `ApiBaseUrl` = `http://localhost:5151/api/` |
| CORS origins | `backend/RSSFeedReader.Api/Program.cs` (from configuration) | Includes `http://localhost:5213` |

## 2. Automated checks

```text
dotnet build
dotnet test
dotnet format --verify-no-changes
```

Expected: zero errors, no new warnings, all tests pass, and no formatting changes.

## 3. Template cleanup check (before any UI work)

Only the MVP page may remain in `frontend/RSSFeedReader.UI/Pages/`. `Home.razor`, `Counter.razor` and `Weather.razor` must be gone, and exactly one page may declare `@page "/"`. Build and open the frontend; the browser console must show no "ambiguous route" error.

## 4. Run

Start the backend, then the frontend, each in its own terminal:

```text
dotnet run --project backend/RSSFeedReader.Api
dotnet run --project frontend/RSSFeedReader.UI
```

Open `http://localhost:5213`.

## 5. Manual scenarios

| # | Action | Expected |
|---|--------|----------|
| 1 | Open the app with a fresh backend | Empty list with a "no subscriptions yet" message (FR-007) |
| 2 | Paste `https://devblogs.microsoft.com/dotnet/feed/` and submit | Appears in the list immediately; input is cleared (FR-001..004) |
| 3 | Add a second, different URL | Both are listed, in the order added (FR-008) |
| 4 | Submit the first URL again | No second entry appears (FR-006) |
| 5 | Submit with an empty or spaces-only field | Nothing is added (FR-005) |
| 6 | Paste ` https://example.com/feed ` with spaces | Added without the spaces |
| 7 | Add `javascript:alert(1)` | Listed as inert text, not clickable, nothing executes |
| 8 | Restart the backend and reload | List is empty (accepted MVP limit) |

## 6. Done criteria

The MVP is working when scenarios 1 to 6 pass, the browser DevTools console shows no connection or CORS errors, and the automated checks in step 2 are green.
