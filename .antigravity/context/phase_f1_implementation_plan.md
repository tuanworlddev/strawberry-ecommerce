# Frontend Phase F1 – Customer Storefront Application

## Summary

Build the customer-facing Storefront SPA on top of the existing Angular 21 monorepo workspace (`e:/Sources/strawberry-ecommerce/frontend`). The storefront lives in the **root project** (`src/`) which is already configured with SSR and Tailwind CSS v4. No new Angular project needs to be created — only feature modules, services, and components.

The backend at `http://localhost:8080` is the live API. No mocking needed.

---

## Technical Stack Confirmed

| Concern | Choice |
|---|---|
| Framework | Angular 21 (standalone components) |
| Styling | Tailwind CSS v4 (already configured via `@tailwindcss/postcss`) |
| State | Angular Signals + `HttpClient` + RxJS |
| Auth | JWT tokens stored in `localStorage`, injected via `HttpInterceptor` |
| Routing | Angular Router with lazy-loaded routes |
| Forms | Angular Reactive Forms |
| SSR | `@angular/ssr` enabled — use `isPlatformBrowser` guards for localStorage |
| API Base URL | `http://localhost:8080` via an `environment.ts` constant |

---

## Proposed Changes

### Project Structure

The `src/app/` folder will be organized as:

```
src/
  app/
    core/
      auth/
        auth.service.ts          # login, register, token helpers
        auth.guard.ts            # protects customer routes
        auth.interceptor.ts      # injects JWT header
      services/
        catalog.service.ts
        cart.service.ts
        order.service.ts
        shipping.service.ts
      models/
        product.model.ts
        cart.model.ts
        order.model.ts
        shipping.model.ts
    shared/
      components/
        navbar/
        footer/
        product-card/
        loading-spinner/
        toast/
    pages/
      home/
      catalog/
      product-detail/
      cart/
      checkout/
      payment-confirm/
      orders/
      order-detail/
      auth/
        login/
        register/
    app.routes.ts                # lazy route declarations
    app.config.ts                # provideRouter, provideHttpClient, interceptors
    app.ts                       # root component with navbar + router-outlet
```

---

### Routing Structure

#### [MODIFY] [app.routes.ts](file:///e:/Sources/strawberry-ecommerce/frontend/src/app/app.routes.ts)

```
/                    → HomeComponent           (public)
/catalog             → CatalogComponent        (public)
/products/:slug      → ProductDetailComponent  (public)
/cart                → CartComponent           (auth-guarded)
/checkout            → CheckoutComponent       (auth-guarded)
/checkout/payment    → PaymentConfirmComponent (auth-guarded)
/orders              → OrdersComponent         (auth-guarded)
/orders/:id          → OrderDetailComponent    (auth-guarded)
/login               → LoginComponent          (redirect if logged in)
/register            → RegisterComponent       (redirect if logged in)
```

All customer routes under `/cart`, `/checkout`, `/orders` will be wrapped behind `authGuard`.

---

### API Service Layer

#### [NEW] core/services/catalog.service.ts

```ts
GET /api/v1/public/catalog/search?page&size&search&categoryId&minPrice&maxPrice&inStock
GET /api/v1/public/catalog/products/:slug
GET /api/v1/public/catalog/filters
```

Maps to `ProductSummary[]` and `ProductDetail`.

#### [NEW] core/services/cart.service.ts

```ts
GET    /api/v1/customer/cart
POST   /api/v1/customer/cart/items
PUT    /api/v1/customer/cart/items/:id
DELETE /api/v1/customer/cart/items/:id
DELETE /api/v1/customer/cart
```

Exposes a `cart` Signal updated after each mutation.

#### [NEW] core/services/order.service.ts

```ts
POST /api/v1/customer/orders/checkout
GET  /api/v1/customer/orders
GET  /api/v1/customer/orders/:id
POST /api/v1/customer/orders/:id/payment-confirmation  (multipart)
GET  /api/v1/customer/orders/:id/tracking
```

