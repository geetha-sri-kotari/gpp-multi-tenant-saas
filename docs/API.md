# Enterprise Multi-Tenant Platform – REST API Guide

## Overview

This API powers an enterprise-level, multi-tenant application designed for managing projects and tasks at scale. It enforces **strong tenant isolation**, **role-based access control (RBAC)**, and **subscription-driven resource limits** to ensure security, scalability, and flexibility.

**Base API URL:**

```
http://localhost:5000/api
```

## Contents

1. Authentication Services
2. Tenant Administration
3. User Management
4. Project Services
5. Task Services
6. Error Handling Standards
7. Sample Login Credentials

---

## Authentication Services

### 1. Tenant Registration

Create a new organization (tenant) along with its primary administrator account.

**Endpoint:**

```
POST /api/auth/register-tenant
```

**Request Body:**

```json
{
  "tenantName": "Acme Corp",
  "subdomain": "acme",
  "adminEmail": "admin@acme.com",
  "adminPassword": "SecurePassword123!",
  "adminFullName": "John Admin"
}
```

**Success Response (201):**

```json
{
  "success": true,
  "message": "Tenant created successfully",
  "data": {
    "tenantId": "uuid-here",
    "adminEmail": "admin@acme.com",
    "tenantName": "Acme Corp"
  }
}
```

**Possible Errors:**

* `400` – Validation failure or duplicate subdomain
* `500` – Server-side error

---

### 2. User Login

Verify user credentials and issue an access token.

**Endpoint:**

```
POST /api/auth/login
```

**Request Body:**

```json
{
  "email": "admin@demo.com",
  "password": "demo123",
  "tenantSubdomain": "demo"
}
```

*(Tenant subdomain is optional for super_admin accounts.)*

**Success Response (200):**

```json
{
  "success": true,
  "message": "Authentication successful",
  "data": {
    "token": "jwt-token-here",
    "user": {
      "id": "uuid-here",
      "email": "admin@demo.com",
      "fullName": "Demo Admin",
      "role": "admin",
      "tenantId": "uuid-here"
    }
  }
}
```

**Token Details:**

* Format: JWT (HS256)
* Validity: 24 hours
* Header usage:

```
Authorization: Bearer <token>
```

**Error Scenarios:**

* `401` – Invalid credentials
* `404` – User does not exist

---

### 3. Fetch Logged-in User

Retrieve details of the currently authenticated user.

**Endpoint:**

```
GET /api/auth/me
```

**Response (200):**

```json
{
  "success": true,
  "message": "User details fetched",
  "data": {
    "id": "uuid-here",
    "email": "admin@demo.com",
    "fullName": "Demo Admin",
    "role": "admin",
    "tenantId": "uuid-here"
  }
}
```

**Errors:**

* `401` – Token missing or invalid
* `404` – User not found

---

### 4. Logout

End the user session (handled client-side for JWT).

**Endpoint:**

```
POST /api/auth/logout
```

**Response (200):**

```json
{
  "success": true,
  "message": "Successfully logged out"
}
```

---

## Tenant Administration

### 1. View All Tenants (Super Admin)

Retrieve a paginated list of all tenants in the system.

**Endpoint:**

```
GET /api/tenants
```

**Query Options:**

* `page` (default: 1)
* `limit` (default: 10)

**Response (200):**

```json
{
  "success": true,
  "data": [
    {
      "id": "uuid-here",
      "name": "Demo Tenant",
      "subdomain": "demo",
      "status": "active",
      "createdAt": "2024-01-01T00:00:00Z"
    }
  ]
}
```

**Errors:**

* `401` – Not authenticated
* `403` – Requires super_admin role

---

### 2. Tenant Details

Fetch information about a specific tenant.

**Endpoint:**

```
GET /api/tenants/:tenantId
```

**Response (200):**

