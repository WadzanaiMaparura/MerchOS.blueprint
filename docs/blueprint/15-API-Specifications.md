# MerchOS Engineering Blueprint

## Volume 15 — API Specifications

---

| Field | Value |
|-------|-------|
| **Document ID** | MERCH-015 |
| **Title** | API Specifications |
| **Version** | 0.1 |
| **Status** | Draft |
| **Owner** | Wadzanai Maparura |
| **Technical Lead** | Kiro AI |
| **Created** | 2026-06-27 |
| **Last Updated** | 2026-06-27 |
| **Next Review** | 2026-07-11 |
| **Classification** | Internal — Confidential |
| **Related Documents** | MERCH-003 (Functional Requirements), MERCH-014 (Database Design), MERCH-017 (Backend Architecture) |

---

## Revision History

| Version | Date | Author | Change Description |
|---------|------|--------|-------------------|
| 0.1 | 2026-06-27 | Kiro AI / Wadzanai Maparura | Initial draft |

---

## Table of Contents

1. [Purpose](#1-purpose)
2. [Scope](#2-scope)
3. [API Design Principles](#3-api-design-principles)
4. [Authentication & Headers](#4-authentication--headers)
5. [Error Handling](#5-error-handling)
6. [Products API](#6-products-api)
7. [Intelligence API](#7-intelligence-api)
8. [Exports API](#8-exports-api)
9. [Inventory API](#9-inventory-api)
10. [Suppliers API](#10-suppliers-api)
11. [Images API](#11-images-api)
12. [Marketplaces API](#12-marketplaces-api)
13. [Tenants & Users API](#13-tenants--users-api)
14. [Webhooks](#14-webhooks)
15. [Assumptions](#15-assumptions)
16. [Dependencies](#16-dependencies)
17. [References](#17-references)

---


## 1. Purpose

This document defines the complete REST API specification for MerchOS. Every endpoint, request/response schema, authentication requirement, and error format is documented to enable parallel frontend and backend development.

---

## 2. Scope

Covers: All public-facing API endpoints (v1), authentication flow, request/response schemas, error handling, pagination, and webhook contracts. Excludes internal service-to-service communication and admin-only APIs (documented separately).

---

## 3. API Design Principles

| Principle | Implementation |
|-----------|---------------|
| RESTful | Resource-oriented URLs; standard HTTP verbs |
| JSON | All request/response bodies as `application/json` |
| Versioned | URL path versioning: `/v1/...` |
| Consistent | Same patterns across all resource types |
| Paginated | Cursor-based pagination for all list endpoints |
| Idempotent | PUT and DELETE are idempotent; POST uses idempotency keys |
| Tenant-scoped | All data automatically scoped to authenticated tenant |
| Documented | OpenAPI 3.0 spec generated and maintained |

### 3.1 URL Structure

```
https://api.merchos.com/v1/{resource}
https://api.merchos.com/v1/{resource}/{id}
https://api.merchos.com/v1/{resource}/{id}/{sub-resource}
```

### 3.2 HTTP Verbs

| Verb | Usage | Idempotent |
|------|-------|-----------|
| GET | Retrieve resource(s) | Yes |
| POST | Create resource / trigger action | No (use idempotency key) |
| PUT | Full update (replace) | Yes |
| PATCH | Partial update | Yes |
| DELETE | Remove resource (soft-delete) | Yes |

### 3.3 Response Envelope

All responses follow a consistent envelope:

```json
{
  "data": { },
  "meta": {
    "requestId": "req_abc123",
    "timestamp": "2026-06-27T10:30:00Z"
  },
  "pagination": {
    "nextCursor": "eyJsYXN0S2V5Ijoi...",
    "hasMore": true,
    "pageSize": 20
  }
}
```

---

## 4. Authentication & Headers

### 4.1 Required Headers

| Header | Required | Description |
|--------|----------|-------------|
| `Authorization` | Yes | `Bearer {access_token}` (Cognito JWT) |
| `Content-Type` | Yes (POST/PUT/PATCH) | `application/json` |
| `X-Request-ID` | Recommended | Client-generated UUID for tracing |
| `X-Idempotency-Key` | Recommended (POST) | Prevent duplicate creation |

### 4.2 Authentication Flow

| Step | Endpoint | Description |
|------|----------|-------------|
| 1 | Cognito Hosted UI / SDK | User authenticates (email+password or social) |
| 2 | — | Receive access_token + id_token + refresh_token |
| 3 | All API calls | Include `Authorization: Bearer {access_token}` |
| 4 | Token refresh | Use refresh_token to get new access_token (client-side) |

### 4.3 Token Claims (available in Lambda context)

```json
{
  "sub": "cognito-user-sub-id",
  "custom:tenantId": "t_xyz789",
  "custom:role": "admin",
  "custom:tier": "professional",
  "email": "user@example.com"
}
```

---

## 5. Error Handling

### 5.1 Error Response Format

```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Request validation failed",
    "details": [
      { "field": "title", "message": "Title is required", "rule": "required" },
      { "field": "barcode", "message": "Invalid EAN-13 checksum", "rule": "checksum" }
    ],
    "requestId": "req_abc123"
  }
}
```

### 5.2 Error Codes

| HTTP Status | Code | Description |
|-------------|------|-------------|
| 400 | `VALIDATION_ERROR` | Request body/params failed validation |
| 400 | `INVALID_REQUEST` | Malformed request structure |
| 401 | `UNAUTHORIZED` | Missing or invalid authentication token |
| 403 | `FORBIDDEN` | Authenticated but insufficient permissions |
| 404 | `NOT_FOUND` | Resource does not exist (or not in tenant scope) |
| 409 | `CONFLICT` | Resource already exists (duplicate) |
| 422 | `UNPROCESSABLE` | Semantic error (valid syntax but invalid logic) |
| 429 | `RATE_LIMITED` | Too many requests; retry after delay |
| 500 | `INTERNAL_ERROR` | Server error (logged; requestId for support) |
| 503 | `SERVICE_UNAVAILABLE` | Temporary outage; retry |

---

## 6. Products API

### 6.1 Endpoints

| Method | Path | Description |
|--------|------|-------------|
| POST | `/v1/products` | Create a product |
| GET | `/v1/products` | List products (paginated, filterable) |
| GET | `/v1/products/{id}` | Get product by ID |
| PATCH | `/v1/products/{id}` | Update product (partial) |
| DELETE | `/v1/products/{id}` | Archive product (soft-delete) |
| POST | `/v1/products/bulk-import` | Bulk import from CSV/Excel |
| GET | `/v1/products/{id}/variants` | List variants for product |
| POST | `/v1/products/{id}/variants` | Create variant |
| PATCH | `/v1/products/{id}/variants/{variantId}` | Update variant |
| DELETE | `/v1/products/{id}/variants/{variantId}` | Delete variant |
| GET | `/v1/products/{id}/readiness` | Get marketplace readiness scores |

### 6.2 Create Product

**POST** `/v1/products`

**Request:**
```json
{
  "title": "Samsung Galaxy S24 Ultra 256GB",
  "brand": "Samsung",
  "category": "Electronics > Smartphones",
  "sku": "SAM-S24U-BLK-256",
  "barcode": "6001234567890",
  "description": "<p>Premium smartphone with 200MP camera...</p>",
  "attributes": {
    "storage": "256GB",
    "colour": "Titanium Black",
    "ram": "12GB",
    "screen_size": "6.8 inches"
  },
  "pricing": {
    "costPrice": 18000.00,
    "sellingPrice": 24999.00,
    "rrp": 27999.00,
    "currency": "ZAR"
  },
  "tags": ["samsung", "flagship", "5G"],
  "status": "draft"
}
```

**Response (201):**
```json
{
  "data": {
    "id": "p_abc123",
    "tenantId": "t_xyz789",
    "title": "Samsung Galaxy S24 Ultra 256GB",
    "brand": "Samsung",
    "sku": "SAM-S24U-BLK-256",
    "status": "draft",
    "completenessScore": 72,
    "createdAt": "2026-06-27T10:30:00Z",
    "updatedAt": "2026-06-27T10:30:00Z"
  },
  "meta": { "requestId": "req_abc123" }
}
```

### 6.3 List Products

**GET** `/v1/products?status=active&category=Electronics&pageSize=20&cursor={cursor}`

**Query Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `status` | String | Filter: draft, active, archived |
| `category` | String | Filter by category (prefix match) |
| `tags` | String | Comma-separated tag filter |
| `search` | String | Full-text search (title, description) |
| `readinessMin` | Number | Minimum completeness score |
| `sort` | String | `updatedAt`, `createdAt`, `title` |
| `order` | String | `asc`, `desc` (default: desc) |
| `pageSize` | Number | 1–100 (default: 20) |
| `cursor` | String | Pagination cursor (from previous response) |

---

## 7. Intelligence API

### 7.1 Endpoints

| Method | Path | Description |
|--------|------|-------------|
| POST | `/v1/intelligence/enrich` | Trigger AI enrichment for a product |
| POST | `/v1/intelligence/batch` | Batch AI enrichment (multiple products) |
| POST | `/v1/intelligence/describe` | Generate description only |
| POST | `/v1/intelligence/extract` | Extract attributes from text |
| POST | `/v1/intelligence/categorise` | Get category recommendations |
| GET | `/v1/intelligence/credits` | Get remaining AI credits |
| GET | `/v1/intelligence/jobs/{jobId}` | Get batch job status |

### 7.2 Enrich Product

**POST** `/v1/intelligence/enrich`

**Request:**
```json
{
  "productId": "p_abc123",
  "tasks": ["describe", "extract", "categorise", "seo"],
  "targetMarketplace": "takealot",
  "options": {
    "tone": "professional",
    "language": "en",
    "overwriteExisting": false
  }
}
```

**Response (202 — Accepted):**
```json
{
  "data": {
    "jobId": "job_def456",
    "productId": "p_abc123",
    "status": "processing",
    "tasks": ["describe", "extract", "categorise", "seo"],
    "estimatedDuration_s": 12,
    "creditsToConsume": 1
  }
}
```

### 7.3 Get Credits

**GET** `/v1/intelligence/credits`

**Response (200):**
```json
{
  "data": {
    "tier": "professional",
    "monthlyAllocation": 5000,
    "used": 1234,
    "remaining": 3766,
    "resetDate": "2026-07-01T00:00:00Z",
    "addOnCredits": 200
  }
}
```

---

## 8. Exports API

### 8.1 Endpoints

| Method | Path | Description |
|--------|------|-------------|
| POST | `/v1/exports` | Create a new export |
| GET | `/v1/exports` | List export history |
| GET | `/v1/exports/{id}` | Get export details + download URL |
| POST | `/v1/exports/validate` | Dry-run validation (no file generation) |
| GET | `/v1/exports/{id}/report` | Get validation report |
| DELETE | `/v1/exports/{id}` | Delete export file (retain record) |

### 8.2 Create Export

**POST** `/v1/exports`

**Request:**
```json
{
  "marketplace": "takealot",
  "mode": "csv",
  "productSelection": {
    "type": "filter",
    "filters": {
      "status": "active",
      "readinessScoreMin": 80
    }
  },
  "options": {
    "deltaOnly": true,
    "includeOutOfStock": false
  }
}
```

**Response (202):**
```json
{
  "data": {
    "exportId": "exp_abc123",
    "status": "processing",
    "marketplace": "takealot",
    "estimatedProducts": 475,
    "estimatedDuration_s": 30
  }
}
```

### 8.3 Get Export (with download URL)

**GET** `/v1/exports/exp_abc123`

**Response (200):**
```json
{
  "data": {
    "exportId": "exp_abc123",
    "status": "completed",
    "marketplace": "takealot",
    "mode": "csv",
    "productCount": 475,
    "fileSize_bytes": 1245000,
    "downloadUrl": "https://merchos-prod-exports.s3.af-south-1.amazonaws.com/...",
    "downloadUrlExpiry": "2026-06-28T10:00:00Z",
    "validationSummary": { "passed": 475, "failed": 20, "warnings": 8 },
    "createdAt": "2026-06-27T10:00:00Z",
    "completedAt": "2026-06-27T10:02:15Z"
  }
}
```

---

## 9. Inventory API

### 9.1 Endpoints

| Method | Path | Description |
|--------|------|-------------|
| GET | `/v1/inventory/{productId}` | Get stock for product (all warehouses) |
| PUT | `/v1/inventory/{productId}/adjust` | Adjust stock |
| POST | `/v1/inventory/bulk-update` | Bulk stock update (CSV) |
| GET | `/v1/inventory/low-stock` | List low-stock products |
| GET | `/v1/inventory/{productId}/history` | Stock movement history |

### 9.2 Adjust Stock

**PUT** `/v1/inventory/{productId}/adjust`

**Request:**
```json
{
  "variantId": "v_def456",
  "warehouseId": "default",
  "adjustmentType": "receive",
  "quantity": 100,
  "reason": "PO-2026-0045 received from TechDistro"
}
```

**Response (200):**
```json
{
  "data": {
    "productId": "p_abc123",
    "variantId": "v_def456",
    "warehouseId": "default",
    "previousOnHand": 50,
    "newOnHand": 150,
    "available": 145,
    "reserved": 5,
    "movementId": "mov_ghi789"
  }
}
```

---

## 10. Suppliers API

### 10.1 Endpoints

| Method | Path | Description |
|--------|------|-------------|
| POST | `/v1/suppliers` | Register a new supplier |
| GET | `/v1/suppliers` | List suppliers |
| GET | `/v1/suppliers/{id}` | Get supplier details |
| PATCH | `/v1/suppliers/{id}` | Update supplier profile |
| POST | `/v1/suppliers/{id}/ingest` | Trigger catalogue ingestion |
| GET | `/v1/suppliers/{id}/reports` | List ingestion reports |
| GET | `/v1/suppliers/{id}/reports/{reportId}` | Get specific report |
| POST | `/v1/suppliers/{id}/mapping` | Configure column mapping |

### 10.2 Trigger Ingestion

**POST** `/v1/suppliers/{id}/ingest`

**Request:**
```json
{
  "fileKey": "imports/t_xyz789/techdistro_2026-06.csv",
  "options": {
    "skipDuplicates": false,
    "updateExisting": true,
    "triggerEnrichment": true
  }
}
```

**Response (202):**
```json
{
  "data": {
    "jobId": "ing_abc123",
    "supplierId": "sup_abc123",
    "status": "processing",
    "fileName": "techdistro_2026-06.csv"
  }
}
```

---

## 11. Images API

### 11.1 Endpoints

| Method | Path | Description |
|--------|------|-------------|
| POST | `/v1/images/upload-url` | Get pre-signed upload URL |
| GET | `/v1/products/{id}/images` | List images for product |
| PATCH | `/v1/products/{id}/images/reorder` | Reorder product images |
| DELETE | `/v1/images/{imageId}` | Delete an image |
| GET | `/v1/images/{imageId}/analysis` | Get AI analysis results |
| GET | `/v1/images/{imageId}/compliance` | Get marketplace compliance status |

### 11.2 Get Upload URL

**POST** `/v1/images/upload-url`

**Request:**
```json
{
  "productId": "p_abc123",
  "fileName": "product-front.jpg",
  "contentType": "image/jpeg",
  "fileSizeBytes": 2500000
}
```

**Response (200):**
```json
{
  "data": {
    "uploadUrl": "https://merchos-prod-media.s3.af-south-1.amazonaws.com/...",
    "fields": { "key": "t_xyz789/originals/p_abc123/img_new1.jpg", "...": "..." },
    "imageId": "img_new1",
    "expiresAt": "2026-06-27T10:45:00Z"
  }
}
```

---

## 12. Marketplaces API

### 12.1 Endpoints

| Method | Path | Description |
|--------|------|-------------|
| GET | `/v1/marketplaces` | List available marketplaces |
| GET | `/v1/marketplaces/{id}` | Get marketplace details |
| GET | `/v1/marketplaces/{id}/categories` | Browse category taxonomy |
| GET | `/v1/marketplaces/{id}/schema` | Get current CSV schema |
| POST | `/v1/marketplaces/{id}/validate` | Validate a product against marketplace |

### 12.2 Validate Product

**POST** `/v1/marketplaces/takealot/validate`

**Request:**
```json
{
  "productId": "p_abc123"
}
```

**Response (200):**
```json
{
  "data": {
    "productId": "p_abc123",
    "marketplace": "takealot",
    "overallStatus": "PASS",
    "score": 95,
    "passedFields": 25,
    "failedFields": 0,
    "warningFields": 2,
    "errors": [],
    "warnings": [
      { "field": "key_feature_5", "message": "Optional field not populated" }
    ]
  }
}
```

---

## 13. Tenants & Users API

### 13.1 Endpoints

| Method | Path | Description |
|--------|------|-------------|
| GET | `/v1/tenant` | Get current tenant info |
| PATCH | `/v1/tenant` | Update tenant settings |
| GET | `/v1/tenant/usage` | Get current usage & quotas |
| GET | `/v1/users` | List users in tenant |
| POST | `/v1/users/invite` | Invite user to tenant |
| PATCH | `/v1/users/{id}/role` | Change user role |
| DELETE | `/v1/users/{id}` | Remove user from tenant |
| GET | `/v1/notifications` | List user notifications |
| PATCH | `/v1/notifications/{id}/read` | Mark notification as read |

### 13.2 Get Tenant Usage

**GET** `/v1/tenant/usage`

**Response (200):**
```json
{
  "data": {
    "tier": "professional",
    "usage": {
      "products": { "used": 4500, "limit": 50000 },
      "teamMembers": { "used": 6, "limit": 10 },
      "aiCredits": { "used": 1234, "limit": 5000, "resetDate": "2026-07-01" },
      "storage_gb": { "used": 12.5, "limit": 100 },
      "marketplaces": { "used": 3, "limit": 5 },
      "exports_this_month": { "used": 45, "limit": null }
    }
  }
}
```

---

## 14. Webhooks

### 14.1 Outbound Webhook Events (Enterprise tier)

| Event | Trigger | Payload Summary |
|-------|---------|----------------|
| `product.created` | New product created | productId, title, status |
| `product.updated` | Product modified | productId, changedFields |
| `export.completed` | Export finished | exportId, marketplace, productCount, downloadUrl |
| `export.failed` | Export failed | exportId, error |
| `enrichment.completed` | AI enrichment done | productId, tasks, confidence |
| `inventory.low_stock` | Stock below threshold | productId, currentStock, threshold |
| `supplier.ingested` | Supplier feed processed | supplierId, summary |

### 14.2 Webhook Configuration

**POST** `/v1/webhooks`

```json
{
  "url": "https://customer-system.example.com/merchos-webhook",
  "events": ["export.completed", "inventory.low_stock"],
  "secret": "whsec_customer_provided_secret"
}
```

### 14.3 Webhook Delivery

| Aspect | Specification |
|--------|--------------|
| Format | JSON POST to configured URL |
| Headers | `X-MerchOS-Signature` (HMAC-SHA256), `X-MerchOS-Event`, `X-MerchOS-Timestamp` |
| Retry | 3 attempts with exponential backoff (1s, 10s, 60s) |
| Timeout | 10 seconds per attempt |
| Verification | HMAC signature using shared secret |

---

## 15. Assumptions

| # | Assumption | Impact if Invalid |
|---|-----------|-------------------|
| A1 | REST API sufficient (no need for GraphQL initially) | Add GraphQL layer for complex frontend queries |
| A2 | Cursor-based pagination handles all list use cases | May need offset pagination for specific UX needs |
| A3 | JSON request/response bodies sufficient | May need multipart for file uploads (handled via pre-signed URLs) |
| A4 | Single API version (v1) sufficient for initial phases | Version management strategy needed earlier |
| A5 | 29-second API Gateway timeout acceptable for all sync endpoints | Move slow operations to async (already designed) |

---

## 16. Dependencies

| Dependency | Impact |
|-----------|--------|
| Amazon API Gateway | API hosting and routing |
| Amazon Cognito | JWT token generation and validation |
| Lambda functions | Request handling |
| DynamoDB | Data persistence for all endpoints |
| S3 | Pre-signed URLs for file operations |
| Frontend (MERCH-016) | Primary API consumer |
| Database Design (MERCH-014) | Data model backing APIs |

---

## 17. References

| # | Reference |
|---|-----------|
| 1 | OpenAPI 3.0 Specification |
| 2 | MERCH-003 (Functional Requirements) |
| 3 | MERCH-014 (Database Design) |
| 4 | MERCH-017 (Backend Architecture) |
| 5 | REST API Design Best Practices (Microsoft) |
| 6 | JSON:API Specification (reference for conventions) |

---

*End of Volume 15 — API Specifications*

*Previous: Volume 14 — Database Design (MERCH-014)*
*Next: Volume 16 — Frontend Architecture (MERCH-016)*