#### [NEW] core/services/shipping.service.ts

```ts
GET /api/v1/public/shipping/zones
GET /api/v1/public/shipping/methods?zoneId=:id
```

#### [NEW] core/auth/auth.service.ts

```ts
POST /api/v1/auth/register/customer
POST /api/v1/auth/login
```

Stores `accessToken` in `localStorage`. Exposes `currentUser` Signal.

---

### State Management Strategy

- **Auth state**: `AuthService` holds a `currentUser = signal<User | null>(null)`. Populated from localStorage on `APP_INITIALIZER`.
- **Cart state**: `CartService` holds `cart = signal<Cart | null>(null)`. The Navbar subscribes to `cart.totalItems` to show the badge.
- **Catalog state**: Stateless — `CatalogComponent` fetches on query param changes using `toSignal(activatedRoute.queryParams)`.
- **Checkout state**: Simple form state, no global state needed.
- No NgRx needed for this phase.

---

### Authentication Handling

#### [NEW] core/auth/auth.interceptor.ts
Adds `Authorization: Bearer <token>` to every outbound request. If a 401 is returned, clears the token and redirects to `/login`.

#### [NEW] core/auth/auth.guard.ts
Functional guard using `inject(AuthService)`. Redirects unauthenticated users to `/login` with `returnUrl` query param preserved.

---

### Page Descriptions

| Page | Key Features |
|---|---|
| **Home** | Hero banner, featured/new products grid (first 8 from catalog API), category quick links |
| **Catalog** | Search bar, filters (category, price range, in-stock toggle), paginated product grid |
| **Product Detail** | Variant selector, price display, add-to-cart button, product images |
| **Cart** | Item list with quantity controls, remove, total price, proceed-to-checkout CTA |
| **Checkout** | Shipping zone & method selectors (fetched from API), delivery address form, order summary, confirm button |
| **Payment Confirm** | Upload receipt image (file input), submit to backend, show pending message |
| **Orders** | Paginated list of customer orders with status badge |
| **Order Detail / Tracking** | Full order breakdown, payment status, shipment tracking number & status |
| **Login / Register** | Reactive forms, JWT storage on success, redirect to returnUrl |

---

### DTO Mapping (Backend → UI Models)

| Backend Response | UI Model |
|---|---|
| `ProductSummaryDto` | `ProductCard` (id, title, slug, price, image, inStock) |
| `ProductDetailDto` | `ProductDetail` (+ variants, characteristics, images) |
| `CartResponseDto` | `Cart` (items[], totalPrice, totalItems) |
| `OrderResponseDto` | `Order` (id, number, status, paymentStatus, shippingCost, items[]) |
| `ShipmentResponseDto` | `Tracking` (trackingNumber, carrier, status, shippedAt, deliveredAt) |

---

### Error & Loading Handling

- A shared `LoadingSpinnerComponent` shows during API calls (tracked via a `loading = signal(false)` per service call).
- A `ToastService` with a fixed toast tray for success/error messages.
- Global HTTP error handling via the `AuthInterceptor` catches 401/403/500.

---

## Verification Plan

### Manual Verification (in-browser)

1. Navigate `/` — confirm hero + product cards load.
2. Navigate `/catalog?search=test` — confirm query filters work.
3. Navigate `/products/:slug` — confirm variant selector and add-to-cart.
4. Navigate `/cart` — confirm items and total match backend.
5. Navigate `/checkout` — confirm shipping zones/methods populate, confirm checkout creates order.
6. Navigate `/checkout/payment` — upload test image and confirm confirmation API is called.
7. Navigate `/orders` — confirm all customer orders visible.
8. Navigate `/orders/:id` — confirm tracking section shows shipment.

### API Smoke Tests

We will build a simple `verify_frontend_f1.py` that hits the frontend dev server and verifies critical routes return HTTP 200.
