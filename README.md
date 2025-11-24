# FastAPI Calculator Application with User Authentication and BREAD Operations

A production-ready FastAPI application featuring user authentication, calculation management with full CRUD operations, PostgreSQL database integration, and automated CI/CD deployment pipeline.

## Table of Contents

- [Overview](#overview)
- [Features](#features)
- [Architecture](#architecture)
- [Technology Stack](#technology-stack)
- [Project Structure](#project-structure)
- [Installation](#installation)
- [Running the Application](#running-the-application)
- [Database Models](#database-models)
- [API Endpoints](#api-endpoints)
- [Authentication](#authentication)
- [Testing](#testing)
- [CI/CD Pipeline](#cicd-pipeline)
- [Docker Deployment](#docker-deployment)
- [Manual Testing](#manual-testing)

## Overview

This project implements a comprehensive FastAPI-based calculator application with user authentication and complete BREAD (Browse, Read, Edit, Add, Delete) operations for calculations. It demonstrates advanced software engineering practices including JWT authentication, polymorphic database models, factory patterns, comprehensive testing strategies, and automated CI/CD deployment to Docker Hub.

The application supports user registration and login with secure password hashing, four arithmetic operations (addition, subtraction, multiplication, division) with database persistence, and user-specific calculation history management.

## Features

- **User Authentication**: Secure registration and login with JWT tokens
- **Password Security**: Bcrypt hashing with configurable rounds
- **User Management**: Complete user profile tracking with activity status
- **Calculation BREAD Operations**: Full create, read, update, delete functionality
- **RESTful API**: Clean, documented endpoints following REST principles
- **Database Persistence**: SQLAlchemy ORM with PostgreSQL
- **Polymorphic Models**: Elegant calculation hierarchy using inheritance
- **Factory Pattern**: Clean object creation for different calculation types
- **Pydantic Validation**: Comprehensive request/response validation
- **Multi-Layer Testing**: Unit, integration, and end-to-end tests
- **CI/CD Pipeline**: Automated testing, security scanning, and deployment
- **Interactive Documentation**: Auto-generated OpenAPI (Swagger) documentation
- **Production-Ready**: Docker containerization with health checks

## Architecture

The application follows a layered architecture pattern:

```
├── Presentation Layer (FastAPI Routes)
├── Authentication Layer (JWT Middleware)
├── Validation Layer (Pydantic Schemas)
├── Business Logic Layer (Operations & Models)
├── Data Access Layer (SQLAlchemy Models)
└── Database Layer (PostgreSQL)
```

Key architectural decisions:

- **Token-Based Authentication**: JWT tokens for stateless authentication
- **Polymorphic Single Table Inheritance**: All calculation types in one table
- **Factory Pattern**: Centralized calculation object creation
- **Repository Pattern**: Database access abstraction
- **Dependency Injection**: FastAPI's dependency system for auth and database

## Technology Stack

- **Framework**: FastAPI 0.115.8
- **Database**: PostgreSQL 17
- **ORM**: SQLAlchemy 2.0.38
- **Authentication**: python-jose (JWT), passlib (bcrypt)
- **Validation**: Pydantic 2.10.6
- **Testing**: pytest 8.3.4, pytest-cov 6.0.0, Playwright 1.50.0
- **Web Server**: Uvicorn 0.34.0
- **Containerization**: Docker with multi-platform builds
- **CI/CD**: GitHub Actions
- **Security**: Trivy vulnerability scanner

## Project Structure

```
.
├── .github/
│   └── workflows/
│       └── python-app.yml          # CI/CD pipeline
├── app/
│   ├── auth/
│   │   ├── dependencies.py         # Auth dependencies
│   │   ├── jwt.py                  # JWT token handling
│   │   └── redis.py                # Token blacklisting
│   ├── core/
│   │   └── config.py               # Application settings
│   ├── models/
│   │   ├── calculation.py          # Polymorphic calculation models
│   │   └── user.py                 # User model with auth
│   ├── operations/
│   │   └── __init__.py             # Arithmetic operations
│   ├── schemas/
│   │   ├── calculation.py          # Calculation validation
│   │   ├── token.py                # Token schemas
│   │   └── user.py                 # User validation
│   ├── database.py                 # Database configuration
│   ├── database_init.py            # Database initialization
│   └── main.py                     # Application entry point
├── tests/
│   ├── unit/                       # Unit tests
│   ├── integration/                # Integration tests
│   ├── e2e/                        # End-to-end tests
│   └── conftest.py                 # Test fixtures
├── Dockerfile                      # Production container
├── docker-compose.yml              # Local development setup
├── requirements.txt                # Python dependencies
├── pytest.ini                      # Test configuration
└── README.md                       # This file
```

## Installation

### Prerequisites

- Python 3.10 or higher
- PostgreSQL 13 or higher (or Docker)
- Docker and Docker Compose (recommended)
- Git

### Local Setup

1. Clone the repository:
```bash
git clone <repository-url>
cd <repository-name>
```

2. Create and activate a virtual environment:
```bash
python -m venv venv
source venv/bin/activate  # Windows: venv\Scripts\activate
```

3. Install dependencies:
```bash
pip install -r requirements.txt
```

4. Install Playwright browsers (for E2E tests):
```bash
playwright install
```

5. Set up environment variables (create `.env` file):
```env
DATABASE_URL=postgresql://postgres:postgres@localhost:5432/fastapi_db
JWT_SECRET_KEY=your-super-secret-key-change-in-production
JWT_REFRESH_SECRET_KEY=your-refresh-secret-key
ACCESS_TOKEN_EXPIRE_MINUTES=30
REFRESH_TOKEN_EXPIRE_DAYS=7
BCRYPT_ROUNDS=12
```

## Running the Application

### Using Docker Compose (Recommended)

```bash
docker-compose up
```

This starts:
- FastAPI application on `http://localhost:8000`
- PostgreSQL database on port 5432
- pgAdmin on `http://localhost:5050`

Access credentials for pgAdmin:
- Email: `admin@example.com`
- Password: `admin`

### Local Development

1. Start PostgreSQL (if not using Docker)

2. Initialize the database:
```bash
python -m app.database_init
```

3. Run the application:
```bash
uvicorn app.main:app --reload --host 0.0.0.0 --port 8000
```

4. Access the application:
- API Documentation: `http://localhost:8000/docs`
- Alternative Docs: `http://localhost:8000/redoc`
- Health Check: `http://localhost:8000/health`

## Database Models

### User Model

Located in `app/models/user.py`:

```python
class User:
    id: UUID
    username: String(50, unique, indexed)
    email: String(unique, indexed)
    password: String (hashed)
    first_name: String(50)
    last_name: String(50)
    is_active: Boolean
    is_verified: Boolean
    created_at: DateTime
    updated_at: DateTime
    last_login: DateTime
    calculations: Relationship (one-to-many)
```

**Key Methods:**
- `register()`: Create new user with password hashing
- `authenticate()`: Verify credentials and generate tokens
- `verify_password()`: Compare plain text with hashed password
- `create_access_token()`: Generate JWT access token
- `create_refresh_token()`: Generate JWT refresh token

### Calculation Model (Polymorphic)

Located in `app/models/calculation.py`:

**Base Class:**
```python
class Calculation:
    id: UUID
    user_id: UUID (Foreign Key)
    type: String (discriminator)
    inputs: JSON
    result: Float
    created_at: DateTime
    updated_at: DateTime
```

**Subclasses:**
- `Addition`: Sums all inputs
- `Subtraction`: Sequential subtraction
- `Multiplication`: Product of all inputs
- `Division`: Sequential division with zero-checking

**Factory Method:**
```python
Calculation.create(type, user_id, inputs)
```

## API Endpoints

### Authentication Endpoints

#### Register User
```http
POST /auth/register
Content-Type: application/json

{
  "first_name": "John",
  "last_name": "Doe",
  "email": "john.doe@example.com",
  "username": "johndoe",
  "password": "SecurePass123!",
  "confirm_password": "SecurePass123!"
}
```

**Response (201):**
```json
{
  "id": "uuid",
  "username": "johndoe",
  "email": "john.doe@example.com",
  "first_name": "John",
  "last_name": "Doe",
  "is_active": true,
  "is_verified": false,
  "created_at": "2025-01-01T00:00:00",
  "updated_at": "2025-01-01T00:00:00"
}
```

#### Login
```http
POST /auth/login
Content-Type: application/json

{
  "username": "johndoe",
  "password": "SecurePass123!"
}
```

**Response (200):**
```json
{
  "access_token": "eyJ0eXAiOiJKV1QiLCJhbGc...",
  "refresh_token": "eyJ0eXAiOiJKV1QiLCJhbGc...",
  "token_type": "bearer",
  "expires_at": "2025-01-01T00:30:00",
  "user_id": "uuid",
  "username": "johndoe",
  "email": "john.doe@example.com",
  "first_name": "John",
  "last_name": "Doe",
  "is_active": true,
  "is_verified": false
}
```

### Calculation Endpoints (BREAD)

All calculation endpoints require authentication via Bearer token in the Authorization header.

#### Add (Create) Calculation
```http
POST /calculations
Authorization: Bearer <access_token>
Content-Type: application/json

{
  "type": "addition",
  "inputs": [10.5, 3, 2]
}
```

**Response (201):**
```json
{
  "id": "uuid",
  "user_id": "uuid",
  "type": "addition",
  "inputs": [10.5, 3, 2],
  "result": 15.5,
  "created_at": "2025-01-01T00:00:00",
  "updated_at": "2025-01-01T00:00:00"
}
```

#### Browse (List) Calculations
```http
GET /calculations
Authorization: Bearer <access_token>
```

**Response (200):**
```json
[
  {
    "id": "uuid",
    "user_id": "uuid",
    "type": "addition",
    "inputs": [10.5, 3, 2],
    "result": 15.5,
    "created_at": "2025-01-01T00:00:00",
    "updated_at": "2025-01-01T00:00:00"
  }
]
```

#### Read (Get) Calculation
```http
GET /calculations/{calculation_id}
Authorization: Bearer <access_token>
```

#### Edit (Update) Calculation
```http
PUT /calculations/{calculation_id}
Authorization: Bearer <access_token>
Content-Type: application/json

{
  "inputs": [42, 7]
}
```

#### Delete Calculation
```http
DELETE /calculations/{calculation_id}
Authorization: Bearer <access_token>
```

**Response (204):** No content

## Authentication

### JWT Token System

The application uses JWT (JSON Web Tokens) for stateless authentication:

**Access Token:**
- Short-lived (30 minutes default)
- Used for API authentication
- Required for all calculation endpoints

**Refresh Token:**
- Long-lived (7 days default)
- Used to obtain new access tokens
- Stored securely by client

### Password Security

- **Hashing Algorithm**: bcrypt with configurable rounds (default: 12)
- **Password Requirements**:
  - Minimum 8 characters
  - At least one uppercase letter
  - At least one lowercase letter
  - At least one digit
  - At least one special character

### Using Authentication in Requests

Include the access token in the Authorization header:

```bash
curl -X GET http://localhost:8000/calculations \
  -H "Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGc..."
```

## Testing

### Test Structure

```
tests/
├── unit/                      # Unit tests for operations
├── integration/               # Integration tests for models/schemas
│   ├── test_calculation.py
│   ├── test_calculation_schema.py
│   ├── test_user.py
│   └── test_user_auth.py
└── e2e/                       # End-to-end API tests
    └── test_fastapi_calculator.py
```

### Running Tests Locally

**All Tests:**
```bash
pytest
```

**With Coverage:**
```bash
pytest --cov=app --cov-report=html
```

**Specific Test Categories:**
```bash
pytest tests/unit/                  # Unit tests
pytest tests/integration/           # Integration tests
pytest tests/e2e/                   # E2E tests
```

**View Coverage Report:**
```bash
open htmlcov/index.html  # macOS
# or
start htmlcov/index.html  # Windows
```

### Test Database Setup

Tests use a separate test database configured in `conftest.py`:
- Automatically created before tests
- Cleaned between test runs
- Dropped after test session (unless `--preserve-db` flag used)

### Key Test Files

**Integration Tests:**

- `test_user.py`: User model database operations
  - User creation and querying
  - Uniqueness constraints
  - Transaction handling

- `test_user_auth.py`: Authentication functionality
  - Password hashing and verification
  - User registration
  - Login and token generation
  - Duplicate prevention

- `test_calculation.py`: Calculation model behavior
  - Factory pattern
  - Polymorphic operations
  - Input validation

- `test_calculation_schema.py`: Pydantic validation
  - Schema validation
  - Business rules
  - Error handling

**E2E Tests:**

- `test_fastapi_calculator.py`: Complete API workflows
  - User registration flow
  - Login and token retrieval
  - Calculation BREAD operations
  - Authentication enforcement

### Running Tests in CI/CD

Tests run automatically on every push and pull request via GitHub Actions. The workflow:

1. Sets up PostgreSQL service
2. Installs dependencies
3. Runs all test suites
4. Generates coverage reports
5. Proceeds to security scanning if tests pass

## CI/CD Pipeline

### GitHub Actions Workflow

Located in `.github/workflows/python-app.yml`, the pipeline has three jobs:

#### 1. Test Job

**Services:**
- PostgreSQL with health checks

**Steps:**
1. Checkout code
2. Setup Python 3.10
3. Cache dependencies
4. Install requirements and Playwright
5. Run unit tests with coverage
6. Run integration tests
7. Run E2E tests

#### 2. Security Job

**Steps:**
1. Build Docker image
2. Run Trivy vulnerability scanner
3. Check for CRITICAL and HIGH vulnerabilities
4. Exit on findings (unless ignored in `.trivyignore`)

#### 3. Deploy Job

**Conditions:**
- Only runs on `main` branch
- Requires test and security jobs to pass
- Uses `production` environment

**Steps:**
1. Setup Docker Buildx
2. Login to Docker Hub
3. Build multi-platform image (`linux/amd64`, `linux/arm64`)
4. Push with tags:
   - `latest`
   - Git commit SHA
5. Use registry caching

### Setting Up CI/CD

1. **Configure GitHub Secrets:**

Navigate to repository Settings > Secrets and variables > Actions:

```
DOCKERHUB_USERNAME: your-dockerhub-username
DOCKERHUB_TOKEN: your-dockerhub-access-token
```

2. **Create Production Environment:**

Settings > Environments > New environment: `production`

3. **Update Docker Hub Repository:**

In `.github/workflows/python-app.yml`, change:
```yaml
tags: |
  your-username/your-repo:latest
  your-username/your-repo:${{ github.sha }}
```

### Monitoring Pipeline

- View runs in Actions tab
- Click on individual runs for detailed logs
- Failed jobs show red X with error details
- Successful deployments show green checkmark

## Docker Deployment

### Docker Hub Repository

**Image:** `cerechukwu27/module12:latest`

### Pulling and Running

```bash
# Pull latest image
docker pull cerechukwu27/module12:latest

# Run with environment variables
docker run -p 8000:8000 \
  -e DATABASE_URL=postgresql://user:pass@host:5432/db \
  -e JWT_SECRET_KEY=your-secret-key \
  -e JWT_REFRESH_SECRET_KEY=your-refresh-secret \
  cerechukwu27/module12:latest
```

### Using Docker Compose

```yaml
version: '3.8'
services:
  app:
    image: cerechukwu27/module12:latest
    ports:
      - "8000:8000"
    environment:
      DATABASE_URL: postgresql://postgres:postgres@db:5432/fastapi_db
      JWT_SECRET_KEY: your-secret-key
      JWT_REFRESH_SECRET_KEY: your-refresh-secret
    depends_on:
      - db
  
  db:
    image: postgres:17
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
      POSTGRES_DB: fastapi_db
```

### Health Check

The Docker image includes a health check:

```bash
curl -f http://localhost:8000/health || exit 1
```

## Manual Testing

### Using Swagger UI

1. Start the application
2. Navigate to `http://localhost:8000/docs`
3. You'll see interactive API documentation

**Testing Registration:**
1. Expand `POST /auth/register`
2. Click "Try it out"
3. Fill in the request body
4. Click "Execute"
5. Verify 201 response with user data

**Testing Login:**
1. Expand `POST /auth/login`
2. Click "Try it out"
3. Enter username and password
4. Click "Execute"
5. Copy the `access_token` from response

**Testing Calculations:**
1. Click "Authorize" button at top
2. Enter: `Bearer <your_access_token>`
3. Click "Authorize" then "Close"
4. Now you can test calculation endpoints

**Creating a Calculation:**
1. Expand `POST /calculations`
2. Click "Try it out"
3. Enter calculation data
4. Click "Execute"
5. Verify 201 response with result

**Listing Calculations:**
1. Expand `GET /calculations`
2. Click "Try it out"
3. Click "Execute"
4. Verify 200 response with array

**Updating a Calculation:**
1. Copy an ID from list response
2. Expand `PUT /calculations/{calc_id}`
3. Enter the ID and new inputs
4. Click "Execute"
5. Verify 200 response with updated result

**Deleting a Calculation:**
1. Expand `DELETE /calculations/{calc_id}`
2. Enter the ID
3. Click "Execute"
4. Verify 204 No Content response

### Using cURL

**Register:**
```bash
curl -X POST http://localhost:8000/auth/register \
  -H "Content-Type: application/json" \
  -d '{
    "first_name": "Test",
    "last_name": "User",
    "email": "test@example.com",
    "username": "testuser",
    "password": "SecurePass123!",
    "confirm_password": "SecurePass123!"
  }'
```

**Login:**
```bash
curl -X POST http://localhost:8000/auth/login \
  -H "Content-Type: application/json" \
  -d '{
    "username": "testuser",
    "password": "SecurePass123!"
  }'
```

**Create Calculation (replace TOKEN):**
```bash
curl -X POST http://localhost:8000/calculations \
  -H "Authorization: Bearer TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "type": "addition",
    "inputs": [10, 20, 30]
  }'
```

**List Calculations:**
```bash
curl -X GET http://localhost:8000/calculations \
  -H "Authorization: Bearer TOKEN"
```

### Using Postman

1. Import the OpenAPI spec from `/docs` JSON
2. Set up environment variable for `access_token`
3. Configure Authorization to use `Bearer {{access_token}}`
4. Test each endpoint systematically

## Development Guidelines

### Adding New Features

**New Calculation Type:**
1. Add subclass in `app/models/calculation.py`
2. Implement `get_result()` method
3. Add to factory dictionary
4. Update `CalculationType` enum in schemas
5. Write tests

**New User Fields:**
1. Add column to User model
2. Update Pydantic schemas
3. Update registration/response logic
4. Write tests

### Code Style

- Follow PEP 8
- Use type hints
- Write docstrings
- Keep functions focused
- Use meaningful names

### Security Considerations

- Never commit secrets to repository
- Use environment variables for configuration
- Validate all user inputs
- Use parameterized queries (SQLAlchemy handles this)
- Keep dependencies updated
- Review security scan results

## Troubleshooting

### Common Issues

**Authentication Errors:**
```
401 Unauthorized: Could not validate credentials
```
- Check token hasn't expired
- Verify Bearer token format
- Ensure JWT_SECRET_KEY is consistent

**Database Connection:**
```
Could not connect to database
```
- Verify PostgreSQL is running
- Check DATABASE_URL is correct
- Ensure database exists

**Tests Failing:**
```
Database fixtures not working
```
- Check PostgreSQL service in tests
- Verify test database creation
- Review `conftest.py` setup

**Docker Build Fails:**
```
Cannot install requirements
```
- Check Python version in Dockerfile
- Verify requirements.txt is complete
- Review build logs for details

### Getting Help

1. Check application logs: `docker-compose logs web`
2. Review test output: `pytest -v`
3. Verify database state: Connect via pgAdmin
4. Check GitHub Actions logs for CI/CD issues
