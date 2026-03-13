# Catalog Sync Persistence Plan (Phase 2)

This document details the exact process and mapping strategy for ingesting catalog data from the Wildberries API into our local `product` and `variant` tables.

## 1. Trigger & Fetch Mechanism

When a `SYNC_JOB` message is picked up by a RabbitMQ consumer:
1. **Fetch Integration Details:** Load the `shop_wb_integrations` record using the `integrationId` from the payload.
2. **Setup HTTP Client:** Decrypt the WB API Key securely from the database.
3. **Pagination Loop:** 
   - Call WB API Endpoint: `POST /content/v2/get/cards/list` (or the v3 equivalent if v2 is deprecated).
   - Use `cursor.nmID` and `cursor.updatedAt` to fetch the next batch.
   - Batch size: Max 100 items per request to avoid timeouts.
4. **Update DB Cursor:** After each successful batch, save the new `cursor` to `shop_wb_integrations` so future partial syncs start from where this left off.

## 2. Mapping Strategy (API Response -> Local Entities)

In the Wildberries JSON response for a card (Product):

- `nmID` -> `Product.wb_nm_id`
- `vendorCode` -> `Product.wb_vendor_code`
- `title` -> `Product.title`
- `description` -> `Product.description`
- `brand` -> `Product.brand`
- `subjectName` -> `Product.category_name`

### Media Mapping
Iterate over `photos`:
- `photos[i].url` -> `ProductImage.wb_url`
- Set `is_main = true` for index 0.

### Characteristics Mapping
Iterate over `characteristics`:
- Key -> `ProductCharacteristic.name`
- Value -> `ProductCharacteristic.value` (Extract safely if array).

### Variant Mapping (The `sizes` array)
WB groups variants under `sizes` on the card. Iterate over `sizes`:
- `sizes[i].chrtID` -> `ProductVariant.chrt_id`
- `sizes[i].techSize` -> `ProductVariant.tech_size`
- `sizes[i].wbSize` -> `ProductVariant.wb_size`

**Initial Sync Stock/Pricing:**
*Note:* The `/cards/list` endpoint usually doesn't return live stock/pricing. For MVP Phase 1/2 sync, we will default `basePrice` to 0 and `stock` to 0. A separate subsequent pricing/stock sync job or manual seller input will populate these fields locally.

### SKUs Mapping
Inside each `sizes[i]`:
Iterate over `sizes[i].skus`:
- `sku` -> `ProductVariantSku.sku`

## 3. Conflict Resolution (Upsert Strategy)

To ensure idempotency during catalog syncs:
1. Lookup existing Product by `shop_id` + `wb_nm_id`.
2. **If Not Exists:** Insert new `Product`, new `Images`, new `Characteristics`, new `Variants`, and `SKUs`.
3. **If Exists:**
   - Update core product details (Title, Description).
   - Delete all old `Characteristics` & `Images` and re-insert new ones (safest way to handle array changes).
   - For `Variants`:
     - Match by `chrt_id`. 
     - If variant exists in DB, update sizes. DO NOT overwrite local `price` or `stock`.
     - If variant is new from API, insert it.
     - If variant exists in DB but not in API response, set `is_active = false` (soft delete to preserve order history).
     - Sync SKUs for the variant similarly.

## 4. Transaction Boundaries
Each batch of 10-50 products should be wrapped in a `@Transactional` block. If a batch fails, it rolls back, but previous batches are safe. The sync job log is updated with warnings or errors for that batch.
