# Multi-Tenant SaaS Application

A fully containerized **Multi-Tenant SaaS Application** built using a modern frontend and backend stack. This project showcases **secure tenant isolation, role-based authorization, automated database setup, and Docker-based deployment**.

It is designed for **automated assessment** and to demonstrate a real-world SaaS architecture with consistent setup using Docker.

---

## Intended Audience

* SaaS system reviewers
* Backend and Full Stack developers
* Organizations evaluating multi-tenant architecture
* Students presenting production-ready SaaS systems

---

## Core Features

* Secure multi-tenant architecture with strict data isolation
* Role-based access control (`super_admin`, `tenant_admin`, `user`)
* JWT-based authentication
* Automatic execution of database migrations on startup
* Automatic seed data insertion on startup
* PostgreSQL with persistent Docker volume storage
* Fully containerized services (frontend, backend, database)
* Health check endpoint for service monitoring
* Centralized API communication layer
* One-command startup using Docker Compose

---

## Technology Stack

### Frontend

* React
* Axios
* React Router
* Docker

### Backend

* Node.js
* Express.js
* PostgreSQL
* JWT Authentication
* bcrypt for password hashing

### Database

* PostgreSQL 15

### DevOps & Tools

* Docker
* Docker Compose
* npm

---

## System Architecture

The application follows a **three-service structure**:

1. **Frontend** – User interface layer
2. **Backend API** – Handles authentication, authorization, and business logic
3. **Database** – PostgreSQL with tenant-level isolation

Services communicate internally using **Docker service names**, not `localhost`.

System Architecture Diagram
Database ERD Diagram

---

## Project Layout

```
multi-tenant-saas-platform/
├── backend/
│   ├── src/
│   ├── migrations/
│   ├── seeds/
│   ├── Dockerfile
│   ├── package.json
│   ├── package-lock.json
│   └── .env.example
│
├── frontend/
│   ├── src/
│   ├── Dockerfile
│   ├── package.json
│   └── package-lock.json
│
├── docs/
│   ├── research.md
│   ├── PRD.md
│   ├── architecture.md
│   ├── technical-spec.md
│   ├── API.md
│   └── images/
│
├── submission.json
├── docker-compose.yml
└── README.md
```

---

## Docker-Based Setup (Required)

### Prerequisites

* Docker
* Docker Compose

---

### Start the Application

```bash
docker-compose up -d
```

This single command will:

* Launch PostgreSQL (`database`)
* Start the Backend API (`backend`)
* Start the Frontend (`frontend`)
* Run database migrations automatically
* Load seed data automatically

No manual setup steps are required.

---

## Port Configuration

| Service  | Host Port | Container Port |
| -------- | --------- | -------------- |
| Database | 5432      | 5432           |
| Backend  | 5000      | 5000           |
| Frontend | 3000      | 3000           |

---

## Access URLs

Frontend:
[http://localhost:3000](http://localhost:3000)

Backend Health Endpoint:
[http://localhost:5000/api/health](http://localhost:5000/api/health)

Expected response:

```json
{
  "status": "ok",
  "database": "connected"
}
```

---

## Environment Variables

All required environment variables are defined directly inside `docker-compose.yml` for automated evaluation.

### Backend Variables

| Variable     | Description                 |
| ------------ | --------------------------- |
| DB_HOST      | Database service name       |
| DB_PORT      | Database port               |
| DB_NAME      | Database name               |
| DB_USER      | Database username           |
| DB_PASSWORD  | Database password           |
| JWT_SECRET   | Secret used for JWT signing |
| FRONTEND_URL | Allowed frontend origin     |

### Frontend Variables

| Variable          | Description             |
| ----------------- | ----------------------- |
| REACT_APP_API_URL | Base URL of backend API |

---

## Automatic Database Setup

* Migrations execute automatically on backend startup
* Seed data loads immediately after migrations
* No manual database scripts required

Seed data includes:

* 1 Super Admin
* 1 Tenant with a Tenant Admin
* At least 1 User per tenant
* At least 1 Project per tenant
* At least 1 Task per project

---

## Test Login Details

All evaluation credentials are provided in:

```
submission.json
```

The automated checker uses the exact credentials defined in this file.

---

## API Documentation

Complete API details are available in:

* `docs/API.md`
* OR Swagger/Postman (if integrated)

All 19 endpoints include:

* HTTP method
* Endpoint path
* Authentication requirements
* Request examples
* Response examples

---

## Documentation Files

| File                     | Purpose                                           |
| ------------------------ | ------------------------------------------------- |
| `docs/research.md`       | Multi-tenant research and security considerations |
| `docs/PRD.md`            | Product Requirements Document                     |
| `docs/architecture.md`   | Architecture explanation and ERD                  |
| `docs/technical-spec.md` | Technical setup details                           |
| `docs/API.md`            | Full API reference                                |

---

## Verification Steps

After running:

```bash
docker-compose up -d
```

Confirm:

* All three services show **Up** using `docker-compose ps`
* Health endpoint returns HTTP 200
* Frontend loads successfully
* Login works using `submission.json` credentials
* Seed data is available in the database

---

## Security Practices

* Password hashing with bcrypt
* JWT-based authentication
* Tenant-level query filtering
* Backend input validation
* Role-based authorization
* Docker service isolation
* Persistent database volumes