# BBHealthApp Project Documentation

## Project Overview

BBHealthApp is a COVID-19 health management system built with a microservices architecture. The application allows users to register as patients, doctors, testers, or government authorities, and manage COVID-19 testing, consultation, and document workflows.

### Technology Stack

**Frontend:**
- React.js
- Material-UI
- Axios for HTTP requests
- RxJS for reactive programming

**Backend (Microservices):**
- Spring Boot (Java)
- Spring Security with JWT authentication
- H2 Database (development) / MySQL (production)
- Maven for build management
- Docker for containerization

**Infrastructure:**
- Docker Compose for orchestration
- Apache HTTP Server for frontend serving

---

## Architecture

### Microservices Architecture

The application consists of four main services:

```
┌─────────────────┐
│   Frontend      │
│   (React)       │
│   Port: 80      │
└────────┬────────┘
         │
         ├─────────────────┐
         │                 │
         ▼                 ▼
┌─────────────────┐  ┌─────────────────┐
│  Master Service │  │ Register Service│
│  Port: 8081     │  │  Port: 8082     │
│  - Auth         │  │  - Registration │
│  - User Mgmt    │  │  - User/Doctor/ │
│  - Test Requests│  │    Tester Reg   │
└─────────────────┘  └─────────────────┘
         │                 │
         └─────────────────┤
                       │
                       ▼
              ┌─────────────────┐
              │ Document Service│
              │  Port: 8083     │
              │  - File Upload  │
              │  - File Download│
              └─────────────────┘
```

### Service Responsibilities

**1. Master Service (Port 8080/8081)**
- User authentication and authorization
- JWT token generation and validation
- User profile management
- Test request management
- Consultation management
- Dashboard and analytics
- Government authority approvals

**2. Register Service (Port 8082)**
- User registration
- Doctor registration
- Tester registration
- User validation

**3. Document Service (Port 8080/8083)**
- Document upload
- Document download
- File storage management

**4. Frontend (Port 80)**
- React-based user interface
- User authentication flow
- Role-based dashboards
- Test request submission
- Document management

### Database

**Development:**
- H2 File-based database (`binarybrainscovid.mv.db`)
- Located at: `/root/binarybrainscovid.mv.db` (in container)
- H2 Console available at: `http://localhost:8081/h2-console`

**Production:**
- AWS RDS MySQL (configured but not used in current setup)

---

## Changes Made During Docker Migration

### 1. Docker Compose Configuration

**File:** `docker-compose.yml`

**Changes:**
- Created comprehensive Docker Compose configuration
- Configured all four services with proper dependencies
- Set up network isolation using custom bridge network
- Configured volume mounts for data persistence
- Set environment variables for service communication

**Port Mappings:**
```
Frontend:        80:80
Master Service:  8081:8080
Register Service: 8082:8082
Document Service: 8083:8080
```

### 2. Backend Configuration Changes

**Master Service (`application-dev.properties`):**
- Explicitly set `server.port=8080`
- Configured H2 database for development
- Enabled H2 console

**Register Service (`application-dev.properties`):**
- Changed `server.port` from 8080 to 8082
- Updated `rootURL` to use `localhost` for local development
- Configured H2 database

**Document Service (`application-dev.properties`):**
- Set `server.port=8080`
- Updated `rootURL` to point to master service on port 8080

### 3. Frontend Configuration Changes

**File:** `.env`

**Changes:**
- Initially set to Docker service names (`register-service`, `master-service`, `document-service`)
- Reverted to `localhost` for browser-accessible URLs
- Final configuration:
  ```
  REACT_APP_REGSVC_HOST=localhost
  REACT_APP_DOCSVC_HOST=localhost
  REACT_APP_MASTERSVC_HOST=localhost
  ```

**File:** `src/environment.js`

**Changes:**
- Fixed `LOGIN_URL` to point to master service (port 8081) instead of register service (port 8082)
- Corrected URL construction to avoid duplicate `http://` prefixes

### 4. Code Fixes

**AuthController.java**
- Added missing package declaration: `package org.binarybrains.bbhealthapp.auth;`

**AuthService.java (Created)**
- Created missing service class for authentication logic
- Implemented login flow with authentication manager
- Added user approval check
- Integrated JWT token generation

**LoginResponse.java**
- Fixed `getToken()` method (was hardcoded to return `null`)
- Changed to return actual token value: `return token;`

---

## Errors Encountered and Resolutions

### Error 1: Missing Package Declaration

**Error:** Compilation error in `AuthController.java`
```
AuthController.java:1: error: class, interface, or enum expected
```

**Cause:** Missing package declaration at the top of the file

