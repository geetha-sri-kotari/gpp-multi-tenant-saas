# Technical Specification

## Multi-Tenant SaaS Platform

---

## 1. Project Organization

### Backend Directory Layout

```
backend/
├── src/
│   ├── config/
│   │   ├── database.js      # PostgreSQL connection configuration
│   │   └── jwt.js           # JWT setup and configuration
│   ├── controllers/
│   │   ├── authController.js      # Handles authentication logic
│   │   ├── tenantController.js    # Tenant-related operations
│   │   ├── userController.js      # User-related operations
│   │   ├── projectController.js   # Project-related operations
│   │   └── taskController.js      # Task-related operations
│   ├── middleware/
│   │   └── auth.js          # Authentication and authorization middleware
│   ├── routes/
│   │   ├── authRoutes.js    # Authentication routes
│   │   ├── tenantRoutes.js  # Tenant routes
│   │   ├── userRoutes.js    # User routes
│   │   ├── projectRoutes.js # Project routes
│   │   └── taskRoutes.js    # Task routes
│   ├── utils/
│   │   ├── runMigrations.js # Script to execute migrations
│   │   ├── runSeeds.js      # Script to insert seed data
│   │   └── auditLogger.js   # Utility for audit logging
│   └── server.js            # Application entry point
├── migrations/              # SQL migration scripts (optional)
├── seeds/                   # Seed files (optional)
├── Dockerfile               # Backend container configuration
└── package.json             # Project dependencies and scripts
```

---

### Frontend Directory Layout

```
frontend/
├── src/
│   ├── components/
│   │   └── Navbar.jsx           # Navigation bar component
│   ├── pages/
│   │   ├── Register.jsx         # Tenant registration screen
│   │   ├── Login.jsx            # Authentication screen
│   │   ├── Dashboard.jsx        # Dashboard view
│   │   ├── Projects.jsx         # Project listing view
│   │   ├── ProjectDetails.jsx   # Detailed project view
│   │   └── Users.jsx            # User listing view
│   ├── services/
│   │   └── api.js               # API communication layer
│   ├── context/
│   │   └── AuthContext.jsx      # Authentication state management
│   ├── utils/
│   │   └── ProtectedRoute.jsx   # Route guarding component
│   ├── App.jsx                  # Root React component
│   ├── main.jsx                 # Application bootstrap file
│   └── index.css                # Global stylesheet
├── index.html                   # HTML entry template
├── vite.config.js               # Vite build configuration
├── Dockerfile                   # Frontend container configuration
└── package.json                 # Project dependencies and scripts
```

---

### Database Directory Layout

```
database/
├── migrations/
│   ├── 001_create_tenants.sql
│   ├── 002_create_users.sql
│   ├── 003_create_projects.sql
│   ├── 004_create_tasks.sql
│   └── 005_create_audit_logs.sql
└── seeds/
    └── seed_data.sql            # Seed dataset (password hashes generated dynamically)
```

---

### Root Directory Layout

```
saas-multitennant/
├── backend/                # Backend service
├── frontend/               # Frontend client
├── database/               # Migrations and seed scripts
├── docs/                   # Documentation files
├── docker-compose.yml      # Container orchestration
├── submission.json         # Sample credentials for evaluation
└── README.md               # Project overview
```

---

## 2. Development Setup Instructions

### Required Tools

* Node.js version 18 or above
* PostgreSQL version 15 or above (for local execution)
* Docker version 20.10 or above
* Docker Compose version 2.0 or above
* Git for version control

---

## Local Setup (Without Docker)

### Step 1: Configure Database

Install PostgreSQL 15 and execute:

```sql
CREATE DATABASE saas_db;
CREATE USER postgres WITH PASSWORD 'postgres';
GRANT ALL PRIVILEGES ON DATABASE saas_db TO postgres;
```

---

### Step 2: Backend Configuration

1. Move to backend folder:

```bash
cd backend
```

2. Install required packages:

```bash
npm install
```

3. Create a `.env` file:

```env
DB_HOST=localhost
DB_PORT=5432
DB_NAME=saas_db
DB_USER=postgres
DB_PASSWORD=postgres
JWT_SECRET=your_jwt_secret_key_min_32_chars
JWT_EXPIRES_IN=24h
PORT=5000
NODE_ENV=development
FRONTEND_URL=http://localhost:3000
```

4. Execute migrations:

```bash
npm run migrate
```

5. Insert seed data:

```bash
npm run seed
```

6. Start development server:

```bash
npm run dev
```

Backend will run at:

```
http://localhost:5000
```

---

### Step 3: Frontend Configuration

1. Navigate to frontend directory:

```bash
cd frontend
```

2. Install dependencies:

```bash
npm install
```

3. Optional `.env` configuration:

```env
VITE_API_URL=http://localhost:5000/api
```

4. Launch development server:

```bash
npm run dev
```

Frontend will be accessible at:

```
http://localhost:3000
```

---

