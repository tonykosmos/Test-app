# Quickstart: Visited Places Map

Validation guide for `specs/001-visited-places-map`. It checks the running app against the spec. It is not the implementation.

## Prerequisites

- Node.js 20+, npm, and Docker.
- A Mapbox account with two tokens:
  - Public token in `frontend/.env` as `VITE_MAPBOX_PUBLIC_TOKEN`, restricted to `http://localhost:5173` and allowed to read styles, tiles, and fonts.
  - Server token in `backend/.env` as `MAPBOX_ACCESS_TOKEN`, allowed to call Search Box and Geocoding. Do not put this token in the frontend.
- Dependencies installed from the repository root with `npm install`.

## Start

```powershell
docker compose up -d
npm run db:migrate
npm run dev:backend
npm run dev:frontend
```

Use a second terminal for the frontend. If port 3000 is already taken, set `PORT` in `backend/.env` and start the frontend with `API_PROXY_TARGET` pointing at that port.

- App: http://localhost:5173
- API contract: [contracts/places.openapi.yaml](contracts/places.openapi.yaml)
- Fields: [data-model.md](data-model.md)

Automated checks, once the feature is implemented:

```powershell
npm test -w backend
npm test -w frontend
```

Backend tests must not call Mapbox. Frontend tests must not require a live map canvas.

## Scenarios

### 1. Add a place by search

1. Open the app. The map shows a world view and a prompt to add the first place.
2. Search for `Eiffel Tower`.
3. Choose the Paris result. The title can be `First trip to Paris`. Save.
4. Expected: a pin sits on the tower. The card shows the title and Paris, France. Reload. The pin is still there.

### 2. Add a place by clicking

1. Click an empty area of ocean, or another unnamed spot.
2. Enter a title and save.
3. Expected: the pin is exactly where you clicked. City and country may be empty. Saving with an empty title is refused.

### 3. Dates and description

1. Add a place with description text and a year-only visit of 2024.
2. Expected: the card shows 2024, not 1 January 2024. A future year or date is refused.

### 4. Edit and delete

1. Change the title and drag or replace the pin location. Save.
2. Expected: the new title and location survive a reload.
3. Delete the place and confirm.
4. Expected: it disappears from the map and the list. Canceling the confirmation leaves it in place.

### 5. List

1. Create places visited in 2022, 2024, and 2025, plus one with no date.
2. Expected order: 2025, 2024, 2022, then the undated place.
3. Choose the 2022 row.
4. Expected: the map centers on it and opens its card.

### 6. Overlapping pins

1. Save two places at the same coordinates.
2. Zoom out until they overlap.
3. Expected: one marker shows `2`. Zooming in, or choosing that marker, shows both pins.

### 7. Search failure

1. Search for a string that returns no suggestions.
2. Expected: the app says nothing was found, and clicking the map still saves a place.
3. Stop the network and search again.
4. Expected: a clear message that search is unavailable. Already saved places still load.