**Resolution:** Added `package org.binarybrains.bbhealthapp.auth;` to `AuthController.java`

---

### Error 2: Port Conflicts

**Error:** 
```
Port 8080 was already in use. Tomcat started on port(s): 8080 (http)
```

**Cause:** Multiple services trying to use the same port (8080)

**Resolution:**
- Explicitly set different ports in each service's configuration
- Master Service: 8080
- Register Service: 8082
- Document Service: 8080 (container) / 8083 (host)

---

### Error 3: Duplicate `http://` in URLs

**Error:** 
```
Request URL: http://http//localhost:8082/auth/login
```

**Cause:** 
- Frontend environment variables had `http://` prefix
- `environment.js` was adding another `http://` prefix
- Result: `http://http//localhost:8082/auth/login`

**Resolution:**
- Removed `http://` prefix from `.env` file
- Set environment variables to just hostnames: `localhost`
- Rebuilt frontend container with `--no-cache` flag

---

### Error 4: 404 Not Found on Login

**Error:**
```
POST http://localhost:8082/auth/login 404 (Not Found)
```

**Cause:** Login endpoint was configured to point to register service (port 8082), but the actual login endpoint is on master service (port 8081)

**Resolution:**
- Updated `environment.js` to point `LOGIN_URL` to master service:
  ```javascript
  LOGIN_URL: `http://${masterSvc}:8081/auth/login`
  ```
- Rebuilt frontend container

---

### Error 5: Null Token on Login

**Error:**
```json
{
  "userName": "tester",
  "message": "Success",
  "token": null
}
```

**Cause:** 
1. `AuthService` class was missing entirely
2. `LoginResponse.getToken()` method was hardcoded to return `null`
3. AuthService was using wrong LoginResponse constructor

**Resolution:**
1. Created `AuthService.java` with proper authentication logic:
   - Uses `AuthenticationManager` for authentication
   - Checks user approval status
   - Generates JWT token using `TokenProvider`
2. Fixed `LoginResponse.getToken()` to return actual token value
3. Updated AuthService to use correct constructor: `new LoginResponse(userName, "Success", token)`

---

### Error 6: 401 Unauthorized on User Details

**Error:**
```json
{
  "timestamp": "2026-05-23T09:01:34.344+00:00",
  "status": 401,
  "error": "Unauthorized",
  "message": "Unauthorized",
  "path": "/users/details"
}
```

**Cause:** Login was returning null token, so subsequent requests to protected endpoints failed authentication

**Resolution:** Fixed null token issue (see Error 5 above)

---

### Error 7: H2 Database Version Incompatibility

**Error:**
```
Unsupported database file version or invalid file header
```

**Cause:** Using H2 JAR version 2.1.214 to query database file created with older version

**Resolution:** Downloaded and used H2 JAR version 1.4.200 for compatibility

---

### Error 8: Browser Caching

**Error:** Frontend still using old JavaScript after rebuilds

**Cause:** Browser caching old JavaScript files

**Resolution:** 
- Instructed user to perform hard refresh (`Cmd + Shift + R` on Mac, `Ctrl + Shift + R` on Windows)
- Alternatively use incognito/private mode

---

## Use Cases

### 1. User Registration

**Actors:** Patient, Doctor, Tester, Government Authority

**Flow:**
1. User navigates to registration page
2. Selects role (User/Doctor/Tester)
3. Fills in personal information (name, email, phone, address, etc.)
4. Submits registration form
5. System validates user doesn't already exist
6. User is created in database with "pending" approval status
7. Government authority approves user

**Endpoints:**
- `POST /auth/register` - Patient registration
- `POST /auth/doctor/register` - Doctor registration
- `POST /auth/tester/register` - Tester registration

---

### 2. User Login

**Actors:** All registered users

**Flow:**
1. User navigates to login page
2. Enters username and password
3. System authenticates credentials
4. System checks if user is approved
5. System generates JWT token
6. Token is returned to frontend
7. Frontend stores token for subsequent requests
8. User is redirected to role-specific dashboard

**Endpoints:**
- `POST /auth/login` - User authentication

---

### 3. Test Request Submission

**Actors:** Patient

**Flow:**
1. Patient logs in
2. Navigates to "Request Test" page
3. Fills in test request details
4. Submits test request
5. Request is assigned to a tester
6. Tester receives notification
7. Tester performs test and updates results

**Endpoints:**
- `POST /test-requests` - Create test request
- `GET /test-requests/pending` - Get pending tests (for tester)
- `PUT /test-requests/{id}/result` - Update test result

---

### 4. Consultation Request

**Actors:** Patient, Doctor

**Flow:**
1. Patient submits consultation request
2. Doctor receives notification
3. Doctor reviews patient details and test results
4. Doctor provides consultation
5. Consultation details are saved

**Endpoints:**
- `POST /consultations` - Create consultation request
- `GET /consultations/pending` - Get pending consultations (for doctor)
- `PUT /consultations/{id}` - Update consultation

---

### 5. Document Management

**Actors:** All users

**Flow:**
1. User uploads document (medical report, ID proof, etc.)
2. Document is stored in document service
3. User can download documents when needed
4. Documents are associated with user profile

**Endpoints:**
- `POST /documents/upload` - Upload document
- `GET /documents/download` - Download document

---

### 6. Dashboard and Analytics

**Actors:** Government Authority

**Flow:**
1. Authority logs in
2. Views dashboard with statistics:
   - Positive/Negative test distribution
   - Home Quarantine/Admission distribution
   - Pincode-based distribution
3. Approves pending user registrations
4. Views all test requests
5. Manages system thresholds

**Endpoints:**
- `GET /dashboard` - Get dashboard statistics
- `GET /users/pending-approvals` - Get pending user approvals
- `PUT /users/{id}/approve` - Approve user
- `GET /test-requests` - Get all test requests
- `PUT /thresholds` - Update system thresholds

---

## Default User Credentials

The application is initialized with default users for testing:

| Username | Password | Role       | Status  |
|----------|----------|------------|---------|
| user     | password | Patient    | Approved|
| doctor   | password | Doctor     | Approved|
| tester   | password | Tester     | Approved|
| authority| password | Government | Approved|

**Note:** All default users have the same password: `password`

---

## Database Access

### H2 Console Access

**URL:** `http://localhost:8081/h2-console`

