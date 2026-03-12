# Wildberries E-Commerce Platform - Project Summary

## Goal
Build an independent e-commerce platform for the Russian market that integrates product catalog data from Wildberries but manages pricing, inventory, orders, and payments internally.

## Key Features
- **Multi-vendor Platform:** Multiple sellers (shops) operate inside the system.
- **Roles:** Super Admin (system management), Seller/Shop Owner (manage shop/orders), Customer (browse and purchase).
- **Wildberries Integration:** WB is used *only* as a catalog data source (via `POST /content/v2/get/cards/list`). The system maintains custom titles, descriptions, pricing, and inventory.
- **Payment Flow:** MVP uses manual bank transfers with receipt uploads for verification.

## Technology Stack
- **Frontend:** Angular v21
- **Backend:** Spring Boot 4
- **Database:** PostgreSQL 18
- **Message Queue:** RabbitMQ (for catalog sync, retries, DLQ, emails)
- **Deployment:** Docker

## Context Structure (.antigravity/context)
- `project_summary.md` - Overarching platform goals and stack.
- `architecture.md` - High-level system architecture and component interactions.
- `erd.md` - Database entity-relationship diagram.
- `api_contracts.md` - Required OpenAPI endpoints.
- `rabbitmq_architecture.md` - Event broker queues and logic.
- `phase_plan.md` - Development phases and timeline.
- `current_tasks.md` - Active checklist of items.

*Note: This folder is automatically ingested by Antigravity upon session start to rebuild context.*
