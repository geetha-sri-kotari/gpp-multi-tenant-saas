# Enterprise Task Management Platform – Quick Guide

## 🚀 Startup Instructions

```bash
cd "d:\GPP\Multi-Tenant SaaS Platform with Project & Task Management"
docker-compose up -d
```

Please wait **10–15 seconds** for all services to fully initialize.

---

## 🌐 Application Endpoints

| Service      | URL                                                                  | Status    |
| ------------ | -------------------------------------------------------------------- | --------- |
| Frontend UI  | [http://localhost:3000](http://localhost:3000)                       | ✅ Running |
| Backend API  | [http://localhost:5000/api](http://localhost:5000/api)               | ✅ Running |
| Health Check | [http://localhost:5000/api/health](http://localhost:5000/api/health) | ✅ Running |
| PostgreSQL   | localhost:5432                                                       | ✅ Running |

---

## 🔐 Demo Login Credentials

### System Super Administrator

(Access across all organizations)

```
Email: super_admin@demo.com
Password: super_admin
```

### Tenant Administrator

(Manages Demo organization)

```
Email: admin@demo.com
Password: demo123
Tenant: demo
```

### Regular User

(Standard access within Demo tenant)

```
Email: user@demo.com
Password: demo123
Tenant: demo
```

---

## 📱 User Interface Sections

1. **Login Page** ([http://localhost:3000](http://localhost:3000))

   * Secure authentication screen
   * Demo credentials visible
   * Link to tenant registration

2. **Organization Registration** ([http://localhost:3000/register](http://localhost:3000/register))

   * Create a new tenant
   * Define admin account
   * Set up company details

3. **Dashboard** ([http://localhost:3000/dashboard](http://localhost:3000/dashboard))

   * High-level organization overview
   * Quick navigation
   * Logged-in user info

4. **User Management** ([http://localhost:3000/users](http://localhost:3000/users))

   * View tenant users
   * Add new members
   * Update roles and permissions
   * Remove users

5. **Projects Section** ([http://localhost:3000/projects](http://localhost:3000/projects))

   * Create new projects
   * Edit project metadata
   * Activate or archive projects
   * Delete projects

6. **Tasks Workspace** ([http://localhost:3000/tasks](http://localhost:3000/tasks))

   * Create tasks under projects
   * Set task priority
   * Update task status
   * Remove tasks

---

## 📚 API Examples

### User Login

```bash
curl -X POST http://localhost:5000/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{
    "email": "admin@demo.com",
    "password": "demo123",
    "tenantSubdomain": "demo"
  }'
```

The response includes a JWT token.
Use it for all protected requests:

```
Authorization: Bearer <token>
```

---

### Fetch Logged-In User

```bash
curl -X GET http://localhost:5000/api/auth/me \
  -H "Authorization: Bearer <token>"
```

---

### Retrieve Users

```bash
curl -X GET http://localhost:5000/api/tenants/{tenantId}/users \
  -H "Authorization: Bearer <token>"
```

---

### Create a Project

```bash
curl -X POST http://localhost:5000/api/tenants/{tenantId}/projects \
  -H "Authorization: Bearer <token>" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "My Project",
    "description": "Project description",
    "status": "active"
  }'
```

---

## 🛠️ Common Commands

### Verify Container Status

```bash
docker-compose ps
```

### View Logs

```bash
# Backend
docker logs backend -f

# Frontend
docker logs frontend -f

# Database
docker logs database -f
```

### Execute Integration Tests

```bash
node integration-test.js
```

### Stop the Platform

```bash
docker-compose down
```

### Reset Everything (Fresh Start)

```bash
docker-compose down -v
docker-compose up -d
```

---

## 📖 Reference Material

* **API Docs:** [docs/API.md](docs/API.md) – All 19 endpoints
* **Setup Manual:** [README.md](README.md) – Complete setup guide
* **Completion Overview:** [COMPLETION_SUMMARY.md](COMPLETION_SUMMARY.md)
* **Deliverables List:** [DELIVERABLES.md](DELIVERABLES.md)
* **Submission Info:** [submission.json](submission.json)

---

## 🔑 Available API Endpoints (19 Total)

### Authentication (4)

* `POST /api/auth/register-tenant` – Create tenant
* `POST /api/auth/login` – Login
* `GET /api/auth/me` – Current user
* `POST /api/auth/logout` – Logout

### Tenant Management (3)

* `GET /api/tenants` – List tenants (super admin)
* `GET /api/tenants/:tenantId` – Tenant details
* `PUT /api/tenants/:tenantId` – Update tenant

### User Management (4)

* `POST /api/tenants/:tenantId/users` – Add user
* `GET /api/tenants/:tenantId/users` – Get users
* `PUT /api/users/:userId` – Update user
* `DELETE /api/users/:userId` – Remove user

### Project Management (4)

* `POST /api/tenants/:tenantId/projects` – Create project
* `GET /api/tenants/:tenantId/projects` – List projects
* `PUT /api/projects/:projectId` – Update project
* `DELETE /api/projects/:projectId` – Delete project

### Task Management (4)

* `POST /api/projects/:projectId/tasks` – Add task
* `GET /api/projects/:projectId/tasks` – View tasks
* `PUT /api/tasks/:taskId` – Update task
* `DELETE /api/tasks/:taskId` – Delete task

---

## ✨ Highlights

* ✅ 19 fully operational APIs
* ✅ Secure multi-tenant design
* ✅ JWT authentication (24-hour validity)
* ✅ Role-based permissions
* ✅ Zod request validation
* ✅ Full audit logging
* ✅ React frontend (6 main pages)
* ✅ Dockerized deployment
* ✅ Auto database seeding
* ✅ Complete documentation

---

## 🐛 Common Issues & Fixes

### Port Conflict

```bash
# Example for port 3000
lsof -ti:3000 | xargs kill -9  # macOS/Linux

# Windows
netstat -ano | findstr :3000
```

### Containers Not Starting

```bash
docker-compose down
docker-compose up -d --build
```

### Database Connectivity Issues

```bash
docker-compose logs database | tail -20
docker-compose down
docker-compose up -d
```

### Expired Token

* JWT tokens expire after 24 hours
* Re-login to obtain a new token

---

## 📊 High-Level Architecture

```
┌─────────────────┐
│ React UI (3000) │
└────────┬────────┘
         ↓
┌─────────────────┐
│ Express API     │
│ (5000)          │
└────────┬────────┘
         ↓
┌─────────────────┐
│ PostgreSQL DB   │
│ (5432)          │
└─────────────────┘
```

---

## 📋 Directory Layout

```
.
├── frontend/           # React frontend
├── backend/            # Express backend
├── docs/               # Documentation
├── docker-compose.yml  # Containers
├── README.md           # Main guide
├── submission.json     # Metadata
└── integration-test.js # Test suite
```

---

## 🎯 Getting Started Checklist

1. Run `docker-compose up -d`
2. Open [http://localhost:3000](http://localhost:3000)
3. Log in using demo credentials
4. Navigate through the UI
5. Review API docs for deeper understanding

---

## 📝 Important Notes

* PostgreSQL stores all application data
* JWT tokens are saved in browser localStorage
* Passwords are encrypted using bcryptjs
* Zod validates all API inputs
* Audit logs capture all data changes
* Super admins have global access
* Tenant admins are restricted to their organization
* Standard users have limited privileges

---

**The platform is fully operational 🚀**
Start now with:

```bash
docker-compose up -d
```
