# Suggested Flyway Migration Plan

To bootstrap the database sequentially while handling constraints explicitly, we recommend the following migration steps:

## V1__init_user_and_auth.sql
- Create `USERS` table.
- Create `SELLER_PROFILES` table.
- Add enums or check constraints for `role` and `approval_status`.

## V2__init_shop_domain.sql
- Create `SHOPS` table (with expanded bank details fields).
- Create `SHOP_WB_INTEGRATIONS` table.
- Define foreign keys to `SELLER_PROFILES`.

## V3__init_catalog_domain.sql
- Create `PRODUCTS` table with unique constraint `UNIQUE(shop_id, wb_nm_id)`.
- Create `PRODUCT_VARIANTS` table with foreign key to `PRODUCTS`.
- Create `PRODUCT_VARIANT_SKUS`.
- Create `PRODUCT_IMAGES`.
- Create `PRODUCT_CHARACTERISTICS`.

## V4__init_order_and_payment_domain.sql
- Create `ORDERS` table (with separate `order_status` and `payment_status`).
- Create `ORDER_ITEMS` table ensuring all snapshot fields (slug, name, price, variant labels) are physically present.
- Create `PAYMENT_CONFIRMATIONS` table with foreign key to `ORDERS` and explicit fields (transfer details, receipt URL, reviewer).

## V5__init_sync_logging.sql
- Create `SYNC_JOBS` table.
- Create `SYNC_JOB_LOGS` table for granular step-by-step reporting.
- Add appropriate indexes on `sync_job_id` and `created_at` for fast admin querying.
