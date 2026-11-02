# Test App

Travel planning app. This repository is the project setup only: the frontend, the backend, PostgreSQL, and Spec Kit. Feature behavior will be added from specifications in `specs/`.

## Prerequisites

- Node.js 20 or newer
- npm 11 or newer
- Docker, for PostgreSQL

## Install

From the repository root:

```powershell
npm install
```

One install covers `frontend`, `backend`, and `packages/shared`.

## Database

Start PostgreSQL:

```powershell
docker compose up -d
```

The container uses the same defaults as `backend/.env.example` (`postgres` / `postgres`, database `test_app`, port `5432`).

To keep a local copy of those settings:

```powershell
Copy-Item backend\.env.example backend\.env
```

## Start

The frontend and backend start separately. Use two terminals from the repository root:

```powershell
npm run dev:backend
```

```powershell
npm run dev:frontend
```

You can also start each app from its own folder:

```powershell
cd backend
npm run start:dev
```

```powershell
cd frontend
npm run dev
```

- Frontend: http://localhost:5173
- API: http://localhost:3000/api
- API docs: http://localhost:3000/api/docs

The API currently responds to `GET /api` with the Nest starter message. That confirms the server and the database connection. There are no product features yet.

Apply database migrations (none are defined yet):

```powershell
npm run db:migrate
```

## Lint and format

```powershell
npm run lint
npm run format
```

## Layout

| Path               | Purpose                                                                            |
| ------------------ | ---------------------------------------------------------------------------------- |
| `frontend/`        | Vite, React, TypeScript, Tailwind                                                  |
| `backend/`         | NestJS, TypeORM, Swagger                                                           |
| `packages/shared/` | Types shared by both apps, empty for now                                           |
| `specs/`           | One directory per feature. The structure inside each feature will be defined later |
| `.specify/`        | Spec Kit templates, scripts, and project memory                                    |
| `.cursor/rules/`   | Spec Kit rule loaded by Cursor                                                     |
| `.cursor/skills/`  | Spec Kit skills (`/speckit-specify`, `/speckit-plan`, and the rest)                |

## Spec Kit

Spec Kit is initialized for Cursor. In chat you can run `/speckit-constitution`, `/speckit-specify`, `/speckit-plan`, `/speckit-tasks`, and `/speckit-implement`.

Feature specifications belong in `specs/`. Do not invent a feature folder layout until that structure is defined.
