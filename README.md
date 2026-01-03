# Enterprise-Ready Multi-Tenant Task & Project Management System

A scalable, production-grade multi-tenant platform built to manage projects and tasks across multiple organizations. The system leverages **Node.js, Express, React, PostgreSQL**, and **Docker** to deliver a secure, modular, and deployment-ready solution.

---

## 🌟 Key Features

* **True Multi-Tenant Isolation** – Each organization’s data is fully segregated
* **Role-Based Access Control (RBAC)** – Three roles supported: `super_admin`, `tenant_admin`, and `user`
* **Secure Authentication** – JWT-based authentication with 24-hour validity and bcrypt password hashing
* **Extensive REST API** – 19 endpoints supporting full CRUD functionality
* **Subscription-Based Limits** – Enforced caps on users and projects per plan
* **Activity Auditing** – Automatic logging of all important system actions
* **Modern React Frontend** – Clean UI with protected routes and centralized auth state
* **Automated Database Setup** – Prisma migrations and seed data run automatically
* **Dockerized Deployment** – Fully containerized services with health checks

---

## 🏗️ Platform Architecture

```
╔═════════════════════════════════════════════════════════╗
║               Frontend (React SPA)                      ║
║              URL: http://localhost:3000                ║
╚═════════════════════════════════════════════════════════╝
                          ↓
╔═════════════════════════════════════════════════════════╗
║              Backend API (Node + Express)               ║
║         Base URL: http://localhost:5000/api             ║
║   • 19 RESTful endpoints                                ║
║   • JWT authentication & authorization                  ║
║   • Zod-based request validation                        ║
║   • Role-based middleware                               ║
╚═════════════════════════════════════════════════════════╝
                          ↓
╔═════════════════════════════════════════════════════════╗
║              Database (PostgreSQL 15)                   ║
║                   Port: 5432                            ║
║   • Tenant-aware schema                                 ║
║   • Prisma ORM                                          ║
╚═════════════════════════════════════════════════════════╝
```

---

## 🚀 Quick Start

### Prerequisites

* Docker & Docker Compose
* Node.js 18+ (for local development)

### Start the Application

```bash
docker-compose up -d
```

### Access Services

* **Frontend UI:** [http://localhost:3000](http://localhost:3000)
* **Backend API:** [http://localhost:5000/api](http://localhost:5000/api)
* **PostgreSQL DB:** localhost:5432

### Check Container Status

```bash
docker-compose ps
```

### Stop Services

```bash
docker-compose down
```

---

## 🧑‍💻 Usage Guide

### Demo Login Credentials

**System Super Admin**

* Email: `superadmin@system.com`
* Password: `Admin@123`

**Tenant Admin (Demo Organization)**

* Email: `admin@demo.com`
* Password: `Demo@123`
* Organization: `demo`

**Standard User**

* Email: `user1@demo.com`
* Password: `User@123`
* Organization: `demo`

Alternate user:

* Email: `user2@demo.com`
* Password: `User@123`

---

### Tenant Registration

Use the **Register** option to create a new organization along with its administrator account.

### User Management

Admins can:

* View all tenant users
* Add new users with assigned roles
* Edit user details
* Remove users

### Project Management

* Create and update projects
* Archive or reactivate projects
* Delete projects

### Task Management

* Create tasks under projects
* Assign priority (low / medium / high)
* Track progress (pending / in progress / completed)
* Edit or remove tasks

---

## 📘 API Reference

Full API documentation is available in **`docs/API.md`**.

### Sample API Requests

**User Login**

```bash
curl -X POST http://localhost:5000/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{
    "email": "admin@demo.com",
    "password": "demo123",
    "tenantSubdomain": "demo"
  }'
```

**Create Project**

```bash
curl -X POST http://localhost:5000/api/tenants/{tenantId}/projects \
  -H "Authorization: Bearer {token}" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Sample Project",
    "description": "Project overview",
    "status": "active"
  }'
```

---

## 🔐 Authentication Details

### JWT Workflow

1. User logs in
2. Server issues a JWT valid for 24 hours
3. Token must be sent in `Authorization` header
4. Expired tokens require re-authentication

### JWT Payload Structure

```json
{
  "userId": "uuid",
  "tenantId": "uuid",
  "email": "user@example.com",
  "role": "admin",
  "iat": 1234567890,
  "exp": 1234654290
}
```

---

## 🗄️ Data Models

### Tenant

* id (UUID)
* name
* subdomain (unique)
* status (active / inactive)
* subscription (free / pro / enterprise)
* maxUsers
* maxProjects

### User

* id
* email
* password (bcrypt hashed)
* fullName
* role (admin / user)
* tenantId
* createdAt

### Project

* id
* name
* description
* status (active / archived)
* tenantId
* createdAt

### Task

* id
* title
* description
* priority (low / medium / high)
* status (pending / in_progress / completed)
* projectId
* createdAt

### Audit Log

* id
* userId
* tenantId
* action
* entityType
* entityId
* changes (JSON)
* createdAt

---

## 🧪 Testing

### Run Integration Tests

```bash
node integration-test.js
```

Tests validate all API endpoints using real-world scenarios.

---

## 📂 Repository Structure

```
frontend/     # React UI
backend/      # Express + TypeScript API
docs/         # API documentation
docker-compose.yml
integration-test.js
README.md
```

---

## 🔒 Security Measures

* Encrypted passwords using bcrypt
* JWT authentication with expiration
* Input validation using Zod
* Role-based authorization middleware
* Tenant-level data isolation
* Audit logs for all write operations
* Configured CORS
* Non-root Docker containers

---

## 📊 Subscription Plans

| Plan       | Users     | Projects  | Access          |
| ---------- | --------- | --------- | --------------- |
| Free       | 5         | 2         | Core features   |
| Pro        | 50        | 10        | Full access     |
| Enterprise | Unlimited | Unlimited | No restrictions |

API enforces limits and returns `400 Bad Request` when exceeded.

---

## 🐳 Docker Utilities

```bash
docker-compose up -d --build
docker-compose down
docker-compose down -v
docker-compose exec backend npm test
```

---

## 🧰 Technology Stack

| Layer            | Technology        |
| ---------------- | ----------------- |
| Frontend         | React + Vite      |
| Backend          | Node.js + Express |
| Language         | TypeScript        |
| Database         | PostgreSQL        |
| ORM              | Prisma            |
| Auth             | JWT               |
| Validation       | Zod               |
| Testing          | Jest              |
| Containerization | Docker            |

---

## 📝 Additional Notes

* All timestamps use UTC
* Emails must be unique per tenant
* Super admin creation is restricted
* Demo data is auto-seeded
* Tokens are stored in browser localStorage

---

## 📜 License

Provided for educational and demonstration use.

---

**Crafted with ❤️ using Node.js, Express, React, PostgreSQL, and Docker**

---
