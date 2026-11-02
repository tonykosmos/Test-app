# Feature Specification: Visited Places Map

**Feature Branch**: `001-visited-places-map`

**Created**: 2026-11-02

**Status**: Draft

**Input**: User description: "Interactive map with points on places that you visited before, some pics and small description."

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Pin a visited place on the map (Priority: P1)

A traveler opens their personal map and marks a place they have been to. They choose the location by searching for a place name or by clicking directly on the map, give it a title, and optionally add when they visited and a short description. The new pin appears on the map immediately and is still there the next time they open the app.

**Why this priority**: The pin on the map is the core of the feature. Without it there is nothing to attach photos or descriptions to, and a map of pins alone already gives the traveler a personal record of where they have been.

**Independent Test**: Add three places (two by search, one by clicking the map), reload the app, and confirm all three pins are shown in the right locations with their titles.

**Acceptance Scenarios**:

1. **Given** a traveler on their map, **When** they search for "Eiffel Tower", pick the result, enter the title "First trip to Paris", and save, **Then** a pin appears at the Eiffel Tower with that title.
2. **Given** a traveler on their map, **When** they click an unnamed spot on a beach, enter a title, and save, **Then** a pin appears at exactly that spot.
3. **Given** a traveler who has saved pins, **When** they close the app and open their map later, **Then** all their pins are shown where they placed them.
4. **Given** a traveler filling in a new pin, **When** they try to save without a title, **Then** the pin is not saved and they are told a title is required.

---

### User Story 2 - Add photos and a description to a place (Priority: P2)

When adding or editing a place, the traveler attaches a few photos and writes a short note about the visit. Selecting the pin later opens a card with the title, visit date, description, and photos, and the traveler can open any photo at full size.

**Why this priority**: Photos and notes turn a list of coordinates into memories, which is the main reason a traveler would come back to this map. It depends on pins existing (User Story 1).

**Independent Test**: Create a pin with a description and three photos, then select the pin and confirm the card shows the description and all three photos in the order they were added, and that each photo opens at full size.

**Acceptance Scenarios**:

1. **Given** a traveler adding a place, **When** they attach three photos and a description and save, **Then** selecting the pin shows the description and the three photos.
2. **Given** a place with photos, **When** the traveler selects a photo on the place card, **Then** the photo opens at full size and they can move to the next and previous photos.
3. **Given** a place that already has the maximum number of photos, **When** the traveler tries to attach another photo, **Then** the photo is not added and they are told the limit.
4. **Given** a traveler attaching a file that is not a supported image or is too large, **When** they try to save, **Then** that file is rejected with a clear reason and the rest of the place is still saved.

---

### User Story 3 - Edit or remove a place (Priority: P3)

The traveler fixes mistakes or changes their mind: they move a pin, change its title, date, or description, add or remove photos, or delete the place entirely.

**Why this priority**: Travelers will make mistakes when placing pins, and they need a way to correct them. Without it, the map becomes less accurate and less trusted over time, but the feature is still usable without it for a first demo.

**Independent Test**: Edit an existing place (new title, moved pin, one photo removed), confirm the changes are shown, then delete the place and confirm it and its photos are gone from both the map and the list.

**Acceptance Scenarios**:

1. **Given** an existing place, **When** the traveler changes its title and moves the pin to a new spot and saves, **Then** the pin shows the new title at the new location.
2. **Given** a place with photos, **When** the traveler removes one photo and saves, **Then** that photo no longer appears on the place card.
3. **Given** an existing place, **When** the traveler chooses delete, **Then** they are asked to confirm before anything is removed.
4. **Given** the traveler confirmed deletion, **When** the deletion completes, **Then** the place and all its photos are permanently removed and no longer appear anywhere in the app.

---

### User Story 4 - Browse places as a list (Priority: P4)

Besides the map, the traveler can see all their places as a list, sorted by visit date with the most recent first. Selecting a list entry shows that place on the map and opens its card.

**Why this priority**: Once a traveler has dozens of places, finding one by panning the map is slow. The list makes the collection easy to scan, but the map alone is enough while collections are small.

**Independent Test**: Create places with different visit dates, open the list, confirm the order, and select an entry to confirm the map moves to that place and opens its card.

**Acceptance Scenarios**:

1. **Given** a traveler with places visited in 2022, 2024, and 2025, **When** they open the list, **Then** the places appear in the order 2025, 2024, 2022.
2. **Given** a place without a visit date, **When** the traveler opens the list, **Then** that place appears after all places that have a visit date.
3. **Given** the list is open, **When** the traveler selects a place, **Then** the map centers on that place and opens its card.

### Edge Cases

