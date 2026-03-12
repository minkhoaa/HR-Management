# HR Management System

![Java](https://img.shields.io/badge/Java_17-ED8B00?style=for-the-badge&logo=openjdk&logoColor=white)
![Spring Boot](https://img.shields.io/badge/Spring_Boot_3.4-6DB33F?style=for-the-badge&logo=springboot&logoColor=white)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-4169E1?style=for-the-badge&logo=postgresql&logoColor=white)
![Spring Security](https://img.shields.io/badge/Spring_Security-6DB33F?style=for-the-badge&logo=springsecurity&logoColor=white)
![JWT](https://img.shields.io/badge/JWT-000000?style=for-the-badge&logo=jsonwebtokens&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-2496ED?style=for-the-badge&logo=docker&logoColor=white)
![Swagger](https://img.shields.io/badge/Swagger_UI-85EA2D?style=for-the-badge&logo=swagger&logoColor=black)
![Cloudinary](https://img.shields.io/badge/Cloudinary-3448C5?style=for-the-badge&logo=cloudinary&logoColor=white)

A RESTful back-end API for managing the full employee lifecycle — from recruitment and onboarding through attendance, payroll, performance evaluation, and offboarding.

---

## Table of Contents

- [Features](#features)
- [Tech Stack](#tech-stack)
- [Architecture](#architecture)
- [API Overview](#api-overview)
- [Getting Started](#getting-started)
  - [Prerequisites](#prerequisites)
  - [Local Setup](#local-setup)
  - [Docker](#docker)
- [Environment Variables](#environment-variables)
- [Authentication](#authentication)

---

## Features

| Domain | Capabilities |
|---|---|
| **Employee Management** | Create, read, update, and delete employee profiles with personal details, job title, department, and education level |
| **Department & Division** | Manage departments (PhongBan) and sub-divisions (BoPhan) with full CRUD support |
| **Job Titles & Education** | Maintain a catalog of positions (ChucVu) and qualification levels (TrinhDo) |
| **Attendance Tracking** | Employee clock-in / clock-out with date filtering; admin can correct attendance records |
| **Payroll Calculation** | Auto-calculate monthly payslips based on attendance, base salary derived from job title, allowances, and insurance deductions |
| **Insurance** | Record and manage employee insurance policies linked to each employee |
| **Labor Contracts** | Create, update, and archive employment contracts with start/end dates and terms |
| **Performance Evaluation** | Add and retrieve performance assessments (DanhGia) per employee |
| **Recruitment** | Post job openings, track candidate applications, and accept candidates with a single action |
| **Resignation Workflow** | Employees submit resignation requests; managers approve or decline with a written reason |
| **User Account Management** | Register accounts, assign roles, and retrieve the currently authenticated user's information |
| **File Uploads** | Upload and store employee profile images via Cloudinary |
| **OAuth2 Social Login** | Sign in with Google (or any configured OAuth2 provider); a JWT is automatically issued on success |

---

## Tech Stack

| Layer | Technology |
|---|---|
| Language | Java 17 |
| Framework | Spring Boot 3.4 |
| Security | Spring Security, JWT (JJWT 0.11.5), OAuth2 Client |
| Persistence | Spring Data JPA, Hibernate, PostgreSQL |
| Validation | Hibernate Validator 8 |
| API Documentation | SpringDoc OpenAPI (Swagger UI) |
| File Storage | Cloudinary |
| Build Tool | Maven 3.9 |
| Containerisation | Docker (multi-stage build, eclipse-temurin:17-jre-alpine) |
| Utilities | Lombok, java-dotenv |

---

## Architecture

The project follows a layered, domain-driven package structure:

```
com.example.demo
├── config/          # Infrastructure configuration (Cloudinary, etc.)
├── controller/      # REST controllers — one per domain aggregate
├── dto/
│   ├── request/     # Inbound payloads (input DTOs)
│   └── response/    # Outbound payloads (response DTOs / ApiResponse wrapper)
├── entity/          # JPA entities mapped to PostgreSQL tables
├── exception/       # Custom exception types
├── repository/      # Spring Data JPA repositories
├── security/        # JWT filter, SecurityConfig, Swagger auth setup, entry points
└── service/         # Business logic — one service class per domain
```

**Request flow:**

```
Client
  └─> JwtAuthFilter (validates Bearer token)
        └─> Controller (validates input, delegates)
              └─> Service (business rules, salary calculations, workflow state)
                    └─> Repository (JPA queries against PostgreSQL)
                          └─> Entity / DTO mapping
                                └─> ApiResponse (uniform JSON envelope)
```

All protected endpoints require a valid JWT passed as `Authorization: Bearer <token>`. The `/api/auth/**`, Swagger UI, and OAuth2 callback paths are publicly accessible.

---

## API Overview

| Prefix | Domain |
|---|---|
| `/api/auth` | Registration, login, token introspection, current user info |
| `/api/nhanvien` | Employee CRUD |
| `/api/phongban` | Department CRUD |
| `/api/bophan` | Division CRUD |
| `/api/chucvu` | Job title CRUD |
| `/api/trinhdo` | Education level CRUD |
| `/api/chamcong` | Attendance check-in / check-out / correction |
| `/api/luong` | Payslip listing and monthly payroll calculation |
| `/api/baohiem` | Insurance record CRUD |
| `/api/hopdong` | Labor contract CRUD |
| `/api/danhgia` | Performance evaluation |
| `/api/tuyendung` | Recruitment and candidate management |
| `/api/nghiviec` | Resignation request workflow |
| `/api/cloudinary` | File upload |

Full interactive documentation is available at `http://localhost:8080/swagger-ui/index.html` once the application is running.

---

## Getting Started

### Prerequisites

- Java 17+
- Maven 3.9+
- PostgreSQL 14+ (database must exist before first run)
- Docker (optional, for containerised deployment)

### Local Setup

1. **Clone the repository**

   ```bash
   git clone https://github.com/minkhoaa/HR-Management.git
   cd HR-Management
   ```

2. **Create a `.env` file** in the project root (see [Environment Variables](#environment-variables)).

3. **Create the PostgreSQL database**

   ```sql
   CREATE DATABASE hr_management;
   ```

4. **Run the application**

   ```bash
   ./mvnw spring-boot:run
   ```

   Spring Boot will run on `http://localhost:8080`.

### Docker

A multi-stage `Dockerfile` is included. It builds the JAR in a Maven image and then copies it into a lightweight JRE Alpine image.

```bash
# Build the image
docker build -t hr-management:latest .

# Run the container (pass env vars at runtime)
docker run -p 8080:8080 \
  --env-file .env \
  hr-management:latest
```

---

## Environment Variables

The application uses `java-dotenv` to read a `.env` file at startup. Create a `.env` in the project root with the following keys:

```dotenv
# PostgreSQL
DB_URL=jdbc:postgresql://localhost:5432/hr_management
DB_USERNAME=your_db_user
DB_PASSWORD=your_db_password

# JWT
JWT_SECRET=your_jwt_secret_key_minimum_256_bits

# Cloudinary
CLOUDINARY_CLOUD_NAME=your_cloud_name
CLOUDINARY_API_KEY=your_api_key
CLOUDINARY_API_SECRET=your_api_secret

# OAuth2 (Google)
GOOGLE_CLIENT_ID=your_google_client_id
GOOGLE_CLIENT_SECRET=your_google_client_secret
```

---

## Authentication

The API supports two authentication mechanisms:

**1. Username / Password (JWT)**

```http
POST /api/auth/login
Content-Type: application/json

{
  "username": "admin",
  "password": "password"
}
```

The response contains a `token` field. Pass it in subsequent requests:

```http
Authorization: Bearer <token>
```

**2. Google OAuth2**

Navigate to `/oauth2/authorization/google`. After a successful Google login, the server issues a JWT and redirects to the configured frontend callback URL with the token as a query parameter.

**Token introspection**

```http
POST /api/auth/authentication
Content-Type: application/json

{ "token": "<token>" }
```

Returns the resolved user details associated with the provided token.
