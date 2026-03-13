# Product Domain Contracts (Phase 2)

This document outlines the planned persistence model and backend representation of products, ensuring complete variant-level support for Wildberries synchronization and local catalog independence.

## 1. Domain Entities

### Product (`products`)
Represents the root item. Grouping container for characteristics and variants.
- `id` (UUID, PK)
- `shop_id` (UUID, FK to shops)
- `wb_nm_id` (BigInt, unique per shop)
- `title` (String)
- `brand` (String)
- `description` (Text)
- `category_name` (String)
- `wb_vendor_code` (String, article)
- `created_at`, `updated_at`

### Product Characteristic (`product_characteristics`)
Arbitrary key-value pairs at the root product level.
- `id` (UUID, PK)
- `product_id` (UUID, FK to products)
- `name` (String, e.g., "Color", "Material", "Season")
- `value` (String)

### Product Image (`product_images`)
Media assets related to the product.
- `id` (UUID, PK)
- `product_id` (UUID, FK)
- `wb_url` (String, original WB hosted URL)
- `local_url` (String, nullable - if we plan to host copies)
- `is_main` (Boolean)
- `sort_order` (Integer)

### Product Variant (`product_variants`)
Represents a specific sellable size or configuration (what WB calls a "size" or `chrtID`).
- `id` (UUID, PK)
- `product_id` (UUID, FK)
- `chrt_id` (BigInt, WB specific characteristic ID for the size)
- `tech_size` (String, e.g., "42", "M")
- `wb_size` (String, e.g., "M")
- `base_price` (Decimal)
- `discount_price` (Decimal)
- `stock_quantity` (Integer)
- `is_active` (Boolean)

### Product Variant SKU (`product_variant_skus`)
SKUs associated with a specific variant. A single size on WB can have multiple barcodes (SKUs).
- `id` (UUID, PK)
- `variant_id` (UUID, FK)
- `sku` (String, barcode)

## 2. DTO Contracts (Seller Portal)

When sellers manage their catalog or view synced items:

```json
{
  "id": "uuid",
  "wbNmId": 1234567,
  "title": "Cotton T-Shirt",
  "categoryName": "T-Shirts",
  "vendorCode": "TSH-001",
  "images": [
    { "url": "https://basket-01.wb.ru/...", "isMain": true }
  ],
  "characteristics": [
    { "name": "Material", "value": "100% Cotton" }
  ],
  "variants": [
    {
      "id": "uuid",
      "chrtId": 9876543,
      "techSize": "M",
      "wbSize": "M",
      "skus": ["2000000123456"],
      "basePrice": 1500.00,
      "discountPrice": 1200.00,
      "stockQuantity": 45,
      "isActive": true
    }
  ]
}
```

## 3. Scope of Internal Independence
Once synchronized from Wildberries:
- **Pricing & Stock:** Sellers will update `base_price`, `discount_price`, and `stock_quantity` locally. These are **not** pushed back to WB.
- **Content:** Sellers can optionally modify `title` and `description` for the local storefront.
- **Orders:** When a user buys a `product_variant` locally, the stock is reduced locally.
