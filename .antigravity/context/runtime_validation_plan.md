# Phase 1 Runtime Validation Plan

This document outlines the steps to start the infrastructure, run the application, and verify the core Phase 1 backend features.

## 1. Local Infrastructure Options

Since Docker CLI is currently unavailable, please choose one of the following approaches to provide PostgreSQL and RabbitMQ:

### Option A: Install Docker Desktop / Docker Engine (Recommended)
1. Install Docker Desktop for Windows.
2. Ensure the Docker daemon is running.
3. Open a terminal in `infra/docker` and run:
   ```bash
   docker compose up -d
   ```
   *(This will start PostgreSQL on port 5432 and RabbitMQ on 5672 + 15672 for the management UI).*

### Option B: Native Installation (Alternative)
1. **PostgreSQL 18:** Install natively and create a database named `strawberry_db` with username `postgres` and password `postgres`.
2. **RabbitMQ:** Install Erlang, then RabbitMQ Server for Windows. Ensure the server is started and running on port 5672.

---

## 2. Environment Variables & App Startup

With your infrastructure running, verify these environment variables are correct in `backend/src/main/resources/application.yaml`, or override them via your IDE:

- **Database URL:** `jdbc:postgresql://localhost:5432/strawberry_db`
- **Database User/Pass:** `postgres` / `postgres`
- **RabbitMQ Host:** `localhost`
- **RabbitMQ Port:** `5672`
- **RabbitMQ User/Pass:** `guest` / `guest`

**Starting the Application:**
Run the Spring Boot application using Maven:
```bash
cd backend
./mvnw spring-boot:run
```
Upon startup, **Flyway** will automatically run `V1` to `V4` migrations, creating the schema.

---

## 3. Recommended Order of Endpoint Testing

Once the application is running (default port `8080`), you can access Swagger UI at:
**[http://localhost:8080/swagger-ui/index.html](http://localhost:8080/swagger-ui/index.html)**

Follow this specific order to validate the system flows:

### Step 1: Registration
- **Customer Registration:** `POST /api/v1/auth/register-customer`
  *(Validates simple user creation).*
- **Seller Registration:** `POST /api/v1/auth/register-seller`
  *(Validates user + seller profile creation in PENDING state).*

### Step 2: Login
- **Login:** `POST /api/v1/auth/login` (Use the seller credentials from Step 1).
  *(Validates JWT generation. Copy the resulting token).*

### Step 3: Admin Flow
- **Login as Admin:** First, use the `db/migration/V1...` seeded admin credentials or seed a user manually with `ROLE_ADMIN`. Receive admin JWT.
- **Admin Seller Approval:** `PUT /api/v1/admin/sellers/{userId}/approve`
  *(Validates RBAC and sets the seller to APPROVED. Verify the `audit_logs` table reflects this action).*

### Step 4: Shop Management
- **Create Shop:** `POST /api/v1/shops` (Use the **Seller's JWT**).
  *(Validates the seller can create a shop now that they are approved. Take note of the `shopId`).*

### Step 5: Wildberries Integration
- **Set API Key:** `PUT /api/v1/shops/{shopId}/integration` (Use the **Seller's JWT**).
  *(Validates API key AES encryption and saving to `shop_wb_integrations`).*

### Step 6: RabbitMQ Sync Publishing
- **Trigger Sync Job:** `POST /api/v1/shops/{shopId}/sync` (Use the **Seller's JWT**).
  *Payload:* `{"syncType": "FULL_CATALOG"}`
  *(Validates that a sync job is persisted into `sync_jobs` as QUEUED, and a message is successfully published to `wb.sync.exchange`).*

### Step 7: Verification
- Inspect the **RabbitMQ Management UI** (`http://localhost:15672`, guest/guest) to see the message in the `wb_sync_jobs_queue`.
- Check the **PostgreSQL `audit_logs` table** to ensure `SYNC_JOB_TRIGGERED`, `SELLER_PROFILE_APPROVED`, and other major operations were logged successfully.
