# Research: Visited Places Map

## Map rendering

- **Decision**: Render the map with Mapbox GL JS in the browser. Show places as a GeoJSON source with `cluster: true`, `clusterRadius: 50`, and `clusterMaxZoom: 14`. A cluster layer shows `point_count`. Clicking a cluster zooms in with `getClusterExpansionZoom`. Individual pins open the place card. On load, `fitBounds` covers every saved coordinate. With no places, the map opens on a world view.
- **Rationale**: The user chose Mapbox. GL JS already clusters by screen distance and can fit the camera to the pins, which matches the spec without a second mapping library.
- **Alternatives considered**: Leaflet or MapLibre would avoid a Mapbox tile token, but the map provider is already chosen. Supercluster as a separate library duplicates what GL JS does.

## Place search

- **Decision**: Search goes through the backend to the Mapbox Search Box API: `GET /search/searchbox/v1/suggest` while the traveler types, then `GET /search/searchbox/v1/retrieve/{mapbox_id}` when they pick a result. The browser sends one UUIDv4 `sessionToken` for the whole suggest-and-retrieve sequence. The backend forwards it and never returns the raw Mapbox payload.
- **Rationale**: The spec's example search is "Eiffel Tower". Mapbox Geocoding API v6 no longer returns points of interest and tells callers to use Search Box for those. Suggest results have no coordinates; retrieve supplies them, plus `context.place` and `context.country`.
- **Alternatives considered**: Geocoding v6 forward search. Rejected because landmark and POI queries, which this feature is for, are outside that API. Calling Search Box directly from the browser would expose a token that is not URL-restricted.

## Click-to-place names

- **Decision**: Clicking the map always sets the pin coordinates. The backend then calls Geocoding v6 reverse (`/search/geocode/v6/reverse`) and fills place name, city, and country when Mapbox returns them. An empty result is valid: the pin is still saved with only the title and coordinates.
- **Rationale**: The spec requires city and country when a click matches a known location, and it explicitly allows unnamed locations. Reverse geocoding answers "what is here?" without POI data. Search Box is not needed for that question.
- **Alternatives considered**: Leaving every clicked pin unnamed. That fails the requirement to record a known location. Using Search Box reverse for the same job adds a second session-billed API for a lookup Geocoding v6 already covers.

## Tokens

- **Decision**: Two tokens. `VITE_MAPBOX_PUBLIC_TOKEN` is a public token used only by Mapbox GL JS, restricted to the frontend origins and to the styles, tiles, and fonts scopes. `MAPBOX_ACCESS_TOKEN` lives only in `backend/.env` and is used for Search Box and Geocoding. It is not shipped to the browser. Requests whose place name, city, or country will be stored use Mapbox's permanent-results flag.
- **Rationale**: A URL-restricted public token cannot call Mapbox from the server, because the server has no browser origin. A secret or unrestricted server token must not be embedded in the frontend.
- **Alternatives considered**: One public token for both. Rejected because URL restrictions and server-side calls conflict, and a leaked search token can create billing charges.

## Stored visit dates

- **Decision**: Store `visited_on` as a date plus `visited_on_precision` of `day`, `month`, or `year`. A month is stored as the first of that month, and a year as 1 January of that year. The interface shows only the precision the traveler entered. Sorting uses `visited_on` descending, with undated places last. A future calendar day is rejected. Year-only and month-only values are future when that year or month has not started.
- **Rationale**: One date column sorts correctly, and the precision column stops a year-only visit from being displayed as 1 January.
- **Alternatives considered**: Separate nullable year, month, and day columns. They make sorting and validation more branching for the same result.

## Ownership without accounts

- **Decision**: A migration inserts one traveler, id `00000000-0000-4000-8000-000000000001`, display name "Demo". Every place request uses that id on the server. The client cannot send a traveler id.
- **Rationale**: The spec forbids sign-in in this version and still requires every place to remember its owner for a later accounts feature.
- **Alternatives considered**: A nullable `traveler_id`. That would have to be backfilled when accounts arrive and breaks the "exactly one traveler" rule.

## Tests

- **Decision**: Backend Jest tests cover validation, CRUD, date rules, and the demo-traveler scope, with Mapbox replaced by a fake client. Frontend Vitest and Testing Library cover the form, the empty state, the list order, and unsaved-change confirmation. The map component is tested through the coordinates it emits, not by clicking the WebGL canvas. `quickstart.md` is the manual check for the real Mapbox map.
- **Rationale**: The constitution requires automated coverage of acceptance scenarios. A real Mapbox canvas is slow and brittle in CI, while the behavior that matters is the data the map hands to the form.
- **Alternatives considered**: Playwright against a live Mapbox account. Rejected for this feature because it needs a paid token in CI and fails when Mapbox is down.

## Scope boundary

- **Decision**: No photo upload, storage, or metadata handling. No PostGIS. No pagination. All of one traveler's places are returned in one list. Five hundred pins is the stated performance target and is small enough for one response plus client-side clustering.
- **Rationale**: Photos were deferred. A geographic index and pages solve problems this version does not have.
- **Alternatives considered**: Adding an empty `photos` table now. Rejected by the simplicity principle.