- **Many pins close together**: When pins overlap at the current zoom level, they are grouped into a single marker showing how many places it contains. Zooming in or selecting the group reveals the individual pins.
- **Search finds nothing**: The traveler is told no place was found and can still place the pin by clicking the map.
- **Location with no name**: Places such as a spot in the mountains or at sea are allowed. The traveler's title is enough to identify them.
- **Same place visited more than once**: The traveler can add another pin at the same location, for example one per visit. Both are shown, grouped when they overlap.
- **Visit date in the future**: The date is rejected, because this map is only for places already visited.
- **Partial visit date**: The traveler can enter only a year, or a month and year, when they do not remember the exact day.
- **Photo upload interrupted**: If saving fails part way, the traveler is told which photos were not saved and can retry them without re-entering the rest of the place.
- **Leaving with unsaved changes**: If the traveler closes the add or edit form with unsaved changes, they are asked whether to discard them.
- **Empty map**: A traveler with no places sees a short prompt explaining how to add their first place.

## Requirements *(mandatory)*

### Functional Requirements

**Places**

- **FR-001**: Travelers MUST be able to add a place by searching for a place name or address and choosing a result.
- **FR-002**: Travelers MUST be able to add a place by clicking any point on the map.
- **FR-003**: Each place MUST have a title of 1 to 100 characters.
- **FR-004**: Each place MAY have a visit date, given as a full date, a month and year, or a year only. The date MUST NOT be in the future.
- **FR-005**: Each place MAY have a description of up to 1,000 characters.
- **FR-006**: When a place is chosen from search results or matches a known location, the system MUST record its place name, city, and country so they can be shown on the place card. These fields MAY be empty for unnamed locations.
- **FR-007**: Travelers MUST be able to edit every field of a place they own, including moving its pin.
- **FR-008**: Travelers MUST be able to delete a place they own, after confirming. Deleting a place MUST permanently remove its photos.

**Photos**

- **FR-009**: Travelers MUST be able to attach up to 5 photos to a place.
- **FR-010**: The system MUST accept JPEG, PNG, WebP, and HEIC photos of up to 10 MB each, and MUST reject other files with a message that names the file and the reason.
- **FR-011**: Photos MUST be shown in the order the traveler added them, and the traveler MUST be able to remove individual photos.
- **FR-012**: Photos MUST be shown upright, respecting the orientation the camera recorded.
- **FR-013**: Stored photos MUST NOT keep embedded location or device metadata, so the only location saved is the one the traveler chose for the pin.

**Map and list**

- **FR-014**: The map MUST show all of the traveler's places as pins and let the traveler pan and zoom.
- **FR-015**: Pins that overlap at the current zoom level MUST be grouped into one marker that shows the number of places it contains.
- **FR-016**: Selecting a pin MUST open a card with the place's title, visit date, place name, description, and photos.
- **FR-017**: When the map opens, it MUST be positioned so that all of the traveler's places are visible. A traveler with no places sees a world view and a prompt to add their first place.
- **FR-018**: Travelers MUST be able to view their places as a list, sorted by visit date with the most recent first and undated places last.
- **FR-019**: Selecting a place in the list MUST center the map on it and open its card.

**Ownership**

- **FR-020**: This version MUST work without sign-up or sign-in. All places and photos belong to one built-in demo traveler.
- **FR-021**: Every place and photo MUST still record which traveler owns it, so that a later accounts feature can give each traveler their own private map without losing existing places.

### Key Entities *(include if feature involves data)*

- **Traveler**: The person who owns a personal map. Every visited place and photo belongs to exactly one traveler. In this version there is a single built-in demo traveler.
- **Visited Place**: One pinned location the traveler has been to. It has a title, a map location, an optional place name with city and country, an optional visit date with its precision (day, month, or year), an optional description, and the dates it was created and last changed. A traveler has any number of visited places.
- **Place Photo**: An image attached to a visited place, with its position in the place's photo order. A visited place has up to 5 photos. Photos are deleted together with their place.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: A traveler can add a new place with a title, a description, and one photo in under 2 minutes.
- **SC-002**: For a traveler with 500 places, the map shows all pins within 3 seconds of opening, and panning and zooming stay smooth.
- **SC-003**: In usability testing, at least 9 out of 10 travelers add their first place without help.
- **SC-004**: In usability testing, travelers find and open a specific place they added earlier within 15 seconds, using either the map or the list.
- **SC-005**: A 10 MB photo is saved and visible on the place card within 10 seconds on a typical home broadband connection.

## Assumptions

- User accounts are out of scope. Until an accounts feature exists, the app runs for one demo traveler, is used locally or in a private environment, and anyone who can open the app can see and change all places. It must not be deployed publicly in this state.
- The first version is a personal map. Sharing places or a public map, a "want to go" wishlist, linking places to planned trips, travel statistics, reading location from photo metadata, adding places through Telegram, and exporting places to PDF are out of scope and can be specified as separate features.
- A place is a single point on the map. Marking whole countries or regions as visited is out of scope.
- The app is used in current desktop and mobile web browsers. A native mobile app is out of scope.
- Place search and showing place names use an external map and place-search service. Its search results may be incomplete in some regions, which is why clicking the map is always available.
- Photos are stored at a quality good enough to view full screen on a phone or laptop. Keeping original files unchanged is not required.
- Data is kept until the traveler deletes it. Deleting a traveler account and its data will be handled by the future accounts feature.
- The interface is in English for the first version.
