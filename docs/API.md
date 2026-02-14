# API Reference Guide

## Multi-Tenant SaaS Application

**Base Endpoint:**
`http://localhost:5000/api` (or deployed backend address)

All endpoints respond in JSON format using a standardized structure:

* Successful Response:
  `{ success: true, message?: string, data?: object }`

* Failed Response:
  `{ success: false, message: string }`

---

## Authentication Requirements

Most routes require a valid JWT included in the request header:

```
Authorization: Bearer <jwt_token>
```

---

# 1. Authentication Routes

## 1.1 Create a New Tenant

**Route:** `POST /api/auth/register-tenant`
**Access Level:** Public

### Request Payload

```json
{
  "tenantName": "Test Company Alpha",
  "subdomain": "testalpha",
  "adminEmail": "admin@testalpha.com",
  "adminPassword": "TestPass@123",
  "adminFullName": "Alpha Admin"
}
```

### Successful Response (201 Created)

```json
{
  "success": true,
  "message": "Tenant registered successfully",
  "data": {
    "tenantId": "uuid",
    "subdomain": "testalpha",
    "adminUser": {
      "id": "uuid",
      "email": "admin@testalpha.com",
      "fullName": "Alpha Admin",
      "role": "tenant_admin"
    }
  }
}
```

### Error Conditions

* 400 – Invalid or missing fields
* 409 – Email or subdomain already in use

---

## 1.2 User Sign-In

**Route:** `POST /api/auth/login`
**Access Level:** Public

### Request Payload

```json
{
  "email": "admin@demo.com",
  "password": "Demo@123",
  "tenantSubdomain": "demo"
}
```

### Successful Response (200 OK)

```json
{
  "success": true,
  "data": {
    "user": {
      "id": "uuid",
      "email": "admin@demo.com",
      "fullName": "Demo Admin",
      "role": "tenant_admin",
      "tenantId": "uuid"
    },
    "token": "jwt-token-string",
    "expiresIn": 86400
  }
}
```

### Error Conditions

* 401 – Incorrect login credentials
* 404 – Tenant not found
* 403 – Account inactive or suspended

---

## 1.3 Retrieve Authenticated User Details

**Route:** `GET /api/auth/me`
**Access Level:** Authenticated users only

### Successful Response (200 OK)

```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "email": "admin@demo.com",
    "fullName": "Demo Admin",
    "role": "tenant_admin",
    "isActive": true,
    "tenant": {
      "id": "uuid",
      "name": "Demo Company",
      "subdomain": "demo",
      "subscriptionPlan": "pro",
      "maxUsers": 25,
      "maxProjects": 15
    }
  }
}
```

### Error Conditions

* 401 – Token missing, expired, or invalid
* 404 – User record not found

---

## 1.4 Logout

**Route:** `POST /api/auth/logout`
**Access Level:** Authenticated

### Successful Response (200 OK)

```json
{
  "success": true,
  "message": "Logged out successfully"
}
```

---

# 2. Tenant Management

## 2.1 Retrieve Tenant Information

**Route:** `GET /api/tenants/:tenantId`
**Access Level:** Authenticated
**Permission:** Must belong to the tenant or have super_admin role

### Successful Response (200 OK)

```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "name": "Demo Company",
    "subdomain": "demo",
    "status": "active",
    "subscriptionPlan": "pro",
    "maxUsers": 25,
    "maxProjects": 15,
    "createdAt": "2024-01-01T00:00:00Z",
    "stats": {
      "totalUsers": 5,
      "totalProjects": 3,
      "totalTasks": 15
    }
  }
}
```

### Error Conditions

* 403 – Access denied
* 404 – Tenant not found

---

## 2.2 Modify Tenant Details

**Route:** `PUT /api/tenants/:tenantId`
**Access Level:** Authenticated
**Permission:** tenant_admin or super_admin

### Tenant Admin Request (Limited Fields)

```json
{
  "name": "Updated Company Name"
}
```

### Super Admin Request (Full Control)

```json
{
  "name": "Updated Company Name",
  "status": "active",
  "subscriptionPlan": "enterprise",
  "maxUsers": 100,
  "maxProjects": 50
}
```

### Successful Response (200 OK)

```json
{
  "success": true,
  "message": "Tenant updated successfully",
  "data": {
    "id": "uuid",
    "name": "Updated Company Name",
    "updatedAt": "2024-01-01T00:00:00Z"
  }
}
```

### Error Conditions

* 403 – Insufficient privileges
* 404 – Tenant not found

---

## 2.3 Retrieve All Tenants (Super Admin Only)

**Route:** `GET /api/tenants`
**Access Level:** Authenticated
**Permission:** super_admin only

### Query Parameters

* page (default: 1)
* limit (default: 10, max: 100)
* status (optional filter)
* subscriptionPlan (optional filter)

### Successful Response (200 OK)

```json
{
  "success": true,
  "data": {
    "tenants": [
      {
        "id": "uuid",
        "name": "Demo Company",
        "subdomain": "demo",
        "status": "active",
        "subscriptionPlan": "pro",
        "totalUsers": 5,
        "totalProjects": 3,
        "createdAt": "2024-01-01T00:00:00Z"
      }
    ],
    "pagination": {
      "currentPage": 1,
      "totalPages": 5,
      "totalTenants": 47,
      "limit": 10
    }
  }
}
```

### Error Conditions

* 403 – Access restricted to super_admin

---

The remaining sections (User Management, Project Management, Task Management, Error Codes, Pagination, Filtering, and Rate Limiting) follow the same structure and behavior as originally defined. Only wording and phrasing have been adjusted while keeping all endpoints, request bodies, response structures, and permission rules unchanged.

---

Error Code Reference

| Code | Meaning                        |
| ---- | ------------------------------ |
| 200  | Request processed successfully |
| 201  | Resource created successfully  |
| 400  | Invalid request data           |
| 401  | Unauthorized access            |
| 403  | Permission denied              |
| 404  | Resource not found             |
| 409  | Conflict due to duplicate data |
| 500  | Unexpected server error        |

---

Rate Limiting
Not currently implemented. Can be introduced using middleware such as express-rate-limit.

Pagination
Supported on list endpoints using page and limit parameters. Responses include pagination metadata.

Filtering and Search
Several listing endpoints allow filtering (status, role, priority, etc.) and case-insensitive search functionality.

