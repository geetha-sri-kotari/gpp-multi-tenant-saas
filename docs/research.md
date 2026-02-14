# Research Document – Multi-Tenant SaaS Platform

---

# 1. Multi-Tenancy Evaluation

Multi-tenancy is an architectural pattern where a single application instance supports multiple customers (tenants). Although infrastructure is shared, each tenant’s data remains logically separated and inaccessible to others. There are three common implementation models:

---

## Model 1: Shared Database with Shared Schema (tenant_id column)

### Overview

All tenants use the same database and schema. Data separation is implemented by including a `tenant_id` field in every table.

### Advantages

* Minimal infrastructure cost due to a single database instance
* Simple onboarding of new tenants without schema changes
* Straightforward backup and maintenance
* Efficient use of system resources
* Faster tenant provisioning
* Simplifies cross-tenant reporting for super administrators

### Disadvantages

* Potential data exposure if tenant filtering is omitted in queries
* Performance may decline as tenant count increases
* Limited ability to customize schema per tenant
* Backup and restore operations affect all tenants
* Harder to scale individual tenants independently

### Suitable For

Applications where tenants share similar data structures, small to medium organizations, and cost-sensitive deployments.

---

## Model 2: Shared Database with Separate Schemas

### Overview

Tenants share one database instance, but each tenant has a dedicated schema (namespace).

### Advantages

* Stronger logical separation compared to shared schema
* Allows limited schema customization per tenant
* Easier tenant-specific migrations
* Improved performance isolation
* Supports tenant-level feature variations

### Disadvantages

* Increased operational overhead
* Larger number of database objects to maintain
* Schema updates must be replicated across all tenants
* Backup and restoration processes become more complex
* Greater resource consumption

### Suitable For

Applications requiring moderate customization and stronger isolation, especially for medium or large tenants.

---

## Model 3: Dedicated Database per Tenant

### Overview

Each tenant operates within its own independent database instance.

### Advantages

* Highest level of isolation and security
* Full customization capabilities
* Independent migration and scaling
* Strong compliance support (e.g., GDPR requirements)
* Best performance isolation

### Disadvantages

* Highest infrastructure and maintenance costs
* Complex operational management
* More difficult to maintain consistency across tenants
* Resource-intensive setup
* Complex deployment and upgrade process
* Increased backup storage requirements

### Suitable For

Large enterprise customers with strict regulatory or customization needs.

---

## Comparative Summary

| Criteria             | Shared DB + Shared Schema | Shared DB + Separate Schema | Separate Database |
| -------------------- | ------------------------- | --------------------------- | ----------------- |
| Cost                 | Low                       | Moderate                    | High              |
| Data Isolation       | Moderate                  | High                        | Very High         |
| Scalability          | High                      | Moderate                    | Lower             |
| Operational Overhead | Low                       | Moderate                    | High              |
| Performance          | Good (with indexing)      | Good                        | Excellent         |
| Customization        | Limited                   | Moderate                    | Extensive         |
| Backup Complexity    | Low                       | Moderate                    | High              |
| Deployment Speed     | Fast                      | Moderate                    | Slow              |

---

## Selected Model: Shared Database with Shared Schema

### Rationale

For this platform, the shared database and shared schema model was selected for the following reasons:

1. **Cost Optimization**
   Maintaining a single database significantly reduces infrastructure expenses.

2. **Architectural Simplicity**
   All tenants share identical data structures (users, projects, tasks), eliminating the need for schema customization.

3. **Rapid Onboarding**
   New tenants can be provisioned instantly without additional database setup.

4. **Simplified Maintenance**
   Migrations and backups are executed once for all tenants, reducing operational complexity.

5. **Efficient Resource Allocation**
   Shared infrastructure prevents underutilized resources, especially for smaller tenants.

6. **Cross-Tenant Reporting**
   Super admin features, such as viewing all tenants, are easier to implement.

---

### Risk Mitigation Strategies

To address potential data leakage risks:

* Enforced middleware automatically applies tenant filtering
* Unique database constraints such as `UNIQUE(tenant_id, email)`
* Strict authorization checks in all endpoints
* Comprehensive audit logging including tenant identifiers
* Strong input validation
* Mandatory code reviews to verify tenant filtering in queries

This approach balances affordability, maintainability, and security.

---

# 2. Technology Stack Rationale

---

## Backend: Node.js with Express.js

### Selected Version

Node.js 18 with Express 4.18.2

### Reasons for Selection

* Unified JavaScript ecosystem for both frontend and backend
* Strong performance for concurrent I/O operations
* Extensive NPM library ecosystem
* Lightweight framework enabling rapid API development
* Large community support
* Easy migration to microservices in the future

### Alternatives Evaluated

