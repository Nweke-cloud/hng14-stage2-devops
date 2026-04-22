# HNG Stage 2 - Containerized Microservices

A job processing system with three services containerized with Docker and deployed via CI/CD.

## Architecture

Internet -> Frontend (Node.js :3000) -> API (FastAPI :8000) -> Redis
                                                 ^
                                           Worker (Python)

## Prerequisites

- Docker >= 24.0
- Docker Compose >= 2.0
- Git

## Run Locally

    git clone https://github.com/Nweke-cloud/hng14-stage2-devops.git
    cd hng14-stage2-devops
    cp .env.example .env
    # Edit .env and set a real REDIS_PASSWORD
    docker compose up -d
    docker compose ps

All four services should show as healthy within 60 seconds.

## Services

| Service  | Port          | Description              |
|----------|---------------|--------------------------|
| Frontend | 3000          | Node.js/Express UI/proxy |
| API      | 8000 internal | FastAPI job management   |
| Worker   | none          | Python job processor     |
| Redis    | 6379 internal | Job queue                |

## Endpoints

| Endpoint    | Method | Response                          |
|-------------|--------|-----------------------------------|
| /           | GET    | Job dashboard HTML                |
| /submit     | POST   | {"job_id": "..."}                 |
| /status/:id | GET    | {"job_id": "...", "status": "..."} |

## CI/CD Pipeline

GitHub Actions runs 6 stages in order:
1. Lint - flake8, eslint, hadolint
2. Test - pytest with coverage report artifact
3. Build - builds and pushes to local registry tagged with git SHA
4. Security - Trivy scan, fails on CRITICAL findings
5. Integration - full stack test, job submitted and verified completed
6. Deploy - rolling update on push to main only

## Tear Down

    docker compose down -v
