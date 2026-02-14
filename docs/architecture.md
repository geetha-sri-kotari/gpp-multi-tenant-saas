# Architecture Overview

## Multi-Tenant SaaS Application

---

# 1. Overall System Design

## Architectural Pattern

The platform is built using a classic three-layer architecture consisting of presentation, application, and data layers.

```
┌─────────────────┐
│   Web Browser   │
│   (Client UI)   │
└────────┬────────┘
         │ HTTP/HTTPS
         │
┌────────▼────────┐
│  React Frontend │
│  (Port 3000)    │
│  - UI Components│
│  - Views        │
│  - API Services │
└────────┬────────┘
         │ REST Calls
         │ JWT Tokens
         │
┌────────▼────────┐
│  Node/Express   │
│  API Server     │
│  (Port 5000)    │
│  - Route Handlers│
│  - Middleware    │
│  - Controllers   │
└────────┬────────┘
         │ SQL Execution
         │
┌────────▼────────┐
│  PostgreSQL DB  │
│  (Port 5432)    │
│  - Tenant Data  │
│  - Users        │
│  - Projects     │
│  - Tasks        │
└─────────────────┘
```

The frontend communicates with the backend through REST APIs, and the backend interacts with PostgreSQL for persistent storage.

---

## Authentication Workflow

```
1. User submits login credentials
   ↓
2. Frontend sends POST /api/auth/login request
   ↓
3. Backend verifies email and password
   ↓
4. JWT token is generated
   ↓
5. Token is saved in browser localStorage
   ↓
6. Token is attached to Authorization header in future requests
   ↓
7. Backend middleware validates token for each request
   ↓
8. User identity (userId, tenantId, role) extracted
   ↓
9. Request processed with tenant-level restrictions
```

JWT is used to maintain stateless authentication across API calls.

---

## Project Creation Data Flow

```
1. User selects "Create Project"
   ↓
2. Frontend sends POST /api/projects including JWT
   ↓
3. Authentication middleware validates token
   ↓
4. tenantId is retrieved from token
   ↓
5. Subscription limits are verified
   ↓
6. New project is saved with tenantId association
   ↓
7. Entry recorded in audit_logs
   ↓
8. Success response returned
   ↓
9. Frontend refreshes project list
```

This ensures that every created resource is linked to the correct tenant.

---

# 2. Database Structure

## Entity Relationship Model

```
┌─────────────┐
│   tenants   │
├─────────────┤
│ id (PK)     │
│ name        │
│ subdomain   │◄─────┐
│ status      │      │
│ plan        │      │
│ max_users   │      │
│ max_projects│      │
└─────────────┘      │
                     │
┌─────────────┐      │
│    users    │      │
├─────────────┤      │
│ id (PK)     │      │
│ tenant_id   │──────┘
│ email       │
│ password    │
│ full_name   │
│ role        │
│ is_active   │
└──────┬──────┘
       │
┌──────▼──────┐
│  projects   │
├─────────────┤
│ id (PK)     │
│ tenant_id   │
│ name        │
│ description │
│ status      │
│ created_by  │
└──────┬──────┘
       │
┌──────▼──────┐
│    tasks    │
├─────────────┤
│ id (PK)     │
│ project_id  │
│ tenant_id   │
│ title       │
│ description │
│ status      │
│ priority    │
│ assigned_to │
│ due_date    │
└─────────────┘

┌─────────────┐
│ audit_logs  │
├─────────────┤
│ id (PK)     │
│ tenant_id   │
│ user_id     │
│ action      │
│ entity_type │
│ entity_id   │
│ ip_address  │
│ created_at  │
└─────────────┘
```

---

## Important Design Principles

1. **Tenant-Level Segregation**
   Every core table includes a `tenant_id` field to enforce isolation.

2. **Referential Integrity**
   Foreign keys use cascade deletion to maintain consistency.

3. **Indexing Strategy**
   All `tenant_id` columns are indexed to optimize filtered queries.

4. **Scoped Uniqueness**
   Emails are unique within each tenant using `UNIQUE(tenant_id, email)`.