```json
{
  "success": true,
  "data": {
    "id": "uuid-here",
    "name": "Demo Tenant",
    "subdomain": "demo",
    "status": "active",
    "subscription": {
      "plan": "pro",
      "maxUsers": 50,
      "maxProjects": 10
    },
    "createdAt": "2024-01-01T00:00:00Z"
  }
}
```

**Errors:**

* `401` – Unauthorized
* `403` – Cross-tenant access denied
* `404` – Tenant not found

---

### 3. Edit Tenant (Admin)

Update tenant name or subscription plan.

**Endpoint:**

```
PUT /api/tenants/:tenantId
```

**Request Body:**

```json
{
  "name": "Updated Tenant Name",
  "subscription": "enterprise"
}
```

**Response (200):**

```json
{
  "success": true,
  "message": "Tenant updated",
  "data": {
    "id": "uuid-here",
    "name": "Updated Tenant Name",
    "subscription": "enterprise"
  }
}
```

---

## User Management

### 1. Create Tenant User

Add a new user under a specific tenant.

**Endpoint:**

```
POST /api/tenants/:tenantId/users
```

**Rules:**

* Email must be unique within the tenant
* Password minimum: 8 characters
* Allowed roles: `user`, `admin`

**Response (201):**

```json
{
  "success": true,
  "message": "User created",
  "data": {
    "id": "uuid-here",
    "email": "newuser@example.com",
    "fullName": "New User",
    "role": "user"
  }
}
```

---

### 2. View Tenant Users

List all users belonging to a tenant.

**Endpoint:**

```
GET /api/tenants/:tenantId/users
```

**Response (200):**

```json
{
  "success": true,
  "data": [
    {
      "id": "uuid-here",
      "email": "admin@demo.com",
      "fullName": "Demo Admin",
      "role": "admin",
      "createdAt": "2024-01-01T00:00:00Z"
    }
  ]
}
```

---

### 3. Modify User

Update user details or role.

**Endpoint:**

```
PUT /api/users/:userId
```

---

### 4. Remove User

Delete a user from the tenant.

**Endpoint:**

```
DELETE /api/users/:userId
```

---

## Project Services

### 1. Create Project

Add a new project under a tenant.

**Endpoint:**

```
POST /api/tenants/:tenantId/projects
```

---

### 2. List Projects

Retrieve all tenant projects with filters.

**Filters:**

* `status`: active | archived

---

### 3. Update Project

Edit project name, description, or status.

---

### 4. Delete Project

Remove a project and all its tasks.

---

## Task Services

### 1. Add Task

Create a task within a project.

**Priority:** low | medium | high
**Status:** pending | in_progress | completed

---

### 2. View Tasks

Retrieve tasks with optional filters.

---

### 3. Edit Task

Update task metadata.

---

### 4. Delete Task

Permanently remove a task.

---

## Error Handling

### Unified Error Format

```json
{
  "success": false,
  "message": "Error message",
  "code": "ERROR_CODE"
}
```

### Common Codes

* `VALIDATION_ERROR`
* `UNAUTHORIZED`
* `FORBIDDEN`
* `NOT_FOUND`
* `LIMIT_EXCEEDED`
* `ALREADY_EXISTS`

---

## Demo Accounts

### Super Administrator

```
Email: super_admin@demo.com
Password: super_admin
```

### Tenant Admin

```
Email: admin@demo.com
Password: demo123
Tenant: demo
```

### Tenant User

```
Email: user@demo.com
Password: demo123
Tenant: demo
```

---

## Security & Compliance

* All write operations are audit logged
* Logs include user, tenant, action, timestamp, and IP
* JWT tokens expire after 24 hours

---

## Subscription Plans

| Plan       | Users     | Projects  | Capabilities        |
| ---------- | --------- | --------- | ------------------- |
| Free       | 5         | 2         | Core task features  |
| Pro        | 50        | 10        | Advanced management |
| Enterprise | Unlimited | Unlimited | Full access         |

---
