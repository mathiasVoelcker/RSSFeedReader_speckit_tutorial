<!--
Sync Impact Report (temporary; remove before commit)
- Version change: (unratified template) → 1.0.0
- Modified principles: none renamed; all five template principle slots filled:
  1. Security by Default
  2. Maintainability & Separation of Concerns
  3. Code Quality & Testing
  4. MVP-First, Incremental Delivery
  5. Simplicity & Scope Discipline
- Added sections: Technology & Environment Constraints; Development Workflow & Quality Gates
- Removed sections: none
- Deferred TODOs: none
-->

# RSS Feed Reader Constitution

## Core Principles

### I. Security by Default
All external input MUST be treated as untrusted, even where the MVP skips validation.
- The backend MUST be the authority on input handling. Once feed URLs are used for outbound
  requests (Extended-MVP), the backend MUST accept only `http`/`https` URLs, MUST apply request
  timeouts and response size limits, and MUST NOT fetch loopback, link-local, or private-network
  addresses (SSRF protection).
- Feed-derived content (titles, links, summaries) MUST be rendered as plain text or sanitized
  (e.g., `HtmlSanitizer`) before any HTML rendering. Raw feed HTML MUST NOT be injected into the DOM.
- CORS MUST allow only the explicit frontend origins; wildcard origins are prohibited.
- Secrets and environment-specific values MUST NOT be committed; configuration MUST come from
  configuration files or environment variables.
- Dependencies MUST be limited to maintained packages and MUST be checked for known
  vulnerabilities (`dotnet list package --vulnerable`) before each release.

Rationale: a feed reader fetches and displays content from arbitrary third parties, so the
fetch and render paths are the main attack surface.

### II. Maintainability & Separation of Concerns
The ASP.NET Core Web API backend and the Blazor WebAssembly frontend MUST remain separate
projects (`backend/RSSFeedReader.Api`, `frontend/RSSFeedReader.UI`) that communicate only through
the HTTP API.
- The backend owns data and feed operations; the frontend owns presentation and user
  interaction. Business logic MUST NOT live in Razor components.
- Storage MUST sit behind an interface (e.g., `ISubscriptionRepository`) so the in-memory
  implementation can be replaced by EF Core + SQLite without changing callers.
- API URLs, ports, and origins MUST be read from configuration (`wwwroot/appsettings.json`,
  `launchSettings.json`) and MUST NOT be hardcoded.
- Backend port, frontend port, `ApiBaseUrl`, and the CORS policy MUST be kept consistent, and any
  change to one MUST update the others in the same change.

Rationale: a clean seam at storage and at the API boundary lets post-MVP features (persistence,
polling) land without a rewrite.

### III. Code Quality & Testing
Code MUST be readable, consistently formatted, and verified by automated tests.
- The solution MUST build with zero errors and zero new compiler warnings; nullable reference
  types MUST be enabled.
- Code MUST follow standard .NET conventions (PascalCase types/members, async methods suffixed
  `Async`, one public type per file) and be formatted with `dotnet format`.
- Backend API behavior MUST be covered by xUnit tests. Each endpoint MUST have at least one
  success-path and one failure-path test; feed parsing MUST be tested against known-good RSS
  and Atom samples.
- Tests MUST NOT depend on live network access; feed fetching MUST be tested through a
  substitutable `HttpClient`/handler.
- A defect fix MUST include a regression test.

Rationale: the stakeholder documents defer testing to post-MVP, but a small test suite is
cheap now and prevents regressions as features are added.

### IV. MVP-First, Incremental Delivery
Work MUST be delivered in this order, and a later phase MUST NOT begin until the earlier phase
meets its definition of done:
1. **MVP**: add a subscription by URL and display the subscription list (in-memory storage, no
   fetching, no parsing).
2. **Extended-MVP**: manual refresh, item display (title and link), and a basic
   "Failed to load feed" error.
3. **Post-MVP**: persistence, removal, background polling, read/unread, and other features, each
   specified separately.

Rationale: the project exists to demonstrate a small slice working end to end; sequencing keeps
each increment shippable.

### V. Simplicity & Scope Discipline
Implement only what the current phase requires (YAGNI).
- Features outside the current phase's scope MUST NOT be added speculatively.
- A new dependency or abstraction MUST be justified in the plan by a requirement it satisfies.
- Blazor template demo pages (`Home.razor`, `Counter.razor`, `Weather.razor`) MUST be removed
  and routing verified (exactly one `@page "/"`) before any UI feature work begins.
- Documented MVP simplifications (in-memory storage, no URL validation, plain-text UI) are
  allowed only for the MVP phase and MUST be revisited when moving to Extended-MVP.

Rationale: speed of delivery is the project's stated priority, and unused complexity is the
largest threat to it.

## Technology & Environment Constraints

- **Stack**: ASP.NET Core Web API (backend) and Blazor WebAssembly (frontend), in C#.
  Deviations require a constitution amendment.
- **Feed parsing**: `System.ServiceModel.Syndication` for RSS/Atom; `HttpClient` (via
  `IHttpClientFactory`) for fetching.
- **Persistence (post-MVP)**: EF Core + SQLite.
- **Platforms**: the app MUST build and run on Windows, macOS, and Linux. Build and verification
  commands in docs and scripts MUST NOT assume a single OS shell.
- **Audience**: a single-user app running locally; authentication and multi-user concerns are
  out of scope until specified.
- **Default local ports**: backend `http://localhost:5151`, frontend `http://localhost:5213`.

## Development Workflow & Quality Gates

- Every change MUST trace to a spec, plan, and task produced through the Spec Kit workflow.
- Before a change is merged, the following MUST pass: clean build, `dotnet test`,
  `dotnet format --verify-no-changes`, and a manual smoke test (backend and frontend running,
  CORS correct, no browser console errors, the MVP flow working).
- Reviews MUST check compliance with this constitution, including security items in Principle I.
- Commits MUST be small, focused, and use descriptive messages.
- Documentation (stakeholder documents, port and config notes) MUST be updated in the same
  change that alters the behavior it describes.

## Governance

This constitution supersedes other practices and conventions in this repository. Where a
stakeholder document conflicts with it, the conflict MUST be resolved by amending one of them.

- **Amendments**: proposed as a change to this file with a stated rationale and a Sync Impact
  Report; approved by the project owner; any affected plans, specs, or tasks MUST be updated
  as part of the amendment.
- **Versioning**: semantic versioning. MAJOR for removing or redefining a principle in a
  backward-incompatible way; MINOR for adding a principle or section or materially expanding
  guidance; PATCH for clarifications and wording fixes.
- **Compliance review**: each plan MUST include a constitution check, and each review MUST verify
  compliance. Violations MUST be fixed or explicitly justified in the plan's complexity-tracking
  section.

**Version**: 1.0.0 | **Ratified**: 2026-10-28 | **Last Amended**: 2026-10-28