* Python/Django: Slower for I/O-intensive operations
* Java/Spring Boot: Robust but more verbose
* Go: High performance but smaller ecosystem

---

## Frontend: React with Vite

### Selected Version

React 18.2.0 with Vite 5.0.8

### Reasons for Selection

* Component-driven UI structure
* Large ecosystem of supporting libraries
* Excellent development tooling
* Efficient rendering via virtual DOM
* Fast build and development experience with Vite
* Strong industry adoption

### Alternatives Evaluated

* Vue.js: Simpler but smaller ecosystem
* Angular: Feature-rich but complex
* Svelte: Lightweight but limited community support

---

## Database: PostgreSQL

### Selected Version

PostgreSQL 15

### Reasons for Selection

* ACID compliance ensures transactional integrity
* Advanced features such as JSON support and indexing
* Strong foreign key enforcement
* Optimized query planner
* Open-source and production-proven
* Supports row-level security (future enhancement)

### Alternatives Evaluated

* MySQL: Comparable but fewer advanced features
* MongoDB: Flexible but lacks strict relational guarantees
* SQLite: Not suitable for production multi-tenant systems

---

## Authentication: JWT (JSON Web Tokens)

### Implementation

jsonwebtoken library

### Reasons for Selection

* Stateless authentication enables horizontal scaling
* Fast validation without repeated database lookups
* Token signing ensures integrity
* Supports expiration controls
* Widely adopted standard
* Suitable for web and mobile clients

### Alternatives Evaluated

* Session-based authentication: Requires server-side storage
* OAuth 2.0: Unnecessary complexity
* API keys: Limited security controls

---

## Deployment: Docker with Docker Compose

### Reasons for Selection

* Environment consistency across stages
* Service isolation via containers
* Platform portability
* Simplified scaling
* Encapsulated dependencies
* Single-command startup
* Version-controlled infrastructure

### Alternatives Evaluated

* Kubernetes: Too complex for current scope
* Vagrant: Less efficient
* Manual deployment: Error-prone

---

# 3. Security Architecture

---

## Data Isolation Strategy

### Implementation Details

* All tables (except super admin users) include `tenant_id`
* Queries automatically apply tenant filtering from JWT
* Unique constraints prevent cross-tenant conflicts
* Middleware enforces isolation before database access
* Super admin users have `tenant_id = NULL`

### Best Practices

* Ignore tenant_id from client requests
* Always derive tenant_id from authenticated token
* Index tenant_id for performance
* Use cascading foreign keys to maintain integrity

---

## Authentication and Authorization Model

### Authentication

* JWT payload includes userId, tenantId, and role
* Tokens expire after 24 hours
* Passwords hashed with bcrypt (10 salt rounds)
* Login requires subdomain, email, and password
* Super admin login bypasses subdomain requirement

### Authorization

* Role-Based Access Control (RBAC)

  * Super Admin: Global privileges
  * Tenant Admin: Full tenant-level control
  * User: Limited operational permissions
* Middleware enforces role validation
* Subscription updates restricted to super admins
* Self-deletion restricted

---

## Password Security Approach

### Implementation

* bcrypt with minimum 10 salt rounds
* Unique salt per password
* Secure comparison via bcrypt.compare
* Minimum length enforced
* No plain-text storage
* Password hashes excluded from API responses

### Recommended Practices

* Avoid logging credentials
* Enforce HTTPS in production
* Introduce password complexity policies
* Add password reset mechanism

---

## API-Level Security Controls

### Validation

* express-validator used for input checks
* Email and subdomain format verification
* Enum validation for fields such as status and role
* Parameterized queries prevent SQL injection

### Error Handling

* Generic client-facing errors
* Detailed server-side logs
* Consistent error response structure

### CORS

* Restricted to configured frontend URL
* Configurable via environment variables

### Rate Limiting

* Can integrate express-rate-limit
* Prevents brute force attacks

---

## Audit Logging Framework

### Implementation

* All critical operations recorded in `audit_logs`
* Includes tenant_id, user_id, action type, entity details, IP, timestamp
* Logs CREATE, UPDATE, DELETE actions
* Authentication events logged
* Logs retained permanently

### Benefits

* Enables forensic investigation
* Supports regulatory compliance
* Tracks user activity
* Detects suspicious behavior

### Planned Enhancements

* Retention policies
* Automated anomaly detection
* Export functionality for compliance reporting

---

# Final Remarks

This multi-tenant SaaS platform applies layered security controls to guarantee data separation, reliable authentication, and strict authorization.

The selected shared database and shared schema model, reinforced by middleware-based tenant enforcement and comprehensive audit logging, provides a scalable and secure foundation capable of supporting a growing number of tenants while maintaining strict isolation and performance standards.
