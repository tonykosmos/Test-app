<!--
Sync Impact Report
- Version change: unratified template → 1.0.0
- Modified principles:
  - [PRINCIPLE_1_NAME] → I. Spec Before Code
  - [PRINCIPLE_2_NAME] → II. Vertical Slices
  - [PRINCIPLE_3_NAME] → III. Tested User Stories
  - [PRINCIPLE_4_NAME] → IV. Explicit Schema Changes
  - [PRINCIPLE_5_NAME] → V. Simplicity
- Added sections:
  - Technology Stack
  - Delivery Workflow
- Removed sections: none (placeholder headings were replaced in place)
- Follow-up TODOs: none
-->

# Test App Constitution

## Core Principles

### I. Spec Before Code

Every product feature MUST start as a specification under `specs/` and MUST
pass its requirements checklist before planning or implementation. Code MUST
NOT add behavior that the current feature spec does not require. Items listed
as out of scope MUST stay out of scope until they have their own spec.

Rationale: The product is specified one feature at a time. Building ahead of
the spec adds unused surface area and contradicts the agreed MVP.

### II. Vertical Slices

A feature MUST be delivered as independently testable user stories, in
priority order. The highest-priority story MUST work end to end, from stored
data through the interface, before later stories are started. A story MUST be
demonstrable on its own.

Rationale: An MVP that sketches every story but finishes none of them cannot
be shown or corrected early.

### III. Tested User Stories

Each user story MUST have automated tests that cover its acceptance scenarios
before that story is considered done. A change that breaks an existing
acceptance scenario MUST NOT be merged.

Rationale: Stories are the unit of delivery. Tests lock the behavior the spec
already described.

### IV. Explicit Schema Changes

Database schema changes MUST be committed TypeORM migrations. `synchronize`
MUST remain disabled. Types shared by the frontend and the backend MUST live
in `packages/shared` and MUST NOT be copied into either app.

Rationale: Automatic schema sync can drop columns. One shared contract keeps
both apps aligned.

### V. Simplicity

The project MUST prefer the smallest design that satisfies the current spec.
New libraries, services, and abstractions MUST be justified by a requirement
in that spec. Placeholder accounts, providers, or features MUST NOT be built
for later work inside the current feature.

Rationale: Accounts, the AI provider, and later travel features are
intentionally deferred. Anticipating them inside the current feature widens
the MVP.

## Technology Stack

The repository is one npm workspace. The frontend and the backend MUST start
as separate processes.

- Frontend: Vite, React, TypeScript, Tailwind CSS, ESLint, and Prettier.
- Backend: NestJS, TypeORM, and PostgreSQL. Request validation and OpenAPI
  docs are added when the feature spec introduces an API.
- Database: PostgreSQL, run locally with Docker Compose. Schema changes
  follow Principle IV.
- Shared code: `packages/shared`.
- AI: any provider MUST sit behind a single project interface and MUST be
  selectable without rewriting feature code. No provider is chosen until a
  spec requires generation.

Deviations MUST be recorded as a constitution amendment or called out in the
feature plan with a reason.

## Delivery Workflow

Work on a feature MUST follow this order:

1. Specification in `specs/NNN-short-name/spec.md`, with its quality
   checklist passing and no unresolved clarifications.
2. Implementation plan, checked against this constitution.
3. Tasks ordered by user-story priority.
4. Implementation of one story at a time, with that story's tests.

The frontend and the backend MUST stay in this repository. Feature code MUST
be grouped by feature, not in one global layer that mixes unrelated features.

## Governance

This constitution supersedes informal preferences when they conflict. Plans
and pull requests MUST be checked against these principles.

Amendments MUST update this file, bump the version below using semantic
versioning, and set the last-amended date. A MAJOR bump removes or redefines
a principle. A MINOR bump adds a principle or section. A PATCH bump clarifies
wording only.

Complexity beyond the current spec MUST be justified in the feature plan. How
to install and run the project lives in the repository README.

**Version**: 1.0.0 | **Ratified**: 2026-11-02 | **Last Amended**: 2026-11-02
