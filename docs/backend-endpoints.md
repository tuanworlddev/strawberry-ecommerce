# Backend API Specification

This document lists all active backend endpoints for the Strawberry E-commerce platform.

## Base URL
`http://localhost:8080/api/v1`

---

## 1. Authentication (`/auth`)

### 1.1 Login
- **Endpoint**: `POST /login`
- **Method**: `POST`
- **Purpose**: Authenticates a user and returns a JWT token.
- **Auth Required**: No
- **Request Body**:
  ```json
  {
    "email": "seller@example.com",
    "password": "password123"
  }
  ```
- **Response**: `LoginResponse` (accessToken, refreshToken, email, fullName, role)

### 1.2 Seller Registration
- **Endpoint**: `POST /register/seller`
- **Method**: `POST`
- **Purpose**: Registers a new seller account.
- **Auth Required**: No
- **Request Body**: `RegisterRequest` (fullName, email, password)

---

## 2. Storefront Catalog (`/public/catalog`)

### 2.1 Product Search & Listing
- **Endpoint**: `GET /products` or `GET /search`
- **Method**: `GET`
- **Purpose**: Paginated product listing with multi-faceted filtering.
- **Auth Required**: No
- **Query Params**:
  - `search`: String (title, brand, vendor code search)
  - `shopSlug`: String
  - `categoryId`: String (Long ID or name fallback)
  - `minPrice`: BigDecimal
  - `maxPrice`: BigDecimal
  - `inStock`: Boolean
  - `page`, `size`, `sort`
- **Response**: `Page<ProductResponseDto>`

### 2.2 Product Detail by Slug
- **Endpoint**: `GET /products/{slug}`
- **Method**: `GET`
- **Purpose**: Detailed product info for SEO pages.
- **Auth Required**: No
- **Response**: `ProductDetailResponseDto`

### 2.3 Filter Suggestions
- **Endpoint**: `GET /filters`
- **Method**: `GET`
- **Purpose**: Returns available categories and brands for the sidebar.
- **Auth Required**: No
- **Response**: `CatalogFiltersDto` (categories, brands)

---

## 3. Categories (`/public/catalog/categories`)

### 3.1 All Categories
- **Endpoint**: `GET /`
- **Method**: `GET`
- **Purpose**: Returns all categories with active product counts.
- **Auth Required**: No
- **Response**: `List<CategoryResponseDto>`

---

## 4. Public Shops (`/public/shops`)

### 4.1 Shop Detail
- **Endpoint**: `GET /{shopSlug}`
- **Method**: `GET`
- **Purpose**: Public shop profile.
- **Auth Required**: No
- **Response**: `ShopResponseDto`

---

## 5. Cart Management (`/customer/cart`)

### 5.1 Get Cart
- **Endpoint**: `GET /`
- **Method**: `GET`
- **Purpose**: Get current user's active cart.
- **Auth Required**: Yes (`CUSTOMER`, `SELLER`, `ADMIN`)
- **Response**: `CartResponseDto`

### 5.2 Add/Update Items
- **Endpoint**: `POST /items` or `PUT /items/{itemId}`
- **Method**: `POST`/`PUT`
- **Auth Required**: Yes
- **Request Body**: `CartItemRequestDto` (variantId, quantity)

---

## 6. Checkout & Orders (`/customer/orders`)

### 6.1 Checkout
- **Endpoint**: `POST /checkout`
- **Method**: `POST`
- **Purpose**: Convert cart into orders (split by shop).
- **Auth Required**: Yes (`CUSTOMER`)
- **Request Body**: `CheckoutRequestDto` (address, comment, etc.)

---

## 7. Seller Management (`/seller/shops`)

### 7.1 My Shops
- **Endpoint**: `GET /`
- **Method**: `GET`
- **Purpose**: List shops owned by the seller.
- **Auth Required**: Yes (`SELLER`)
- **Response**: `List<ShopResponseDto>`

---

## 8. Seller Inventory (`/seller/shops/{shopId}/variants`)

### 8.1 Inventory Listing
- **Endpoint**: `GET /inventory`
- **Method**: `GET`
- **Purpose**: Flat list of variants for stock management.
- **Auth Required**: Yes (`SELLER`)
- **Response**: `Page<VariantInventoryResponseDto>`

### 8.2 Bulk Inventory Update
- **Endpoint**: `POST /bulk-inventory`
- **Method**: `POST`
- **Purpose**: Atomic update of stock quantities.
- **Auth Required**: Yes (`SELLER`)
- **Request Body**: `VariantInventoryBulkUpdateRequestDto`
