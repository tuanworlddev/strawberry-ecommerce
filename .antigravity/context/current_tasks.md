# Refine Planning

- [x] Update Database ERD
  - [x] Add `product_variants`, `product_images`, `product_characteristics`, `product_variant_skus`
  - [x] Update uniqueness constraint `UNIQUE(shop_id, wb_nm_id)`
  - [x] Create `shop_wb_integrations` table
  - [x] Update cursor types to timestamp and bigint
  - [x] Split `order_status` and `payment_status`
  - [x] Add snapshot fields to `order_items`
  - [x] Make payment schema fields explicit
  - [x] Normalize bank details on `shops`
  - [x] Add `sync_jobs` and `sync_job_logs`
  - [x] Separate `users.status` and `seller_profiles.approval_status`
- [x] Update API Specification
  - [x] Sync endpoints
  - [x] Payment endpoints consistency
  - [x] Granular product update endpoints
- [x] Update RabbitMQ Event Architecture
  - [x] Remove API key from payload, store `integrationId`
- [x] Add Flyway Migration Plan
- [x] Update `implementation_plan.md`
- [x] Sync `.antigravity/context/` files

# Phase 1 Execution

- [x] Initialize the Spring Boot backend skeleton
- [ ] Setup Docker local environment (PostgreSQL, RabbitMQ) - *Blocked: Docker CLI not found*
- [x] Configure Flyway migrations
- [x] Setup Swagger/OpenAPI
- [x] Implement JWT authentication
- [x] Implement seller registration flow
- [x] Implement admin seller approval flow
- [x] Implement shop creation APIs
- [x] Implement Wildberries API key storage (encrypted)
- [x] Implement sync job creation + RabbitMQ publishing