## Docker-Based Setup (Preferred)

### Step 1: Confirm Docker Installation

```bash
docker --version
docker-compose --version
```

---

### Step 2: Launch All Services

From the project root directory:

```bash
docker-compose up -d
```

This command will:

* Initialize PostgreSQL
* Apply database migrations
* Populate seed data
* Start backend service
* Start frontend service

---

### Step 3: Verify Containers

```bash
docker-compose ps
```

Ensure all containers display an “Up” status.

---

### Step 4: Access the Application

* Frontend: [http://localhost:3000](http://localhost:3000)
* Backend API: [http://localhost:5000](http://localhost:5000)
* Health Endpoint: [http://localhost:5000/api/health](http://localhost:5000/api/health)

---

### Step 5: Monitor Logs

```bash
docker-compose logs -f
docker-compose logs -f backend
docker-compose logs -f frontend
docker-compose logs -f database
```

---

### Step 6: Shut Down Services

```bash
docker-compose down
```

To remove volumes and reset the database:

```bash
docker-compose down -v
```

---

## 3. Configuration Variables

### Backend Environment Variables

| Variable       | Purpose                  | Default                                        | Mandatory |
| -------------- | ------------------------ | ---------------------------------------------- | --------- |
| DB_HOST        | Database server host     | localhost                                      | Yes       |
| DB_PORT        | Database port            | 5432                                           | Yes       |
| DB_NAME        | Database name            | saas_db                                        | Yes       |
| DB_USER        | Database username        | postgres                                       | Yes       |
| DB_PASSWORD    | Database password        | -                                              | Yes       |
| JWT_SECRET     | Secret for token signing | -                                              | Yes       |
| JWT_EXPIRES_IN | Token validity duration  | 24h                                            | No        |
| PORT           | Application port         | 5000                                           | No        |
| NODE_ENV       | Environment mode         | development                                    | No        |
| FRONTEND_URL   | Allowed frontend origin  | [http://localhost:3000](http://localhost:3000) | Yes       |

---

### Frontend Environment Variables

| Variable     | Purpose              | Default                                                | Mandatory |
| ------------ | -------------------- | ------------------------------------------------------ | --------- |
| VITE_API_URL | Backend API base URL | [http://localhost:5000/api](http://localhost:5000/api) | No        |

---

## 4. Database Migration Process

Migrations are automatically triggered when running in Docker. For manual execution:

```bash
npm run migrate
```

or

```bash
node src/utils/runMigrations.js
```

Migration files are executed sequentially in the `database/migrations/` directory.

---

## 5. Seed Data Process

Seeds run automatically in Docker. For manual execution:

```bash
npm run seed
```

or

```bash
node src/utils/runSeeds.js
```

Seed dataset contains:

* One Super Administrator
* One Demo Tenant
* One Tenant Administrator
* Two Standard Users
* Two Projects
* Five Tasks

Refer to `submission.json` for login credentials.

---

## 6. Testing Procedures

### Health Endpoint

```bash
curl http://localhost:5000/api/health
```

Expected output:

```json
{
  "status": "ok",
  "database": "connected"
}
```

---

### Login Request

```bash
curl -X POST http://localhost:5000/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{
    "email": "admin@demo.com",
    "password": "Demo@123",
    "tenantSubdomain": "demo"
  }'
```

---

### Retrieve Current User

```bash
curl http://localhost:5000/api/auth/me \
  -H "Authorization: Bearer YOUR_TOKEN_HERE"
```

---

## 7. Troubleshooting Guide

### Database Connectivity Issues

Possible fixes:

* Confirm database container is active
* Inspect database logs
* Validate environment variables
* Ensure database is ready before backend initialization

---

### Migration Failures

Possible solutions:

* Verify database accessibility
* Confirm migration files exist
* Review backend logs
* Execute migrations manually

---

### CORS Problems

Possible fixes:

* Confirm `FRONTEND_URL` matches frontend address
* In Docker use service name instead of localhost
* Review CORS settings in `server.js`

---

### Port Conflicts

If ports 3000, 5000, or 5432 are occupied:

* Stop conflicting services
* Modify ports in `docker-compose.yml`
* Update frontend API URL accordingly

---

## 8. Production Deployment Guidelines

### Security Recommendations

* Store secrets securely using a secrets manager
* Enforce HTTPS with valid SSL certificates
* Generate strong JWT secret keys
* Use strong database credentials
* Restrict CORS to production frontend domain

---

### Performance Optimization

* Ensure database indexes are configured
* Adjust connection pool settings
* Introduce caching with Redis
* Deliver static content through CDN

---

### Monitoring and Observability

* Continuously monitor health endpoint
* Centralize logs (e.g., ELK stack, CloudWatch)
* Track metrics (Prometheus, DataDog)
* Configure alerting mechanisms

---

### Scaling Strategy

* Add additional backend instances behind a load balancer
* Configure database read replicas
* Implement caching layer for high-read operations