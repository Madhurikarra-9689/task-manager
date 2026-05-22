# Team Task Manager

Full-stack web app for creating projects, managing teams, assigning tasks, and tracking progress. Includes **role-based access control** per project: **Admin** and **Member**.

## Features

- **Authentication:** Email/password signup and login, optional **Sign in with Google** (JWT in the browser).
- **Projects:** Create projects; creators become project **Admin**. Edit or delete projects (Admin only).
- **Teams:** Add users by email (they must register first), change roles, remove members (Admin). Members see a read-only roster.
- **Tasks:** Create tasks with optional due date and assignee. **Admins** may assign anyone on the team; **members** may assign only themselves or leave tasks unassigned. Status workflow: `TODO` → `IN_PROGRESS` → `DONE`. Members may update status (and details via API) only on tasks assigned to them; admins have full control including delete.
- **Dashboard:** Summary of tasks assigned to you, counts by status, and a highlighted **overdue** list (due date passed and not `DONE`).

## Stack

| Layer | Technology |
|--------|------------|
| API | Node.js, Express 5, REST |
| Validation | Zod |
| Auth | bcrypt, JWT, optional Google Sign-In |
| Database | PostgreSQL |
| ORM | Prisma 5 |
| UI | React 18, React Router 6, Vite 6 |

## Repository layout

```
server/           Express API + Prisma schema & migrations
client/           Vite + React SPA
```

Production builds place static files in `client/dist`; the server serves them alongside `/api/*`.

## Local development

### Prerequisites

- Node.js 20+
- PostgreSQL (local instance or cloud URL)

### Setup

1. Copy environment files:

   ```bash
   cp server/.env.example server/.env
   ```

2. Set `DATABASE_URL` in `server/.env` to a PostgreSQL connection string and `JWT_SECRET` to a long random string.

3. **Optional — Sign in with Google**

   In [Google Cloud Console](https://console.cloud.google.com/) → **APIs & Services** → **Credentials** → create an **OAuth client ID** (Web application). Under **Authorized JavaScript origins** add `http://127.0.0.1:5173` and your production URL.

   Use the same **Client ID** in:

   - `server/.env` → `GOOGLE_CLIENT_ID`
   - `client/.env.local` → `VITE_GOOGLE_CLIENT_ID`

   Copy `client/.env.example` to `client/.env.local` if helpful.

4. Install dependencies and run migrations (includes Google columns after you pull latest):

   ```bash
   npm install
   cd server && npx prisma migrate deploy && cd ..
   ```

   For iterative schema work, use `npm run db:migrate:dev` from the repo root (runs `prisma migrate dev` in the server workspace).

5. Start API and UI together:

   ```bash
   npm run dev
   ```

   - API: [http://127.0.0.1:4000](http://127.0.0.1:4000)
   - UI: [http://127.0.0.1:5173](http://127.0.0.1:5173) (Vite proxies `/api` to the server)

## Production build

```bash
npm install
npm run build
```

Runs Prisma `generate` and builds the client into `client/dist`.

## Deploying on Railway

The repo includes [`railway.toml`](railway.toml) so Railway uses **`npm install && npm run build`** and **`npm start`** automatically after you connect the repo.

### Steps

1. **Push this repo to GitHub** (if it is not already).

2. In [Railway](https://railway.app): **New project** → **Deploy from GitHub** → select the repo.

3. Add **PostgreSQL**: **New** → **Database** → **PostgreSQL**. In your **web service** → **Variables** → **Add reference** → choose `DATABASE_URL` from the Postgres service (Railway fills `DATABASE_URL` for you).

4. On the **same Node service** that builds from GitHub, set:

   | Variable | Required | Notes |
   |----------|----------|--------|
   | `DATABASE_URL` | Yes | From Postgres reference (or paste connection string). |
   | `JWT_SECRET` | Yes | e.g. run locally: `openssl rand -hex 32` |
   | `NODE_ENV` | Yes | `production` |
   | `PORT` | No | Railway sets this automatically; the server reads `process.env.PORT`. |
   | `GOOGLE_CLIENT_ID` | No | Server-side Google verify (optional). |
   | `VITE_GOOGLE_CLIENT_ID` | No | **Same value as** `GOOGLE_CLIENT_ID` if you want the Google button in the built UI (must be present at **build** time). |

5. **Deploy** — Railway runs build → `npm start`. Start runs `prisma migrate deploy` then serves `/api` and the SPA from `client/dist`.

6. Open the service **public URL**. Routes outside `/api` fall back to `index.html` for React Router.

### If the deploy fails

- **Missing `DATABASE_URL` or `JWT_SECRET`** — check Variables on the **web** service (production requires both).
- **Build fails on Prisma** — ensure `postinstall` runs (`npm install` at repo root); logs should show `prisma generate`.
- **App crashes on boot** — check logs for migration errors; fix local migrations then push again.

## API overview

| Method | Path | Notes |
|--------|------|--------|
| POST | `/api/auth/register` | Body: `email`, `password`, `name` |
| POST | `/api/auth/login` | Body: `email`, `password` (Google-only accounts must use `/google`) |
| POST | `/api/auth/google` | Body: `{ "credential": "<Google JWT>" }` from Google Identity Services |
| GET | `/api/me` | Current user (requires `Authorization: Bearer <jwt>`) |
| GET | `/api/dashboard` | Assigned tasks, overdue, aggregates |
| GET/POST | `/api/projects` | List / create |
| GET/PATCH/DELETE | `/api/projects/:projectId` | Member read; Admin patch/delete |
| GET/POST | `/api/projects/:projectId/members` | Admin POST adds by email |
| PATCH/DELETE | `/api/projects/:projectId/members/:userId` | Admin |
| GET/POST | `/api/projects/:projectId/tasks` | Create tasks |
| PATCH/DELETE | `/api/tasks/:taskId` | RBAC as described above |

## Assignment deliverables checklist

- [ ] Live URL after Railway deploy  
- [ ] GitHub repository link  
- [ ] This README  
- [ ] 2–5 minute demo video (record screens for signup, project creation, inviting a member, tasks, dashboard)

## Scripts (root)

| Script | Purpose |
|--------|---------|
| `npm run dev` | Server + client in watch mode |
| `npm run build` | Prisma generate + Vite production build |
| `npm start` | Migrate + start server (production-style) |
| `npm run db:migrate` | `prisma migrate deploy` |
| `npm run db:migrate:dev` | `prisma migrate dev` |
| `npm run db:studio` | Prisma Studio |

## License

ISC
