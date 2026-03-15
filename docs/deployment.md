# Deployment Guide

## Build Audit Summary

### Backend

- Build tool: Maven Wrapper
- Main command: `./mvnw -DskipTests package`
- Runtime: Spring Boot 4 jar
- Dependencies: PostgreSQL, RabbitMQ, Flyway migrations, Cloudinary for receipt uploads

### Frontend

- Build tool: npm + Angular CLI
- Main command: `npm run build`
- Runtime: Angular SSR Node server
- Deployment contract: browser talks to frontend, frontend proxies `/api` to backend

## Local Demo Workflow

1. Copy the environment file:

```bash
cp .env.example .env
```

2. Set:

- `JWT_SECRET`
- `CLOUDINARY_URL` if uploads are required

3. Start the stack:

```bash
docker compose --env-file .env -f infra/docker/docker-compose.yml up --build
```

4. Optional tooling:

```bash
docker compose --env-file .env -f infra/docker/docker-compose.yml --profile tools up --build
```

## Image Strategy

Recommended image names:

- `ghcr.io/<owner>/strawberry-backend`
- `ghcr.io/<owner>/strawberry-frontend`

Recommended tagging:

- release tags: `v1.0.0`, `v1.1.0`, etc.
- `latest` for the default branch or the latest approved release

## CI

The CI workflow validates:

- backend package build
- frontend production build
- Docker Compose syntax
- backend Docker image build
- frontend Docker image build

## CD / Release Path

The release workflow in `.github/workflows/docker-release.yml` is designed to:

1. trigger on tags like `v1.0.0`
2. build backend and frontend images
3. push both images to GHCR

## Production-Oriented Environment Variables

### Backend

- `SPRING_PROFILES_ACTIVE`
- `SPRING_DATASOURCE_URL`
- `SPRING_DATASOURCE_USERNAME`
- `SPRING_DATASOURCE_PASSWORD`
- `SPRING_RABBITMQ_HOST`
- `SPRING_RABBITMQ_PORT`
- `SPRING_RABBITMQ_USERNAME`
- `SPRING_RABBITMQ_PASSWORD`
- `JWT_SECRET`
- `JWT_EXPIRATION`
- `CLOUDINARY_URL`

### Frontend

- `PORT`
- `BACKEND_URL`

## Example Deployment Story

For a simple VPS deployment:

1. provision Docker + Docker Compose
2. copy `.env`
3. pull release-tagged images from GHCR
4. run the same compose file with production image names
5. place a domain/reverse proxy in front if desired

This keeps the deployment story simple and easy to explain in a portfolio review.
