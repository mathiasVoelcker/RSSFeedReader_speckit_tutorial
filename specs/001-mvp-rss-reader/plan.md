# Implementation Plan: MVP RSS Reader (Subscription Management)

**Branch**: `001-mvp-rss-reader` | **Date**: 2026-10-28 | **Spec**: [spec.md](spec.md)

**Input**: Feature specification from `/specs/001-mvp-rss-reader/spec.md`

## Summary

Build the smallest end-to-end slice of the feed reader: a user pastes a feed URL, it is added to an
in-memory subscription list, and the list is shown in the UI and updates immediately. The backend is
an ASP.NET Core Web API with two endpoints (add, list) over an in-memory repository behind an
interface. The frontend is a Blazor WebAssembly single page that calls those endpoints. No feed
fetching, parsing, or URL validation is done in this phase (see [research.md](research.md)).

## Technical Context

**Language/Version**: C# on .NET 10 (LTS). See the SDK note in [research.md](research.md): the SDK installed locally is 3.1.402 and must be upgraded.

**Primary Dependencies**: ASP.NET Core Web API, Blazor WebAssembly. No other runtime packages for the MVP.

**Storage**: In-memory list inside a singleton repository behind `ISubscriptionRepository`. Data is lost on restart (accepted for the MVP).

**Testing**: xUnit; `Microsoft.AspNetCore.Mvc.Testing` for in-process API tests; unit tests for the repository and normalization logic.

**Target Platform**: Windows, macOS and Linux, local development (browser frontend, localhost backend).

**Project Type**: Web application (separate API and Blazor WebAssembly projects).

**Performance Goals**: A new subscription appears in the list within 1 second of submission (SC-002). No throughput target; single user.

**Constraints**: Single user, localhost only, no auth. Configuration-driven ports and API base URL. Explicit CORS origins.

**Scale/Scope**: One page, two endpoints, one entity. Tens of subscriptions at most.

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

| Principle | Gate | Status |
|-----------|------|--------|
| I. Security by Default | Explicit CORS origins only; no secrets; URLs rendered as text, never as links or HTML; input length bounded; no outbound requests in the MVP, so SSRF rules are not yet triggered | Pass |
| II. Maintainability & Separation | Separate `backend/` and `frontend/` projects talking only over HTTP; storage behind `ISubscriptionRepository`; API URL and ports from configuration | Pass |
| III. Code Quality & Testing | Nullable enabled; xUnit tests with success and failure cases per endpoint; no network in tests; `dotnet format` gate | Pass |
| IV. MVP-First | Scope limited to add and list; no fetching, parsing or persistence | Pass |
| V. Simplicity | Two endpoints, one entity, no extra packages; Blazor template demo pages removed and routing verified before UI work | Pass |

**Post-design re-check (after Phase 1)**: Pass. The design adds no projects, packages or abstractions beyond those the constitution requires. No complexity justification is needed.

## Project Structure

### Documentation (this feature)

```text
specs/001-mvp-rss-reader/
├── plan.md              # This file (/speckit-plan command output)
├── research.md          # Phase 0 output (/speckit-plan command)
├── data-model.md        # Phase 1 output (/speckit-plan command)
├── quickstart.md        # Phase 1 output (/speckit-plan command)
├── contracts/
│   └── subscriptions-api.md   # Phase 1 output (/speckit-plan command)
└── tasks.md             # Phase 2 output (/speckit-tasks command - NOT created by /speckit-plan)
```

### Source Code (repository root)

```text
backend/
├── RSSFeedReader.Api/
│   ├── Program.cs                      # Composition root, CORS policy, endpoint mapping
│   ├── Properties/launchSettings.json  # Port 5151
│   ├── Models/Subscription.cs
│   ├── Contracts/                      # Request and response DTOs
│   ├── Services/
│   │   ├── ISubscriptionRepository.cs
│   │   └── InMemorySubscriptionRepository.cs
│   └── Controllers/SubscriptionsController.cs
└── RSSFeedReader.Api.Tests/
    ├── InMemorySubscriptionRepositoryTests.cs
    └── SubscriptionsApiTests.cs        # In-process endpoint tests

frontend/
└── RSSFeedReader.UI/
    ├── Program.cs                      # Reads ApiBaseUrl from configuration
    ├── Properties/launchSettings.json  # Port 5213
    ├── wwwroot/appsettings.json        # ApiBaseUrl
    ├── Layout/                         # NavMenu trimmed to MVP entries
    ├── Pages/Subscriptions.razor       # The only page; owns @page "/"
    └── Services/SubscriptionsClient.cs # Typed HTTP client for the API
```

**Structure Decision**: The web application layout from the constitution and TechStack document: `backend/RSSFeedReader.Api` and `frontend/RSSFeedReader.UI`, plus one backend test project. Frontend component tests are deferred, since the page is a thin view and holds no business logic. The API tests plus the manual smoke test in [quickstart.md](quickstart.md) cover the UI flow.

## Complexity Tracking

No constitution violations. Nothing to justify.
