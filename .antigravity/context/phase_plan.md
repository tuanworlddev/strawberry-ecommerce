# MVP Development Phases & Estimates

## Phase 1: Foundation & Infrastructure (Weeks 1-2)
- Server infrastructure setup (Docker, staging DB).
- Core Spring Boot structure, authentication (JWT), and RBAC implementation.
- Basic Angular 21 shell (Routing, Theme, Layouts).
- Database migration system setup (Flyway/Liquibase).
- *Deliverables: Working auth system, empty dashboards.*

## Phase 2: Shop & Catalog Management (Weeks 3-5)
- **Seller Flow:** Registration, Wait for Approval, Shop Creation.
- **Admin Flow:** Seller approval/rejection, Shop management.
- **WB Sync Engine:** RabbitMQ integration, WB API integration, Cursor-based sync logic.
- **Product Management:** Custom title/desc overrides, inventory, pricing management.
- *Deliverables: Sellers can onboard, sync products, and see them in their portal.*

## Phase 3: Storefront & Ordering (Weeks 6-7)
- **Customer:** Browsing, Cart, Checkout flow.
- **Order System:** Order creation, status tracking, stock reservation.
- *Deliverables: Customers can place orders.*

## Phase 4: Payments & Workflow (Weeks 8-9)
- **Payment:** Manual bank transfer instructions, Receipt upload.
- **Seller:** Payment Verification, Order status progression (Assembling, Shipping, Delivered).
- *Deliverables: Complete end-to-end purchasing lifecycle.*

## Phase 5: Polish & UAT (Week 10)
- End-to-end testing, bug fixing.
- Load testing for WB sync workers.
- UI/UX refinements.

---

## Estimated Timeline
**Total Time to MVP:** ~10 Weeks
**Team Size Assumption:** 2 Back-End Engineers, 2 Front-End Engineers, 1 QA, 1 PM.

---

## Technical Risks & Mitigations

1. **Wildberries API Rate Limits & Changes**
   - *Risk:* Sync can fail if rate limits are exceeded or the API schema changes unexpectedly.
   - *Mitigation:* Implement exponential backoff, circuit breakers, and comprehensive dead-letter queues. Add schema validation tests on ingest.

2. **Concurrent Sync Operations**
   - *Risk:* Multiple syncs firing for the same shop corrupts data.
   - *Mitigation:* Implement distributed locking (e.g., Redis via Spring Integration or DB row locking) when a sync job starts.

3. **Data Overlap / Accidental Overwrite**
   - *Risk:* Subsequent syncs overwrite the Seller's custom pricing and descriptions.
   - *Mitigation:* Clear separation of fields. `products` table has `custom_title` and `wb_title` (stored in JSON). The application layer always resolves preferences correctly (coalesce custom fields over raw WB data).

4. **Manual Payment Bottlenecks**
   - *Risk:* Image uploads for receipts and manual verification can be slow and error-prone.
   - *Mitigation:* Build a highly optimized UI for the seller to quickly approve payments. In V2, consider external payment gateways.
