# Docker Architecture

## Overview

The Docker setup is designed for two goals:

1. Fast local demo for recruiters and reviewers
2. A clean deployment-ready foundation without unnecessary platform complexity

## Services

### `frontend`

- Built from [`frontend/Dockerfile`](/e:/Sources/strawberry-ecommerce/frontend/Dockerfile)
- Angular SSR app served by Node.js
- Exposes port `4000` inside the container
- Mapped to `localhost:4200` in Docker Compose
- Proxies `/api` requests to the backend through `BACKEND_URL`

### `backend`

- Built from [`backend/Dockerfile`](/e:/Sources/strawberry-ecommerce/backend/Dockerfile)
- Spring Boot 4 application packaged as a runnable jar
- Exposes port `8080`
- Uses environment variables for database, RabbitMQ, JWT, and Cloudinary config
- Exposes health checks through Spring Boot Actuator

### `postgres`

- PostgreSQL 18
- Persistent volume for data
- Used by Spring Data JPA + Flyway

### `rabbitmq`

- RabbitMQ with management UI
- Used by the sync workflow

### `pgadmin` (optional)

- Enabled through the `tools` profile
- Useful for demos and debugging, but kept optional

## Networking Story

- The browser only needs to call the frontend on `localhost:4200`
- The frontend SSR server proxies `/api` to the backend
- This removes the need for frontend builds that hardcode backend hostnames

## Health Checks

- Backend: `GET /actuator/health`
- Frontend: `GET /health`
- PostgreSQL: `pg_isready`
- RabbitMQ: `rabbitmq-diagnostics -q ping`

## Why SSR Was Kept

The frontend already builds as an Angular SSR application and includes a Node server entrypoint.
Keeping SSR avoids a second deployment architecture just for Docker and remains stable for demos:

- one frontend image
- one backend image
- same routing behavior as the current app
- same-origin API proxy support
