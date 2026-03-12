# RabbitMQ Event Architecture

Reliable asynchronous processing is required for the Wildberries product catalog sync, transactional emails, and state-change events.

## Exchanges & Queues

### 1. Catalog Sync Exchange (`wb.sync.exchange`)
- **Type:** Direct
- **Routing Key:** `sync.job.start`
- **Queue:** `wb_sync_jobs_queue`
- **Message Payload:**
  - `syncJobId` (UUID)
  - `shopId` (UUID)
  - `integrationId` (UUID)
  - `syncType` ("FULL" | "INCREMENTAL")
  - `cursor` (JSON object, nullable)
  - `attemptNumber` (Integer)
- **Security Rule:** API Keys are *never* transmitted in the payload. The worker securely retrieves the API key directly from the Database (`SHOP_WB_INTEGRATIONS` table) using the `integrationId`.
- **Worker Behavior:**
  - Consumes message.
  - Fetches Integration Settings from DB.
  - Updates `SYNC_JOBS` history.
  - Fetches data iteratively from Wildberries APIs.
  - Upserts into `PRODUCTS` and `PRODUCT_VARIANTS` tables whilst ensuring local overrides (prices/stock/content) are preserved.
  - Appends logs sequentially to `SYNC_JOB_LOGS` table.

- **Dead Letter Exchange (DLX):** `wb.sync.dlx`
  - Routes failed jobs to retry mechanisms and eventually a dead-letter queue.

### 2. Notification Exchange (`notification.exchange`)
- **Type:** Topic
- **Routing Keys & Queues:**
  - `email.order.created` -> `email_delivery_queue`
  - `email.payment.status_changed` (e.g., CONFIRMED/REJECTED) -> `email_delivery_queue`
  - `email.seller.approved` -> `email_delivery_queue`

### 3. Order Events Exchange (`order.events.exchange`)
- **Type:** Topic
- **Description:** Broadcasts order or payment state changes so various modules can handle their logic.
- **Routing Keys:** 
  - `order.status.*`
  - `payment.status.*` 
- **Subscribers:**
  - **Auto-Cancel Worker:** Listens to `order.created`, sets a TTL in Redis or Quartz. If TTL expires and Order still has Payment Status `PENDING` or `WAITING_CUSTOMER_SUBMISSION`, it cancels the order.
  - **Inventory Service:** Listens to `order.status.CANCELLED` to release previously reserved variant stock. Listens to `payment.status.CONFIRMED` to complete variant deduction.
