# Core API Specification (OpenAPI 3.0 Snippet)

```yaml
openapi: 3.0.0
info:
  title: Wildberries E-Commerce Platform API
  version: 1.1.0
paths:
  /api/v1/auth/register/seller:
    post:
      summary: Register a new seller
      requestBody:
        required: true
        content:
          application/json:
            schema:
              type: object
              properties:
                email: { type: string }
                password: { type: string }
      responses:
        '201':
          description: Created (Status PENDING_APPROVAL)

  /api/v1/admin/sellers/{id}/approve:
    post:
      summary: Admin approves a seller profile
      parameters:
        - name: id
          in: path
          required: true
          schema: { type: string, format: uuid }
      responses:
        '200':
          description: Seller profile approved

  /api/v1/shops:
    post:
      summary: Create a new shop & integration
      description: Only accessible by APPROVED sellers.
      requestBody:
        required: true
        content:
          application/json:
            schema:
              type: object
              properties:
                name: { type: string }
                slug: { type: string }
                wbApiKey: { type: string }
                bankDetails: 
                  type: object
                  properties:
                    bankName: { type: string }
                    accountNumber: { type: string }
                    accountHolderName: { type: string }
      responses:
        '201':
          description: Shop & Integration Created

  /api/v1/shops/{shopId}/sync/full:
    post:
      summary: Trigger a COMPLETE WB catalog sync
      responses:
        '202':
          description: Sync job queued
          content:
            application/json:
              schema:
                type: object
                properties:
                  syncJobId: { type: string, format: uuid }
                  status: { type: string }
                  startedAt: { type: string, format: date-time }

  /api/v1/shops/{shopId}/sync/incremental:
    post:
      summary: Trigger a delta WB catalog sync based on cursors
      responses:
        '202':
          description: Sync job queued
          content:
            application/json:
              schema:
                type: object
                properties:
                  syncJobId: { type: string, format: uuid }
                  status: { type: string }
                  startedAt: { type: string, format: date-time }

  /api/v1/shops/{shopId}/products/{productId}:
    put:
      summary: Update base product settings

  /api/v1/shops/{shopId}/products/{productId}/visibility:
    put:
      summary: Toggle product visibility

  /api/v1/shops/{shopId}/products/{productId}/content:
    put:
      summary: Update custom title and description

  /api/v1/shops/{shopId}/variants/{variantId}/pricing:
    put:
      summary: Update variant local pricing overrides

  /api/v1/shops/{shopId}/variants/{variantId}/inventory:
    put:
      summary: Update variant local stock overrides

  /api/v1/orders:
    post:
      summary: Customer places an order
      requestBody:
        required: true
        content:
          application/json:
            schema:
              type: object
              properties:
                shopId: { type: string }
                items:
                  type: array
                  items:
                    type: object
                    properties:
                      productId: { type: string, format: uuid }
                      variantId: { type: string, format: uuid }
                      quantity: { type: integer }
      responses:
        '201':
          description: Order created with WAITING_PAYMENT_CONFIRMATION status

  /api/v1/orders/{orderCode}/payment/confirm:
    post:
      summary: Customer submits payment confirmation
      requestBody:
        content:
          multipart/form-data:
            schema:
              type: object
              properties:
                receipt: { type: string, format: binary }
                referenceMessage: { type: string }
                transferAmount: { type: number }
                payerName: { type: string }
                transferTime: { type: string, format: date-time }
      responses:
        '200':
          description: Payment proof uploaded (WAITING_SELLER_REVIEW)

  /api/v1/seller/orders/{orderId}/payments/{paymentId}/review:
    post:
      summary: Seller confirms or rejects the bank payment
      requestBody:
        required: true
        content:
          application/json:
            schema:
              type: object
              properties:
                action: { type: string, enum: [CONFIRM, REJECT] }
                reviewerNote: { type: string }
      responses:
        '200':
          description: Payment status updated
```
