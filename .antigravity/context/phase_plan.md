# MVP Development Phases & Status

## Phase 1: Foundation & Infrastructure ✅ COMPLETE
- Spring Boot backend skeleton, JWT authentication, RBAC (Admin / Seller / Customer).
- Flyway migration system, Swagger/OpenAPI.
- Seller registration, admin approval flow, shop creation.
- Wildberries API key storage (AES-256 encrypted) and sync job publishing via RabbitMQ.

## Phase 2: Wildberries Catalog Sync ✅ COMPLETE
- Full & incremental sync engine with cursor-based pagination.
- Product, Variant, SKU, Image, and Characteristic persistence.
- Idempotent upsert logic. Local field preservation across sync cycles.
- Seller product enrichment APIs (custom title, description, pricing, inventory, visibility).

## Phase 3: Automated Sync Scheduler & Metrics ✅ COMPLETE
- Per-shop scheduling rules (interval 15 min – 24 h, pause/resume).
- PostgreSQL advisory locking to prevent concurrent syncs for the same shop.
- Sync history and metrics APIs (duration, created/updated/failed counts).
- Exponential backoff and retry for transient Wildberries API failures.

## Phase 4: Storefront Discovery & Product Enrichment ✅ COMPLETE
- SEO slug generation (deterministic, stable across re-syncs).
- Public catalog search with multi-faceted filtering (shop, category, brand, price, stock).
- Public product detail API (slug-based), filter suggestion API.
- Seller granular update APIs (metadata, variant pricing, variant inventory).

## Phase 5: Cart, Checkout & Order Management ✅ COMPLETE
- Cart management: add, update, remove, clear with stock validation.
- Checkout: splits cart by shop into separate Orders, reserves stock.
- OrderItem snapshot fields: title, slug, attributes, image, WB NM ID.
- PaymentConfirmation with Cloudinary receipt image upload.
- Seller fulfillment: approve/reject payment, state transitions (NEW → ASSEMBLING → SHIPPING → DELIVERED).
- Independent `order_status` and `payment_status` lifecycles.
- Migrations: V9 (scheduler fields), V10 (carts), V11 (orders), V12 (payment_confirmations), V13 (reserved_stock).

## Phase 6: Polish & UAT (Next)
- End-to-end testing and bug fixing across all phases.
- Load testing for WB sync workers.
- UI/UX refinements and frontend integration.
- Optional: External payment gateway integration (V2 consideration).

---

## Technical Stack
- **Backend:** Spring Boot 3.x, PostgreSQL, Flyway, RabbitMQ, Spring Security (JWT)
- **Storage:** Cloudinary (receipt image uploads)
- **Build:** Maven (mvnw wrapper)
- **API Docs:** Swagger / OpenAPI 3

## Key Technical Decisions
1. **WB field separation:** `wb_title` vs `local_title` — syncs never overwrite seller customizations.
2. **Concurrent sync protection:** PostgreSQL advisory locks per integration ID.
3. **Order splitting:** One `Order` per shop per checkout — simplifies seller fulfillment.
4. **Status independence:** `order_status` and `payment_status` are separate columns (both VARCHAR).
5. **Stock reservation:** `reserved_stock` column in `product_variants`; decremented on DELIVERED or CANCELLED.
6. **Cloudinary for receipts:** Payment proof images uploaded directly to Cloudinary; URL persisted in DB.

## Technical Risks & Mitigations
1. **WB API Rate Limits** → Exponential backoff, dead-letter queues, schema validation on ingest.
2. **Concurrent Syncs** → PostgreSQL advisory locks prevent overlapping jobs per shop.
3. **Data Overwrite** → Strict field separation; application layer always prefers local custom fields.
4. **Manual Payment Bottleneck** → Optimized seller UI for quick approval; external gateway in V2.
