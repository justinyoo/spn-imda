# Smart Parking Navigator PRD

## Document Metadata

| Field | Value |
| --- | --- |
| Product | Smart Parking Navigator |
| Repository | `justinyoo/spn-imda` |
| Document status | Draft for review |
| Current workflow status | PRD generated; TRD must not be generated until this PRD is approved |
| Source material | `IDEATION.md`, `docs/02-generate-prd-trd.md`, prepared `data/` files, credential setup guides, scaffolded Aspire solution |
| Last updated | 2026-08-18 |

## 1. Product Summary

Smart Parking Navigator is a workshop-sized web MVP for drivers looking for HDB
car parks near a Singapore destination. A user searches for a destination,
reviews nearby HDB car parks on a map, compares live lot availability and
parking conditions, and chooses a suitable car park with clear visibility into
data freshness.

The MVP is intentionally limited to the prepared .NET Aspire scaffold and the
provided Singapore datasets. It defines product requirements only; application
behavior, API contracts, implementation details, and tests are not implemented
by this document.

## 2. Target Users

- **Occasional drivers in Singapore** who need to park near an HDB estate,
  market, hawker centre, clinic, or other local destination.
- **Regular commuters and residents** who compare nearby HDB car parks before
  driving to a familiar area.
- **Drivers of vehicles with constraints** such as vehicle type, height, or
  night-parking needs.
- **Workshop participants** who need a realistic but bounded product scope for
  implementing the Aspire, API, and Blazor application in later steps.

## 3. Singapore Context

- The MVP covers **Singapore HDB car parks** using the prepared HDB Car Park
  Information CSV and data.gov.sg Car Park Availability API contract.
- Destination search is restricted to Singapore addresses or places.
- Nearby car parks are limited to HDB car parks within **500 metres** of the
  selected destination.
- Time-sensitive display and freshness interpretation use Singapore Standard
  Time where user-facing time interpretation is required.
- The application must account for Singapore-specific HDB parking attributes,
  including short-term parking, free parking, night parking, parking system,
  car-park type, deck count, gantry height, and basement status.

## 4. Goals and Success Measures

### Goals

1. Help users find a suitable available HDB car park near a destination within
   seconds.
2. Present live availability, distance, occupancy, and data freshness in a way
   that supports quick decisions.
3. Keep the MVP achievable within the workshop by using only the prepared
   scaffold, Google Maps integration, HDB CSV, and data.gov.sg availability
   source.
4. Preserve clear frontend/backend credential boundaries.

### Success Measures

- A user can search for a Singapore destination and view nearby HDB car parks
  within 500 metres.
- Results show available and total lots by supported vehicle type when live data
  is available.
- Results visibly distinguish fresh, stale, unavailable, loading, empty, and
  error states.
- Users can filter results by availability, vehicle type, night parking, and
  car-park type.
- The product avoids out-of-scope infrastructure such as deployment, databases,
  MCP, and agentic AI.

## 5. User Journeys

### Journey 1: Find parking near a destination

1. The user opens the app.
2. The user searches for a Singapore address or destination.
3. The app displays the selected destination on a map.
4. The app lists HDB car parks within 500 metres.
5. The user compares distance, available lots, total lots, occupancy, and latest
   update time.
6. The user selects a suitable car park and reviews its details.

### Journey 2: Avoid full or unsuitable car parks

1. The user searches for a destination.
2. The app returns nearby car parks and live availability.
3. The user enables an available-only filter and selects a supported vehicle
   type.
4. The app hides car parks without available lots for that vehicle type.
5. If a selected or preferred car park is full, the app makes nearby available
   alternatives easy to identify.

### Journey 3: Check operating and vehicle constraints

1. The user opens a car park details view.
2. The app displays static HDB attributes such as address, car-park type,
   parking system, short-term parking, free parking, night parking, deck count,
   gantry height, and basement status.
3. The user uses details and filters to exclude car parks that do not meet their
   practical needs.

### Journey 4: Understand data freshness

1. The app loads live availability from data.gov.sg through the backend.
2. The user sees the source update time for each result.
3. The app clearly labels stale or unavailable data.
4. When live retrieval fails but previous data exists, the user can tell the
   displayed data is last-known information rather than a fresh reading.

## 6. MVP Scope

### P0 Requirements

