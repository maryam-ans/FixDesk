# FixDesk

## Project Name

FixDesk — campus and hostel issue tracking.

## Problem Statement

Campus complaints are often sent in informal chats, where they lose their owner, deadline and visible progress. FixDesk gives each issue an accountable technician and an SLA, and escalates overdue work.

## Target Users

- **Students** report and track issues, comment, confirm a fix or reopen it.
- **Staff** see work in their categories, act on assigned tickets and record resolution notes.
- **Administrators** manage categories and technicians, reassign tickets and monitor campus service levels.

## Solution

FixDesk is a MERN issue-management application with role-aware access, automatic least-loaded assignment, duplicate prevention, an enforced ticket state machine, SLA escalation and operational analytics.

## Key Features

- JWT access tokens, bcrypt password hashing and role-based authorization.
- Duplicate ticket detection for the same student, category and location within 24 hours; similar open reports are shown before submission.
- Category-based SLA deadlines and least-loaded technician assignment, with admin notifications when no technician is available.
- Validated transitions, status history, activity logs, discussion comments and notifications.
- Idempotent SLA escalation, automatic resolution closure after 72 hours without student response and a manual job endpoint.
- Student, staff and administrator views; 14-day ticket trends, resolution metrics, workload and SLA charts.
- Optional Cloudinary image uploads (JPEG, PNG or WebP, maximum 3 MB). Upload failures do not prevent a report from being recorded.

## Tech Stack

| Layer | Tools |
|---|---|
| API | Node.js, Express, Mongoose, MongoDB, JWT, bcryptjs, Zod |
| Security | Helmet, restricted CORS, express-rate-limit, Multer |
| Jobs and media | node-cron, Cloudinary |
| UI | React, Vite, React Router, Axios, Context API, Recharts |
| Tests | Jest, Supertest, mongodb-memory-server |

## Architecture

```text
React/Vite client ──HTTP/JWT──> Express API ──> Controllers ──> Services
                                      │              │              ├─ Assignment/workflow
                                      │              │              ├─ Notifications/logs
                                      │              └──────────────┴─ Mongoose ──> MongoDB
                                      └─ node-cron SLA checks ────────────────┘
```

Ticket state machine:

```text
OPEN ──admin/auto assignment──> ASSIGNED ──staff──> IN_PROGRESS ──staff + note──> RESOLVED
                                  ▲                       ▲                         │
                                  │                       │                         ├─ student confirms ─> CLOSED
                                  │                       └─ staff resumes ─ REOPENED <─ student + reason ─┘
                                  └──────── admin reassigns at any non-closed state ──────────┘
```

An admin can reassign any non-closed ticket. A reopened ticket retains its technician and receives a one-level priority bump.

## Local Setup

Requirements: Node.js 20+, npm and a MongoDB instance (local or Atlas).

1. Copy `server/.env.example` to `server/.env`; set `MONGO_URI` and a long random `JWT_SECRET`.
2. Copy `client/.env.example` to `client/.env`.
3. Install dependencies in `server/` and `client/` with `npm install`.
4. From `server/`, run `npm run seed` to create demo data.
5. Start the API with `npm run dev` in `server/`; start Vite with `npm run dev` in `client/`.
6. Open `http://localhost:5173`; API health is at `http://localhost:5000/api/health`.

## Environment Variables

| Variable | Where | Purpose |
|---|---|---|
| `PORT` | server | API port (default 5000) |
| `MONGO_URI` | server | MongoDB connection string |
| `JWT_SECRET` | server | Signing secret; use a unique production value |
| `JWT_EXPIRES_IN` | server | Access-token lifetime (default `7d`) |
| `CLIENT_URL` | server | Allowed browser origin (default localhost:5173) |
| `CLOUDINARY_CLOUD_NAME` | server | Cloudinary account name; optional |
| `CLOUDINARY_API_KEY` | server | Cloudinary API key; optional |
| `CLOUDINARY_API_SECRET` | server | Cloudinary API secret; optional |
| `SLA_CRON` | server | Cron expression (default `*/10 * * * *`) |
| `NODE_ENV` | server | Runtime mode |
| `VITE_API_URL` | client | API root (default `http://localhost:5000/api`) |

Never commit `.env` files or production credentials.

## Seed/Demo Credentials

All accounts created by `npm run seed` use password `FixDeskDemo!26`.

| Role | Email |
|---|---|
| Admin | `admin@fixdesk.demo` |
| Staff | `wifi@fixdesk.demo`, `plumbing@fixdesk.demo`, `housing@fixdesk.demo`, `labs@fixdesk.demo` |
| Student | `student1@fixdesk.demo` through `student5@fixdesk.demo` |

Seed data is for local demonstration only. Change or remove these credentials before any public deployment.

## API Overview

All endpoints are rooted at `/api`; successful responses use `{ success, data, message }`.

| Area | Routes |
|---|---|
| Auth | `POST /auth/register`, `POST /auth/login`, `GET /auth/me` |
| Tickets | `POST/GET /tickets`, `GET /tickets/:id`, `PATCH /tickets/:id/status`, `PATCH /tickets/:id/assign`, `POST /tickets/:id/comments`, `GET /tickets/similar` |
| Notifications | `GET /notifications`, `PATCH /notifications/:id/read`, `PATCH /notifications/read-all` |
| Admin | `/admin/categories`, `/admin/staff`, `/admin/analytics/*`, `POST /admin/jobs/run-sla-check` |
| Health | `GET /health` |

Pass JWTs as `Authorization: Bearer <token>`. Pagination uses `page` and `limit`; ticket lists accept status, category, priority, search, assignment and overdue filters.

## Testing

Run `cd server && npm test`. Jest/Supertest tests cover workflow transitions, assignment fairness, duplicate checks, tenant isolation and SLA idempotency. Tests use an ephemeral MongoDB when integration setup is enabled; no real campus data is needed.

## Deployment

1. Create a MongoDB Atlas cluster and database user; allow the backend host to connect, then use its connection URI as `MONGO_URI`.
2. Deploy `server/` as a Render web service with build command `npm install`, start command `npm start`, and set the server environment variables. Set `CLIENT_URL` to the deployed frontend origin.
3. Deploy `client/` to Vercel with root directory `client`, build command `npm run build`, output `dist`, and `VITE_API_URL` set to the Render API URL plus `/api`. The included `client/vercel.json` supplies SPA rewrites.
4. Replace demo secrets and verify `/api/health`, CORS, signup and admin workflows.

Render's free tier may sleep. A sleeping service can delay cron runs, so administrators can trigger the same idempotent SLA job with `POST /api/admin/jobs/run-sla-check` after signing in.

**Live links:** Frontend: `[add Vercel URL]` · API: `[add Render URL]`

## Screenshots

_Add screenshots of the student dashboard, technician queue and admin analytics here._

## Future Improvements

- Email and push notifications, campus SSO and audit export.
- Real-time queue updates, configurable escalation policies and service feedback.
- Accessible chart descriptions and multilingual student-facing content.
