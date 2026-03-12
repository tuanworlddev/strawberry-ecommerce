# Phase 1 Execution Package

## 1. Backend First-Cut Scope
We confirm the First-Cut scope as strictly the core foundation:
- **Auth Foundation:** JWT authentication and Spring Security.
- **Roles & RBAC:** Domain roles (ADMIN, SELLER, CUSTOMER).
- **Domain logic:** Seller registration, Admin seller approval, Shop creation.
- **Integrations:** Wildberries integration entity/configuration.
- **Async Foundation:** Initial RabbitMQ sync job creation flow (Queueing / Job Tracking).
- **Core Infrastructure:** Flyway baseline schema, Swagger/OpenAPI setup, Base Exception handling (@ControllerAdvice), Audit logging foundation.

## 2. Phase 1 Implementation Breakdown

### Exact Modules to Implement
- `auth`: Login, Registration, JWT issuing/verification.
- `user`: Seller profiles, Admin approval actions.
- `shop`: Shop creation, updating WB Integration settings.
- `sync`: Submitting Sync Jobs to MQ, retrieving Job status.

### Exact Database Tables Included
1. `users`
2. `seller_profiles`
3. `shops`
4. `shop_wb_integrations`
5. `sync_jobs`
6. `sync_job_logs`

### Exact Flyway Migrations
1. `V1__create_users_and_roles.sql`
2. `V2__create_seller_profiles_and_shops.sql`
3. `V3__create_shop_wb_integrations.sql`
4. `V4__create_sync_jobs_and_audit_logs.sql`

### Exact RabbitMQ Assets
- **Exchange:** `wb.sync.exchange` (Direct)
- **Queues:** `wb_sync_jobs_queue`, `wb_sync_dlq`
- **Routing:** `sync.job.start`

### Exact Frontend Screens (Angular Shell)
- **Shared:** Login, Registration (Seller & Customer switch).
- **Admin Portal:** Sellers List (Approve/Reject actions).
- **Seller Portal:** Create Shop, Shop Settings (WB Key input), Sync History.

---

## 3. Concrete API Contracts (Phase 1)

### 3.1 `POST /api/v1/auth/register/seller`
**Request Body:**
```json
{
  "email": "seller@example.com",
  "password": "SecurePassword123!"
}
```
**Response (201 Created):**
```json
{
  "userId": "uuid",
  "email": "seller@example.com",
  "role": "SELLER",
  "status": "ACTIVE",
  "approvalStatus": "PENDING"
}
```

### 3.2 `POST /api/v1/auth/register/customer`
**Request Body:**
```json
{
  "email": "customer@example.com",
  "password": "SecurePassword123!"
}
```
**Response (201 Created):**
```json
{
  "userId": "uuid",
  "email": "customer@example.com",
  "role": "CUSTOMER"
}
```

### 3.3 `POST /api/v1/auth/login`
**Request Body:**
```json
{
  "email": "seller@example.com",
  "password": "SecurePassword123!"
}
```
**Response (200 OK):**
```json
{
  "accessToken": "eyJhbGci...",
  "refreshToken": "def456...",
  "role": "SELLER"
}
```

### 3.4 `POST /api/v1/admin/sellers/{id}/approve`
**Request Body:**
```json
{
  "action": "APPROVE", 
  "note": "Verified identity"
}
```
**Response (200 OK):**
```json
{
  "sellerId": "uuid",
  "approvalStatus": "APPROVED",
  "reviewedAt": "2026-03-12T18:00:00Z"
}
```

### 3.5 `POST /api/v1/shops` (Requires APPROVED SELLER role)
**Request Body:**
```json
{
  "name": "My Great Shop",
  "slug": "my-great-shop",
  "contactInfo": "support@example.com",
  "bankName": "Sberbank",
  "accountNumber": "40817810...",
  "accountHolderName": "Ivan Ivanov"
}
```
**Response (201 Created):**
```json
{
  "shopId": "uuid",
  "name": "My Great Shop",
  "status": "DRAFT"
}
```

### 3.6 `PUT /api/v1/shops/{shopId}/integration`
**Request Body:**
```json
{
  "wbApiKey": "eyJhb...long_token..."
}
```
**Response (200 OK):**
```json
{
  "integrationId": "uuid",
  "shopId": "uuid",
  "isActive": true,
  "updatedAt": "2026-03-12T18:15:00Z"
}
```

### 3.7 `POST /api/v1/shops/{shopId}/sync/full`
**Request Body:** Empty
**Response (202 Accepted):**
```json
{
  "syncJobId": "uuid",
  "status": "QUEUED",
  "startedAt": "2026-03-12T18:16:00Z"
}
```

### 3.8 `GET /api/v1/shops/{shopId}/sync/{syncJobId}`
**Request Body:** Empty
**Response (200 OK):**
```json
{
  "syncJobId": "uuid",
  "status": "RUNNING",
  "startedAt": "2026-03-12T18:16:00Z",
  "totalFetched": 500,
  "totalCreated": 0,
  "totalUpdated": 0,
  "totalFailed": 0,
  "errorSummary": null
}
```

---

## 4. Project Structure Initialization

We will utilize a monorepo-style structure, housing Backend, Frontend, and Infrastructure together for the MVP scale.

```text
/
├── .antigravity/context/      # Persistent Context state
├── backend/                   # Spring Boot Root
│   └── src/main/java/com/platform/
│       ├── common/            # Exceptions, Security Context, Annotations
│       ├── auth/              # JWT, Login, Registration
│       ├── user/              # Seller Profiles, Admin actions
│       ├── shop/              # Shop Entity, Integrations
│       └── sync/              # Job Queueing, RabbitMQ Listeners
├── frontend/                  # Angular Root
│   ├── apps/
│   │   ├── storefront/        # Customer entry point
│   │   ├── seller-portal/     # Seller entry point
│   │   └── admin-portal/      # Admin entry point
│   └── shared/                # HTTP Interceptors, Guards, UI Lib
├── infra/                     # Docker setup
│   ├── docker-compose.yml     # PostgreSQL + RabbitMQ + pgAdmin
│   └── rabbitmq-entrypoint.sh 
└── docs/                      # General design documents
```

---

## 5. Definition of Done (Phase 1)

Phase 1 is considered strictly "Done" when:

1. **Local Environment:** `docker-compose up` cleanly starts PostgreSQL and RabbitMQ.
2. **Database:** Flyway migrations V1 to V4 run perfectly on boot.
3. **API & Security:** Swagger UI is accessible; APIs are secured via JWT (Authentication and RBAC).
4. **Seller Flow:** A User can register as SELLER, Admin can APPROVE, User can Create a Shop.
5. **WB Integration:** An Approved Seller can securely attach their WB API string.
6. **Sync Foundation:** Seller can hit the `/sync/full` endpoint, a `SYNC_JOB` record is created, and the RabbitMQ queue sees the `syncJobId` arrive safely (no actual catalog data is processed yet).
7. **Frontend Shell:** Angular shell can navigate between the Register/Login/Portal views and execute the above API flows.
