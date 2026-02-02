# Copilot Instructions – Smart Docs

## Big Picture Architecture

Smart Docs is a microservices-based system for managing clients, cases, documents, and workflows for advisors (e.g. mortgage consultants).

**Services:**

- **auth-service** (port 8000) – User authentication, JWT issuance, roles (admin / client)
- **client-service** (port 8001) – Clients, Cases, Templates (business domain)
- **document-service** (port 8002) – File uploads, metadata, linked to Cases
- **frontend** (React/Vite + TS) – Separate admin and client UIs
- **nginx** – Reverse proxy routing `/api/*` to services

All services run via `docker-compose` and share MongoDB.

---

## Domain Model – CRITICAL (Model C)

**This project strictly separates User from Client. Do NOT conflate them.**

- **User** = login identity (email, password, name, role)
  - Belongs in auth-service only
  - Created when someone registers
  - JWT `req.user.id` is always a User ID

- **Client** = business entity (firstName, lastName, phone, company, status, advisorId, userId?)
  - Belongs in client-service
  - Created by advisor (admin)
  - Client.userId is **optional** (sparse index) — initially null
  - Client links to User later via POST `/:id/link-myself`

**Rules:**

- A User is NOT automatically a Client
- A Client can exist without a User (advisor creates, client registers later)
- `advisorId` comes **ONLY** from `req.user.id` (JWT) — NEVER from request body
- Frontend must **never** send `advisorId` or `userId`

---

## Authorization & Role Checks

**Admin (Advisor):**

- `POST /clients` – create clients (requires `requireAdmin`)
- `GET /clients/my-clients` – list only their clients
- Full access to Cases and Documents

**Client:**

- `POST /clients/:id/link-myself` – link their User to a pre-created Client
- `GET /clients/me` – get their own Client record
- `GET /cases/my` – get only their Cases
- Download/view only their Documents

---

## Backend Conventions

- Express + TypeScript
- Controllers are thin; business logic in Managers
- Zod for request validation
- Errors thrown (central error handler catches them)
- Models: Client (firstName, lastName, phone, company, status, advisorId, userId?)

---

## Frontend Conventions

- React + TypeScript + Vite
- Redux Toolkit for auth state only
- React Query for server state (cases, clients, documents)
- Admin UI = `/admin/*` routes
- Client UI = `/client/*` routes
- Login/Register = `/login`, `/register` (public)

---

## Developer Workflow

```bash
# Start everything
docker-compose up --build

# Check service logs
docker-compose logs <service-name>

# Stop
docker-compose down
```

**Environment:** JWT secret from `.env`

---

## Common Mistakes to Avoid

❌ Sending `advisorId` from frontend  
❌ Trusting `userId` from request body  
❌ Putting business fields in User (auth-service)  
❌ Requiring Client.userId to be set on creation  
❌ Allowing clients to list other clients

✅ Always extract `advisorId` from `req.user.id`  
✅ Validate role and ownership server-side  
✅ Keep auth-service identity-only  
✅ Use sparse index for optional Client.userId