**Connection Settings:**
- **JDBC URL:** `jdbc:h2:file:~/binarybrainscovid`
- **User Name:** `sa`
- **Password:** `sa`

### Key Database Tables

- **USER** - User information and credentials
- **ROLE** - User roles (USER, DOCTOR, TESTER, AUTHORITY)
- **TEST_REQUEST** - COVID-19 test requests
- **CONSULTATION** - Doctor consultation records
- **DOCUMENT** - Uploaded documents

---

## Running the Application

### Prerequisites

- Docker installed
- Docker Compose installed

### Start All Services

```bash
docker-compose up -d --build
```

### Stop All Services

```bash
docker-compose down
```

### View Logs

```bash
# All services
docker-compose logs -f

# Specific service
docker-compose logs -f master-service
docker-compose logs -f register-service
docker-compose logs -f document-service
docker-compose logs -f frontend
```

### Access Services

- **Frontend:** http://localhost
- **Master Service:** http://localhost:8081
- **Register Service:** http://localhost:8082
- **Document Service:** http://localhost:8083
- **H2 Console:** http://localhost:8081/h2-console

---

## Security Considerations

### JWT Authentication

- Tokens are generated using HS256 algorithm
- Token secret is configured in `application.properties`
- Tokens include user subject, roles, and expiration time
- Tokens are validated on each protected endpoint request

### CORS Configuration

- Cross-origin requests are allowed from all origins (`@CrossOrigin("*")`)
- In production, this should be restricted to specific domains

### Password Storage

- Passwords are hashed using BCrypt
- Default password for all test users is "password" (should be changed in production)

---

## Future Improvements

1. **Security:**
   - Restrict CORS to specific domains
   - Implement refresh token mechanism
   - Add rate limiting for authentication endpoints
   - Change default passwords

2. **Infrastructure:**
   - Add health check endpoints
   - Implement centralized logging (ELK stack)
   - Add monitoring and alerting (Prometheus/Grafana)
   - Use Kubernetes for orchestration instead of Docker Compose

3. **Database:**
   - Migrate to production database (PostgreSQL/MySQL)
   - Add database migration scripts (Flyway/Liquibase)
   - Implement database backup strategy

4. **Testing:**
   - Add integration tests
   - Add end-to-end tests
   - Implement API documentation (Swagger/OpenAPI)

5. **Frontend:**
   - Add error boundary components
   - Implement lazy loading for better performance
   - Add internationalization support

---

## Conclusion

This project demonstrates a microservices architecture for a COVID-19 health management system. The Docker-based deployment ensures consistent environments across development and production. The application successfully handles user registration, authentication, test requests, consultations, and document management through well-defined REST APIs.

The migration to Docker required careful configuration of service communication, port management, and environment variables. Several issues were encountered and resolved, including missing service classes, hardcoded null returns, URL construction errors, and browser caching issues.

The application is now fully functional and can be deployed using Docker Compose with all services communicating correctly.
