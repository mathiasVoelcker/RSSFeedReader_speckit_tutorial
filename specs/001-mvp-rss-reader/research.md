# Research: MVP RSS Reader

All Technical Context items were resolved from the stakeholder documents and the constitution.
The only open question was the .NET version.

## R1: .NET version and local SDK

- **Decision**: Target .NET 10 (LTS).
- **Rationale**: It is the current long-term-support release. .NET 8 leaves support in November 2026. The stakeholder documents name ASP.NET Core and Blazor WebAssembly but no version.
- **Finding**: `dotnet --version` on this machine reports **3.1.402**, which cannot build or even scaffold a Blazor WebAssembly project of this kind. Installing the .NET 10 SDK is a prerequisite for implementation.
- **Alternatives considered**: .NET 8 (near end of support); staying on 3.1 (out of support and lacks the Blazor WebAssembly template).

## R2: API style

- **Decision**: Controller-based Web API with `[ApiController]`, JSON, under the `/api/` prefix.
- **Rationale**: `ApiBaseUrl` in the TechStack document ends in `/api/`. Controllers are a conventional, testable shape and the model-validation attributes cover the input rules.
- **Alternatives considered**: Minimal APIs (equally valid, but the controller shape makes the contract and tests more conventional for later growth).

## R3: In-memory storage and thread safety

- **Decision**: A singleton `InMemorySubscriptionRepository` holding a list guarded by a lock, implementing `ISubscriptionRepository`.
- **Rationale**: Requests are handled concurrently, so a bare `List<T>` is unsafe. The lock keeps insertion order (FR-008) and makes the duplicate check and the add a single atomic step. The interface is the seam for EF Core + SQLite later (Principle II).
- **Alternatives considered**: `ConcurrentDictionary` (loses insertion order); a database now (violates MVP-first).

## R4: Duplicate handling (FR-006)

- **Decision**: Normalize by trimming whitespace, then compare **case-sensitively** on the full string. A duplicate is a no-op that returns the existing entry.
- **Rationale**: The spec says identical URLs are the same subscription. Normalizing case, trailing slashes or schemes would need URL parsing, which the MVP explicitly skips. Add that when validation is introduced.
- **Alternatives considered**: Case-insensitive comparison (host is case-insensitive but the path is not, so it is only correct with real parsing).

## R5: Duplicate and create responses

- **Decision**: `POST` returns `201 Created` for a new subscription and `200 OK` for an existing one, both with the subscription body. Empty or whitespace input returns `400`.
- **Rationale**: Lets the UI refresh from one response shape while letting tests tell the two cases apart. The idempotent 200 avoids an error state the spec does not call for.
- **Alternatives considered**: `409 Conflict` on duplicates (forces the UI to treat a harmless action as an error).

## R6: Input safety without validation

- **Decision**: Reject empty input and cap length at 2048 characters. Beyond that, accept any text. The UI renders the URL as **plain text, not an anchor**.
- **Rationale**: The spec requires no format validation, but the constitution requires untrusted input to be handled safely. A text-only render prevents `javascript:` URL links, and Blazor encodes text by default. The length cap bounds memory use. Scheme restriction is deferred to the phase that makes outbound requests, as the constitution specifies.
- **Alternatives considered**: Full URL validation now (contradicts FR-009 and the stakeholder scope).

## R7: CORS and ports

- **Decision**: Backend on `http://localhost:5151`, frontend on `http://localhost:5213`, with the frontend HTTPS port `https://localhost:7025` also allowed. The CORS policy lists exactly these origins and reads them from configuration. The frontend reads `ApiBaseUrl` from `wwwroot/appsettings.json`.
- **Rationale**: Matches the TechStack document and the constitution's no-wildcard rule.
- **Alternatives considered**: A dev proxy (extra moving part); `AllowAnyOrigin` (prohibited).

## R8: Testing approach

- **Decision**: xUnit. Repository unit tests; in-process API tests via `WebApplicationFactory`. No frontend component tests for the MVP.
- **Rationale**: Meets Principle III with the least machinery. No network access is needed, since the MVP makes no outbound calls.
- **Alternatives considered**: bUnit for the page (more setup than a one-component page justifies right now).
