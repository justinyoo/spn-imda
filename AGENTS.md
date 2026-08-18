# Repository Instructions for Coding Agents

This document describes the **prepared starter** for the Smart Parking
Navigator workshop as it exists today. It must not be treated as evidence that
parking features, tests, or the internal API contract have already been
implemented. Follow it so that agent-generated changes stay safe, buildable,
and consistent with the current repository.

## What This Repository Is

A .NET Aspire solution scaffold for a Singapore HDB car park finder. The
frontend and backend surfaces are currently empty scaffolds. `IDEATION.md`
holds the starting product concept; `PRD.md` and `TRD.md` do not exist yet and
are created later in the workshop (see `docs/02-generate-prd-trd.md`). Do not
invent application behavior, endpoints, or UI beyond what is described below
until those requirement documents exist and are reviewed.

## Solution Layout

The solution is defined in `CarparkAvailability.slnx` and contains four
projects under `src/`:

- **`CarparkAvailability.ApiApp`** (`Microsoft.NET.Sdk.Web`) — ASP.NET Core
  minimal API. Currently exposes only `GET /api`, returning a static starter
  payload. References `CarparkAvailability.ServiceDefaults`. Links
  `data/HDBCarparkInformation.csv` into `Data/HDBCarparkInformation.csv` at
  build/publish time via a `Content` item. This project is the only place
  intended to hold the `DataGovSg:ApiKey` value and to call data.gov.sg.
- **`CarparkAvailability.WebApp`** (`Microsoft.NET.Sdk.Web`) — Blazor app using
  interactive server render mode and `Microsoft.FluentUI.AspNetCore.Components`
  (Fluent UI Blazor). Contains `Components/App.razor`,
  `Components/Layout/MainLayout.razor`, `Components/Pages/Home.razor`, and
  `Components/Pages/Error.razor`. References
  `CarparkAvailability.ServiceDefaults`. This project is where the
  `GoogleMaps:ApiKey` value is supplied for browser-side map rendering.
- **`CarparkAvailability.AppHost`** (Aspire AppHost SDK) — the Aspire
  orchestrator (`AppHost.cs`). It reads `google-maps-api-key` and
  `data-gov-sg-api-key` as secret configuration parameters, injects
  `DataGovSg__ApiKey` into ApiApp only, injects `GoogleMaps__ApiKey` into
  WebApp only, and wires WebApp to reference and wait for ApiApp with an
  external HTTP endpoint. `aspire.config.json` (repository root and under
  `src/CarparkAvailability.AppHost/`) points the Aspire CLI at this project.
- **`CarparkAvailability.ServiceDefaults`** — shared Aspire service-defaults
  library (`IsAspireSharedProject`). `Extensions.cs` configures OpenTelemetry
  logging/metrics/tracing, service discovery, the standard resilience HTTP
  handler, and `/health` and `/alive` health-check endpoints via
  `AddServiceDefaults()` and `MapDefaultEndpoints()`. Both ApiApp and WebApp
  reference this project and call these extension methods.

## Data and API Contract

- `data/HDBCarparkInformation.csv` — static HDB car park reference data
  (columns: `car_park_no`, `address`, `x_coord`, `y_coord`, `car_park_type`,
  `type_of_parking_system`, `short_term_parking`, `free_parking`,
  `night_parking`, `car_park_decks`, `gantry_height`, `car_park_basement`).
  `x_coord`/`y_coord` are SVY21 coordinates and are not yet WGS84
  latitude/longitude.
- `data/carpark-availability-sample.json` — a representative sample response
  from the data.gov.sg Car Park Availability API, keyed by
  `carpark_number`/`update_datetime`/`carpark_info` (with `total_lots`,
  `lot_type`, `lots_available` as strings).
- `data/CarparkAvailability.json` — the published data.gov.sg OpenAPI contract.
  Treat it as the source of truth for the live API shape.
- `data/carpark-availability.http` — a `.http` file with sample requests
  against `https://api.data.gov.sg/v1/transport/carpark-availability`, using a
  `DATA_GOV_SG_API_KEY` value from a local `.env`/`$dotenv` source. Never
  commit a populated `.env` file or a real key inside this file.

## Documentation

