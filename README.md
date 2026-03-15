# Strawberry E-Commerce

Strawberry is a full-stack multi-shop e-commerce platform with:

- Spring Boot 4 backend
- Angular 21 SSR storefront and seller UI
- PostgreSQL
- RabbitMQ
- Flyway database migrations

## Monorepo Layout

- [`backend`](./backend): Spring Boot API, business workflows, RabbitMQ consumers, Flyway migrations
- [`frontend`](./frontend): Angular SSR storefront and seller/admin UI
- [`infra`](./infra): Docker Compose setup for local orchestration
- [`docs`](./docs): deployment and architecture notes

## Clone the Project

This repository uses Git submodules for `backend/` and `frontend/`, so clone it with recursive submodule checkout:

```bash
git clone --recurse-submodules https://github.com/your-org/strawberry-ecommerce.git
cd strawberry-ecommerce
```

If you already cloned the repository without submodules, run:

```bash
git submodule update --init --recursive
```

## Quick Start With Docker

1. Copy the environment template:

```bash
cp .env.example .env
```

2. Fill in at least:

- `JWT_SECRET`
- `CLOUDINARY_URL` if you want payment receipt uploads to work

3. Start the full stack:

```bash
docker compose --env-file .env -f infra/docker/docker-compose.yml up --build
```

## Default Local URLs

- Frontend: `http://localhost:4200`
- Backend API: `http://localhost:8080`
- Swagger UI: `http://localhost:8080/swagger-ui.html`
- RabbitMQ Management: `http://localhost:15672`
- PgAdmin: `http://localhost:5050` with `--profile tools`

## CI / CD

- CI workflow: [`.github/workflows/ci.yml`](./.github/workflows/ci.yml)
- Docker release workflow: [`.github/workflows/docker-release.yml`](./.github/workflows/docker-release.yml)

## Documentation

- Docker architecture: [`docs/docker-architecture.md`](./docs/docker-architecture.md)
- Deployment guide: [`docs/deployment.md`](./docs/deployment.md)