| ID | Requirement | Acceptance criteria |
| --- | --- | --- |
| P0-1 | Search for a Singapore destination. | Users can enter an address or place query; searches are constrained to Singapore; invalid or unresolvable searches produce a clear empty or error state. |
| P0-2 | Show nearby HDB car parks. | Results include only matched HDB car parks within 500 metres of the selected destination; each result includes distance from the destination. |
| P0-3 | Show live lot availability. | For each matched result with live data, users can see available lots, total lots, occupancy rate, supported lot type, and source update time. |
| P0-4 | Support basic filters. | Users can filter by available-only, vehicle lot type, night parking, and car-park type without losing the selected destination context. |
| P0-5 | Show car park details. | Users can view address, car-park type, parking system, short-term parking, free parking, night parking, deck count, gantry height, and basement status for a selected result when present in HDB data. |
| P0-6 | Communicate freshness states. | Loading, fresh, stale, unavailable, last-known-good, empty, and error states are distinguishable in the UI. |
| P0-7 | Preserve credential boundaries. | Google Maps API key is used only for browser map/search needs with appropriate restrictions; data.gov.sg API key remains server-side and is not sent to the browser or logged. |

### P1 Requirements

| ID | Requirement | Acceptance criteria |
| --- | --- | --- |
| P1-1 | Rank recommendations. | Results are ordered using distance, availability, and occupancy so that likely useful car parks appear first. |
| P1-2 | Highlight alternatives. | When a car park has no available lots for the selected vehicle type, nearby alternatives with availability are easy to identify. |
| P1-3 | Refresh results from map movement. | If the user moves the map after selecting a destination, the app can refresh visible results while preserving the 500-metre destination-based MVP constraint. |

## 7. Data Requirements

- Static car park reference data comes from `data/HDBCarparkInformation.csv`.
- Live availability data comes from the data.gov.sg Car Park Availability API
  described by `data/CarparkAvailability.json` and represented by
  `data/carpark-availability-sample.json`.
- Static and live records are matched by HDB `car_park_no` and data.gov.sg
  `carpark_number`.
- HDB `x_coord` and `y_coord` values are SVY21 coordinates and must be
  converted before map display in a later technical design.
- data.gov.sg numeric lot values are strings and must be handled safely in the
  later implementation.
- Lot types shown by the product are the supported API values:
  - `C` — cars
  - `H` — heavy vehicles
  - `S` — motorcycles with side car
  - `Y` — motorcycles
- The product must handle car parks that have static data but no live data, live
  data but no static match, or malformed/unavailable live values.

## 8. UX and State Requirements

- The primary experience is mobile-first but usable on desktop.
- The map and list should work together: selecting a map marker or list item
  reveals the same car park details.
- The app must not infer undocumented meanings for free-parking or
  short-term-parking text; values should be displayed plainly unless a later
  approved requirement defines interpretation rules.
- Empty states should distinguish between no destination selected, no matching
  destination, and no HDB car parks within 500 metres.
- Error states should avoid exposing secrets, raw stack traces, or confusing
  implementation details.

## 9. Non-Goals and Exclusions

The MVP explicitly excludes:

- User accounts, saved profiles, favourites, and personalized vehicle settings.
- Availability alerts and notifications.
- Historical storage, databases, analytics warehouses, and occupancy
  forecasting.
- Traffic, routing, weather, camera, or incident integrations.
- Payments, booking, season parking, or enforcement workflows.
- Deployment architecture, cloud hosting, CI/CD design beyond existing build
  expectations, or production operations.
- MCP servers, Copilot canvas implementation, and agentic AI features.
- Any application behavior or internal API contract changes as part of creating
  this PRD.

## 10. Security, Privacy, and Compliance Requirements

- Do not commit API keys, `.env` files, credentials, or populated secret
  examples.
- The data.gov.sg API key is confidential and must remain backend-only.
- The Google Maps API key is browser-exposed by design and must be configured
  with website and API restrictions.
- User-entered search text must be treated as untrusted input in later
  implementation.
- Logs and user-facing errors must not include secrets.
- The product uses public open datasets and does not require collecting personal
  data for the MVP.

## 11. Workshop Acceptance Criteria

- `PRD.md` defines target users, the Singapore context, user journeys, MVP
  scope, exclusions, and measurable acceptance criteria.
- The MVP covers destination search, nearby HDB car parks, live lot
  availability, basic filters, details, and freshness states.
- Google Maps and data.gov.sg credentials remain in their appropriate
  frontend/backend boundaries.
- Deployment, databases, MCP, and agentic AI are explicitly out of scope.
- `TRD.md` is intentionally not created until this PRD is reviewed and approved.