- `README.md` — workshop overview, curriculum, and starter structure.
- `IDEATION.md` — the approved product idea; source material for the future
  PRD, not an implemented feature list.
- `docs/00-setup.md` through `docs/04-create-canvas.md` — the ordered workshop
  steps (tooling and credential setup, `AGENTS.md` generation, PRD/TRD
  generation, application implementation, Copilot canvas).
- `docs/google-maps-api-key.md` and `docs/data-gov-sg-api-key.md` — credential
  setup guides; see [Secrets and API Boundaries](#secrets-and-api-boundaries).
- `CONTRIBUTING.md` — branch naming, commit convention, and PR expectations
  used across this repository.

## Technology Stack

- .NET 10 SDK (`global.json` pins `10.0.100`, `net10.0` target framework in
  `Directory.Build.props`), with nullable reference types and implicit usings
  enabled solution-wide.
- .NET Aspire (`Aspire.AppHost.Sdk/13.4.6`) for orchestration, service
  discovery, and resilience.
- ASP.NET Core minimal APIs (ApiApp) and ASP.NET Core Blazor with interactive
  server components (WebApp).
- Fluent UI Blazor components (`Microsoft.FluentUI.AspNetCore.Components`).
- OpenTelemetry (logging, metrics, tracing) via `ServiceDefaults`.
- NuGet package versions are centrally managed in `Directory.Packages.props`
  (`ManagePackageVersionsCentrally`); add new package versions there rather
  than pinning versions in individual `.csproj` files.

## Verified Commands

Run these from the repository root. They have been executed against this
starter and succeed:

```bash
dotnet restore CarparkAvailability.slnx
dotnet build CarparkAvailability.slnx --no-restore --configuration Release
```

Run the full application through Aspire (requires the Aspire CLI, a container
runtime, and both API keys configured as user secrets):

```bash
aspire run
```

The CI workflow (`.github/workflows/ci.yml`) only runs `dotnet restore` and
`dotnet build --no-restore --configuration Release` on push/PR to `main`. No
automated test command exists yet. Do not invent or reference a `dotnet test`
target until test projects are actually added to the solution.

## Secrets and API Boundaries

- Never commit API keys, `.env` files, or other credentials. `GoogleMaps:ApiKey`
  and `DataGovSg:ApiKey` must be stored only in AppHost user secrets
  (`dotnet user-secrets set ... --project src/CarparkAvailability.AppHost`), as
  described in `docs/google-maps-api-key.md` and
  `docs/data-gov-sg-api-key.md`.
- `DataGovSg:ApiKey` must remain server-side inside ApiApp. WebApp and the
  browser must never call `data.gov.sg` directly or receive this key; all
  parking-data requests from the browser must go through ApiApp.
- `GoogleMaps:ApiKey` is intentionally exposed to the browser for Maps
  JavaScript API usage, but must be restricted (HTTP referrer and API
  restrictions) as described in `docs/google-maps-api-key.md`.
- Do not log secret values, and do not add configuration files or samples that
  contain populated key values.

## Tests and Documentation

- No test projects exist in this starter yet. When application behavior is
  implemented, add tests and follow the process defined by the eventual
  `PRD.md` and `TRD.md` (created in the PRD/TRD workshop step) rather than
  inventing test scope or frameworks now.
- Keep `README.md`, workshop `docs/*.md`, and any future `PRD.md`/`TRD.md` in
  sync with implemented behavior. Update documentation whenever behavior or
  setup steps change.
- Do not implement parking features, tests, or the internal ApiApp/WebApp
  contract as part of generating or updating this file.

## Commit and Pull Request Guidance

Follow `CONTRIBUTING.md`:

- Branch names start with `feat/`, `fix/`, `docs/`, or `chore/`.
- Use [Conventional Commits](https://www.conventionalcommits.org/) prefixes:
  `feat:`, `fix:`, `docs:`, `test:`, `refactor:`, `chore:`.
- Keep changes focused; commit each validated, coherent step before starting
  the next.
- Run `dotnet restore CarparkAvailability.slnx` and
  `dotnet build CarparkAvailability.slnx --no-restore --configuration Release`
  before committing.
- Open a pull request that links the related issue and describes the change.
