# HelpDesk Lite — Support Ticket Tracker

HelpDesk Lite is an internal support-ticket app with a React + TypeScript frontend, an Express + TypeScript API, MongoDB persistence, and JWT authentication.

## Requirements

- Node.js 20 or later and npm
- MongoDB, either a local MongoDB server or an Atlas connection string

## Configure environment variables

Create `Backend/.env` from the example:

```sh
cd Backend
cp .env.example .env
```

Edit `Backend/.env` before starting the API:

| Variable | Required | Purpose |
| --- | --- | --- |
| `MONGODB_URI` | Yes | MongoDB connection string, including the database name |
| `JWT_SECRET` | Yes | Random secret of at least 32 characters used to sign JWTs |
| `ADMIN_NAME` | Yes | HelpDesk Admin |
| `ADMIN_EMAIL` | Yes | admin@example.com |
| `ADMIN_PASSWORD` | Yes | Admin@123456 |
| `PORT` | No | API port; defaults to `5000` |
| `FRONTEND_ORIGIN` | No | Allowed browser origin; defaults to `http://localhost:5173` |

Use unique, private passwords for all seeded accounts. Do not commit `.env` files or share secrets. The repository ignores `.env` files.

For Atlas, use the connection string from your Atlas cluster and ensure your network and database user are configured to allow the connection.

## Install dependencies

From the project root, install each app's dependencies:

```sh
cd Backend
npm install
cd ../Frontend
npm install
```

## Run locally

Open two terminals from the project root.

**Terminal 1 — backend API**

```sh
cd Backend
npm run dev
```

**Terminal 2 — frontend**

```sh
cd Frontend
npm run dev
```

Open [http://localhost:5173](http://localhost:5173). The API listens on port `5000` by default, and its health endpoint is [http://localhost:5000/api/health](http://localhost:5000/api/health).

Optional: set `VITE_API_URL` in `Frontend/.env` to override the API base URL. For example:

```dotenv
VITE_API_URL=http://localhost:5000/api
```

Restart Vite after changing frontend environment variables. Set `FRONTEND_ORIGIN` to the exact origin from which the browser app is served.

## Implemented

- Employee signup and login with bcrypt-hashed passwords and JWT-protected API requests.
- Session profile endpoint that returns public user fields only; expired or invalid JWTs send the frontend back to login.
- Role-based access for Employees, Agents, and Admins. Employees only see their own tickets; Agents and Admins can view all tickets. Admins can manage user roles.
- Ticket creation with a required 5–12 character title, required description, `low` / `medium` / `high` priority, and `open` / `in-progress` / `resolved` / `closed` status.
- Ticket detail views with creator, assignee, populated comments, and role-appropriate controls.
- Comments from any user who can access a ticket.
- Agent/Admin self-assignment and ticket status changes.
- Owner-only deletion while a ticket is open.
- Title search, status and priority filters, paginated ticket results, and total counts.
- Frontend loading, empty, validation, and error states.
- Central API error handling and secrets supplied through environment variables.

## Not included yet

- Password reset, email verification, MFA, or refresh-token/session revocation. JWTs expire after eight hours; signing out removes the browser's token.
- File attachments, email notifications, or other external notification integrations.
- A production deployment pipeline, hosting configuration, or production secrets management. Configure HTTPS, CORS, database access, and a frontend static host before deployment.
- An automated test suite wired into the package scripts.

## Production builds

Build both applications:

```sh
cd Backend
npm run build
cd ../Frontend
npm run build
```

Start the compiled API with `npm start` in `Backend` and serve `Frontend/dist` with a static web server configured to fall back to `index.html` for client-side routes such as `/login` and `/tickets/:ticketId`.

## API overview

For protected endpoints, send the JWT in the `Authorization` header using bearer-token authentication.

| Method | Endpoint | Access |
| --- | --- | --- |
| `GET` | `/api/health` | Public |
| `POST` | `/api/auth/register` | Public; creates an Employee |
| `POST` | `/api/auth/login` | Public |
| `GET` | `/api/auth/me` | Signed-in users |
| `GET` | `/api/tickets/stats` | Signed-in users |
| `GET` | `/api/tickets` | Signed-in users; supports `q`, `status`, `priority`, `page`, and `limit`; returns `tickets`, `total`, `page`, `limit`, and `totalPages` |
| `POST` | `/api/tickets` | Signed-in users |
| `GET` | `/api/tickets/:id` | Users who can view the ticket |
| `POST` | `/api/tickets/:id/comments` | Users who can view the ticket; body: `{ "text": "..." }` |
| `PATCH` | `/api/tickets/:id` | Agents and Admins; update status or priority |
| `PATCH` | `/api/tickets/:id/assignment` | Agents and Admins; assign to self |
| `DELETE` | `/api/tickets/:id` | Ticket owner, while status is `open` |
| `GET` | `/api/users` | Admins |
| `PATCH` | `/api/users/:id/role` | Admins |
