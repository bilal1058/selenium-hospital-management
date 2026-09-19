# Hospital Management System

A full-stack hospital management system with a web frontend, FastAPI backend, PostgreSQL database, automated Selenium testing, Docker-based deployment, and Jenkins CI/CD integration.

## Overview

The application provides a web-based interface backed by a Python/FastAPI service and PostgreSQL database.

The project also includes an automated testing and deployment workflow:

```text
Frontend
   ↓
FastAPI Backend
   ↓
PostgreSQL
   ↓
Docker
   ↓
Selenium / PyTest
   ↓
Jenkins CI/CD
```

## Tech Stack

### Frontend

* React
* Node.js
* npm

### Backend

* Python
* FastAPI
* Uvicorn
* PostgreSQL

### Testing

* Selenium WebDriver
* PyTest
* pytest-html
* JUnit XML reports

### Infrastructure

* Docker
* Docker Compose
* Jenkins
* PostgreSQL

## Project Structure

```text
.
├── backend/
├── frontend/
├── eval/
├── selenium-tests/
├── Dockerfile
├── docker-compose.yml
├── Jenkinsfile
├── .gitignore
└── README.md
```

### `backend/`

Contains the FastAPI backend and its Python dependencies.

### `frontend/`

Contains the React frontend. The production build is generated during the Docker image build and copied into the backend's static directory.

### `selenium-tests/`

Contains the Selenium/PyTest test suite used for end-to-end browser testing.

### `eval/`

Contains evaluation-related project files.

## Application Architecture

The production Docker image uses a multi-stage build.

First, the frontend is built with Node.js:

```text
Node.js
   ↓
npm install
   ↓
npm run build
```

The resulting frontend build is then included in the Python application image:

```text
React Build
     ↓
FastAPI Application
     ↓
Uvicorn
     ↓
PostgreSQL
```

The backend container is based on Python 3.11 and includes the PostgreSQL client and application dependencies.

## Docker

The application is configured to run with Docker Compose.

The Compose configuration defines two services:

```text
┌──────────────────────────┐
│      hospital-web        │
│                          │
│   FastAPI + React Build  │
│        Port 8200         │
└────────────┬─────────────┘
             │
             ▼
┌──────────────────────────┐
│       hospital-db        │
│                          │
│      PostgreSQL 15       │
│       hospital_db        │
└──────────────────────────┘
```

PostgreSQL uses a persistent Docker volume, and the web service waits for the database health check before starting.

### Start the application

```bash
docker compose up -d
```

### Check containers

```bash
docker compose ps
```

### Stop the application

```bash
docker compose down
```

To remove the database volume as well:

```bash
docker compose down -v
```

The application is exposed through port `8202` on the host and mapped to port `8200` in the Compose configuration.

## Configuration

The database configuration can be supplied through environment variables:

```env
POSTGRES_USER=postgres
POSTGRES_PASSWORD=yourpassword
POSTGRES_DB=hospital_db
SECRET_KEY=your-secret-key
```

The application uses these values to construct its PostgreSQL connection string.

For deployments outside a local environment, credentials and secret keys should be provided through a secure environment or secret-management system rather than committed to source control.

## Automated Testing

The `selenium-tests` directory contains browser-based end-to-end tests executed with Selenium and PyTest.

Tests are executed against the running application using:

```bash
pytest test_all.py -v
```

The test pipeline generates both HTML and JUnit-compatible reports:

```bash
pytest test_all.py -v \
  --html=report.html \
  --self-contained-html \
  --junitxml=results.xml \
  --tb=short
```

### Test Reports

Two report formats are generated:

**HTML**

```text
selenium-tests/report.html
```

Provides a browser-readable test report.

**JUnit XML**

```text
selenium-tests/results.xml
```

Used by Jenkins to publish automated test results.

## Jenkins CI/CD

The repository includes a `Jenkinsfile` that automates the application deployment and testing workflow.

The pipeline follows these stages:

```text
Checkout
   ↓
Deploy Application
   ↓
Run Selenium Tests
   ↓
Publish Test Results
   ↓
Archive Report
   ↓
Email Notification
```

### Checkout

Jenkins checks out the repository using the configured SCM.

### Deploy Application

The existing Docker Compose deployment is stopped and the application is started again:

```bash
docker-compose down
docker-compose up -d
```

### Run Selenium Tests

The Selenium test suite runs inside the configured Selenium test container:

```bash
pytest test_all.py -v \
  --html=report.html \
  --self-contained-html \
  --junitxml=results.xml \
  --tb=short
```

### Publish Results

Jenkins reads:

```text
selenium-tests/results.xml
```

and publishes the JUnit test results.

### Archive Reports

The generated HTML report is archived as a Jenkins build artifact.

### Notifications

The pipeline extracts the test totals, failures, skipped tests, and passed tests from the generated JUnit results and includes the summary in the build notification.

## Deployment Architecture

```text
                         Git Repository
                              │
                              ▼
                         ┌─────────┐
                         │ Jenkins │
                         └────┬────┘
                              │
                    ┌─────────┴─────────┐
                    │                   │
                    ▼                   ▼
              Docker Compose      Selenium Container
                    │                   │
          ┌─────────┴─────────┐         │
          │                   │         │
          ▼                   ▼         ▼
     hospital-web        hospital-db   PyTest
          │                   │         │
          │                   │         ▼
          │                   │    HTML / JUnit
          └─────────┬─────────┘         │
                    │                   │
                    └───────────────────┘
```

## Docker Image

The main `Dockerfile` uses a multi-stage build:

1. Build the React frontend using Node.js 18.
2. Create a Python 3.11 runtime image.
3. Install backend dependencies.
4. Copy the backend application.
5. Copy the compiled frontend into the backend static directory.
6. Run the application with Uvicorn.

## Development

Clone the repository:

```bash
git clone https://github.com/bilal1058/Selenium-assignment.git
cd Selenium-assignment
```

Start the services:

```bash
docker compose up -d
```

View running services:

```bash
docker compose ps
```

Run the Selenium test suite:

```bash
cd selenium-tests
pytest test_all.py -v
```

## Testing Workflow

```text
Application Start
       ↓
Browser Launch
       ↓
Selenium Test Execution
       ↓
Assertions
       ↓
JUnit Results
       ↓
HTML Report
       ↓
Jenkins Publication
```

## Security

For production deployments:

* Use strong database credentials.
* Store secrets outside the repository.
* Use environment variables or a secret-management service.
* Avoid exposing database services publicly.
* Use HTTPS when exposing the application externally.
* Rotate credentials and secret keys when necessary.

## License

See the repository for licensing information.
