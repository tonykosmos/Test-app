# Implementation Plan: Visited Places Map

**Branch**: `001-visited-places-map` | **Date**: 2026-11-02 | **Spec**: [spec.md](spec.md)

**Input**: Feature specification from `/specs/001-visited-places-map/spec.md`

## Summary

Travelers pin places they have already visited on a personal map, with a title, an optional visit date, and an optional description. Photos, accounts, and sharing are out of scope.

The map is Mapbox GL JS. Typing a place searches the Mapbox Search Box API through the NestJS backend, because Geocoding v6 does not return landmarks. A map click saves those coordinates and, when Mapbox knows the location, the place name, city, and country. Places belong to one seeded demo traveler and are stored in PostgreSQL through a TypeORM migration.

## Technical Context

**Language/Version**: TypeScript 5 on the backend (Node.js 20+) and TypeScript 6 on the frontend

**Primary Dependencies**: NestJS 11, TypeORM 0.3, Mapbox GL JS, Mapbox Search Box API, Mapbox Geocoding API v6 (reverse only), React 19, Vite, Tailwind CSS

**Storage**: PostgreSQL 16. Tables `travelers` and `visited_places`. No file storage.

**Testing**: Jest and Supertest on the backend. Vitest and Testing Library on the frontend. Mapbox is faked in automated tests.

**Target Platform**: Current desktop and mobile browsers for the UI. The API runs on the local NestJS server.

**Project Type**: Web application, one npm workspace, frontend and backend started separately.

**Performance Goals**: A traveler with 500 places sees every pin within 3 seconds of opening the map, and pan and zoom stay smooth. Adding a titled place with a description takes under 2 minutes.

**Constraints**: No sign-in. No photos. `synchronize` stays false. The server Mapbox token never reaches the browser. The app is for local or private use until accounts exist.

**Scale/Scope**: One demo traveler and hundreds of places. Three user stories: add a pin, edit or delete it, browse a list.

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

| Principle | Gate | Result |
| --- | --- | --- |
| I. Spec Before Code | Plan covers only the current spec. Photos stay a later feature. | Pass |
| II. Vertical Slices | Story 1 can ship as map, search, create, and reload. Edit and list follow. | Pass |
| III. Tested User Stories | Each story has automated coverage listed below. The live map is checked with the quickstart. | Pass |
| IV. Explicit Schema Changes | One migration creates and seeds the tables. Shared place types go in `packages/shared`. | Pass |
| V. Simplicity | No accounts, photo storage, PostGIS, or pagination. Search is proxied only because the token and the POI API require it. | Pass |

Post-design re-check: pass. The data model, contract, and quickstart add no extra features and no constitution exceptions.

## Project Structure

### Documentation (this feature)

```text
specs/001-visited-places-map/
├── plan.md
├── research.md
├── data-model.md
├── quickstart.md
├── contracts/
│   └── places.openapi.yaml
└── tasks.md              # created later by /speckit-tasks
```

### Source Code (repository root)

```text
packages/shared/src/
└── visited-place.ts       # place, suggestion, and date-precision types

backend/src/
├── places/
│   ├── dto/
│   ├── entities/
│   ├── places.controller.ts
│   ├── places.service.ts
│   ├── places.module.ts
│   └── places.service.spec.ts
├── place-search/
│   ├── mapbox-place-search.client.ts
│   ├── place-search.controller.ts
│   ├── place-search.service.ts
│   └── place-search.module.ts
├── migrations/
│   └── <timestamp>-CreateVisitedPlaces.ts
└── database/demo-traveler.ts

backend/test/
└── places.e2e-spec.ts

frontend/src/features/visited-places/
├── api.ts
├── VisitedPlacesPage.tsx
├── PlaceMap.tsx
├── PlaceForm.tsx
├── PlaceList.tsx
├── PlaceCard.tsx
└── *.test.tsx

frontend/
└── .env.example           # VITE_MAPBOX_PUBLIC_TOKEN only
```

**Structure Decision**: This is the existing `frontend/` and `backend/` workspace. The feature is a Nest module pair (`places`, `place-search`) and one frontend feature folder. Shared request and response types live in `packages/shared`, not copied into either app.

## Implementation notes

- **Demo owner**: migration inserts traveler `00000000-0000-4000-8000-000000000001`. Services take that id from configuration, never from the request body.
- **API**: CRUD under `/api/places` and search under `/api/place-search`, as in [contracts/places.openapi.yaml](contracts/places.openapi.yaml). DTOs use `class-validator`. A place id that is missing or belongs to anyone else returns 404.
- **Search**: the client creates one UUIDv4 session token per search box session. Suggest returns names only. Retrieve and reverse return name, city, country, and coordinates. Mapbox errors become HTTP 502 with a short message. An empty reverse result is HTTP 200 `null`.
- **Map**: `PlaceMap` loads Mapbox GL with `VITE_MAPBOX_PUBLIC_TOKEN`. Pins are GeoJSON with clustering. The component reports clicks and pin selection upward. It does not call Mapbox search itself.
- **Dates**: stored as `visited_on` plus `visited_on_precision` (`day`, `month`, `year`). Details are in [data-model.md](data-model.md).
- **Unsaved edits**: closing the form with dirty fields asks before discarding.
- **Tests**:
  - Story 1: API creates a place from coordinates, rejects a blank title and a future date, reloads it, and the form shows a search miss without blocking a map click.
  - Story 2: API updates coordinates and title, delete requires the UI confirmation and then returns 204, and a second delete returns 404.
  - Story 3: API order matches the sort rules, and choosing a list row centers the map.

## Complexity Tracking

No constitution violations.
