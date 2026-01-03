# Platform Technical Design

## Technology Stack Overview

* **Server Layer:** Node.js 18.x runtime with Express.js framework, written in TypeScript. Data access handled via Prisma ORM, request validation using Zod, authentication through JWT, and password security with bcryptjs.
* **Frontend Layer:** React 18.x application built using Vite, client-side routing with React Router, and REST communication implemented via the Fetch API.
* **Persistence Layer:** PostgreSQL version 15.x as the primary relational database.
* **Containerization & Orchestration:** Docker for containerization and Docker Compose for multi-service coordination.

---

## Project Repository Layout (Key Directories)

* **backend/**

  * `src/index.ts` – Entry point for initializing the Express server
  * `src/controllers/*.ts` – Business logic for auth, tenants, users, projects, and tasks
  * `src/middleware/auth.ts` – JWT authentication and role-based authorization checks
  * `src/routes/*.ts` – API route definitions
  * `src/utils/{jwt,audit}.ts` – Helper utilities for token handling and audit logging
  * `src/prisma.ts` – Prisma client initialization
  * `prisma/schema.prisma`, `prisma/seed.js` – Database schema and seed data
* **frontend/**

  * `src/main.jsx`, `src/App.jsx` – Application bootstrap and route configuration
  * `src/context/AuthContext.jsx` – Global authentication state management
  * `src/components/ProtectedRoute.jsx` – Route access control logic
  * `src/services/api.js` – Centralized API communication layer
  * `src/pages/*.jsx` – UI pages (Login, Registration, Dashboard, Users, Projects, Tasks)
* **docs/** – Documentation including API specs, PRD, research notes, architecture, and requirements
* `docker-compose.yml` – Defines and orchestrates database, backend, and frontend services
* `integration-test.js` – End-to-end API verification script

---

## Environment Configuration

### Backend Environment Variables (`.env`)

* `DATABASE_URL=postgresql://postgres:postgres@database:5432/saas_db`
* `JWT_SECRET=<secure-random-string-min-32-characters>`
* `JWT_EXPIRES_IN=24h`
* `PORT=5000`
* `FRONTEND_URL=http://frontend:3000`

### Frontend Environment Variables

* `VITE_API_URL=http://backend:5000/api`

---

## Running Locally Without Containers

1. Start a PostgreSQL instance on port 5432 and update `DATABASE_URL` accordingly.
2. Start the backend:

   ```bash
   cd backend
   npm install
   npx prisma migrate deploy
   npx prisma db seed
   npm run dev
   ```
3. Start the frontend:

   ```bash
   cd frontend
   npm install
   npm run dev
   ```

   Ensure `VITE_API_URL` points to the backend.

---

## Docker-Based Deployment

1. Start all services:

   ```bash
   docker-compose up -d
   ```
2. Confirm service availability:

   * Backend health check: `/api/health`
   * Frontend accessible on port `3000`
3. Stop services:

   ```bash
   docker-compose down
   ```

   Use `-v` to remove persistent volumes if required.

---

## Testing & Quality Assurance

* **Integration Tests:** Run `node integration-test.js` from the project root while services are active.
* **Backend Unit Tests:** Execute:

  ```bash
  cd backend && npm test
  ```

---

## API Design Notes

* **Auth Endpoints:** `/api/auth/register-tenant`, `/login`, `/me`, `/logout`
* Role-based access enforced through middleware
* All tenant-specific data endpoints apply tenant-level filtering
* Platform administrators operate without tenant binding (`tenantId = null`)
* Consistent JSON response format:

  ```json
  { "success": true, "message": "...", "data": {} }
  ```

  accompanied by appropriate HTTP status codes

---

## Production Readiness Guidelines

* Secure injection of environment variables
* Use a strong and confidential JWT secret
* Enable HTTPS/TLS for all external traffic
* Configure strict CORS allowlists
* Apply database migrations during deployment using:

  ```bash
  npx prisma migrate deploy
  ```
* Build frontend assets via:

  ```bash
  npm run build
  ```

  and serve static files through the configured Docker setup

---