5. **Global Super Admins**
   Super admins have `tenant_id = NULL` and can access all tenants.

---

# 3. API Design Structure

## Resource-Based Routing

Endpoints follow REST standards and are grouped by resource type.

### Authentication (`/api/auth`)

* Register a tenant
* User login
* Retrieve current user
* Logout

### Tenant Management (`/api/tenants`)

* View all tenants (super admin)
* View tenant details
* Modify tenant configuration

### User Management

* Add users to tenant
* List tenant users
* Update user information
* Remove users

### Project Management (`/api/projects`)

* Create project
* Retrieve projects
* Edit project
* Delete project

### Task Management

* Create task under project
* Retrieve tasks
* Update task status
* Modify task details

---

## Access Control Matrix

Each endpoint requires authentication except registration and login. Authorization is enforced based on role:

* Super admin: full system access
* Tenant admin: manage own tenant
* Standard user: limited to own tenant data

Role-based restrictions are enforced at middleware level.

---

## Response Structure Standardization

All responses use a consistent JSON format.

Successful response:

```json
{
  "success": true,
  "message": "Optional information",
  "data": {}
}
```

Error response:

```json
{
  "success": false,
  "message": "Error details"
}
```

---

## HTTP Status Codes Used

* 200 – Request successful
* 201 – Resource created
* 400 – Invalid input
* 401 – Authentication failed
* 403 – Permission denied
* 404 – Resource missing
* 409 – Conflict detected
* 500 – Unexpected server issue

---

# 4. Security Framework

## Multi-Tier Security Model

1. **Transport Security** – HTTPS in production
2. **Authentication** – JWT-based stateless authentication
3. **Authorization** – Role-Based Access Control (RBAC)
4. **Data Filtering** – Tenant-specific query enforcement
5. **Input Protection** – Validation and SQL injection prevention
6. **Audit Logging** – System-wide activity tracking

---

## Tenant Isolation Logic

```
Incoming Request
      ↓
Authentication Middleware (validate JWT)
      ↓
Authorization Middleware (verify role)
      ↓
Controller (append tenantId condition)
      ↓
Database (return tenant-specific data)
```

All database queries are filtered by `tenant_id` to prevent cross-tenant data exposure.

---

## Secure Data Access Flow

1. Request arrives with JWT
2. Token verified and decoded
3. Role permissions checked
4. Controller enforces tenant filter
5. Database returns scoped results
6. Response delivered to client

---

# 5. Deployment Strategy

## Containerized Deployment (Docker Compose)

```
┌─────────────────────────────────────┐
│      Docker Internal Network        │
│                                     │
│  ┌──────────┐    ┌──────────┐       │
│  │ Frontend │──▶ │ Backend  │       │
│  │ :3000    │    │ :5000    │       │
│  └──────────┘    └────┬─────┘       │
│                       │             │
│                  ┌────▼─────┐       │
│                  │ Database │       │
│                  │ :5432    │       │
│                  └──────────┘       │
└─────────────────────────────────────┘
```

### Internal Communication

* Frontend communicates with backend via service name
* Backend connects to database using Docker network alias
* External access provided through mapped localhost ports

---

# 6. Scalability Strategy

## Supported Scaling Options

* **Horizontal Backend Scaling** – Multiple API instances behind load balancer
* **Database Replication** – Read replicas for high-read scenarios
* **Caching Layer** – Redis integration for performance improvement
* **Load Balancing** – Reverse proxy for traffic distribution

---

## Performance Enhancements

1. Indexed `tenant_id` fields
2. Connection pooling in backend
3. Pagination for list APIs
4. Optimized SQL joins and filters

---

# 7. Planned Improvements

Future architectural upgrades may include:

1. Breaking into microservices (auth, tenant, project modules)
2. Introducing message brokers like Kafka or RabbitMQ
3. Adding Redis caching
4. CDN for static asset distribution
5. API gateway for centralized routing and rate limiting
6. Monitoring using Prometheus and Grafana
7. Centralized logging through ELK stack