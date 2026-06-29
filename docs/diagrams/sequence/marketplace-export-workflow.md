# Marketplace Export Workflow

> Sequence showing how approved products are exported to marketplace-specific formats.

```mermaid
sequenceDiagram
    participant S as Seller
    participant FE as Frontend
    participant API as API Gateway
    participant L as Lambda (Export API)
    participant SF as Step Functions
    participant MIE as Marketplace Intelligence Engine
    participant VAL as Validation Engine
    participant GEN as Export Generator
    participant DB as DynamoDB
    participant S3 as S3 (Exports)
    participant N as Notification Service

    S->>FE: Select products + target marketplace
    FE->>API: POST /exports (productIds, marketplace: "takealot")
    API->>L: Create export job
    L->>DB: Save export record (status: processing)
    L->>SF: Trigger export workflow
    L-->>API: 202 Accepted (exportId)
    API-->>FE: Export started
    FE-->>S: "Generating export..."

    Note over SF: Step 1 — Load Marketplace Schema
    SF->>MIE: Get schema for "takealot"
    MIE->>DB: Load marketplace config
    DB-->>MIE: Column definitions, validation rules, taxonomy
    MIE-->>SF: Complete marketplace schema

    Note over SF: Step 2 — Validate Each Product
    loop For each product
        SF->>VAL: Validate product against schema
        VAL->>VAL: Check required fields present
        VAL->>VAL: Validate field formats + lengths
        VAL->>VAL: Check category in taxonomy
        VAL->>VAL: Verify image compliance
        alt Passes validation
            VAL-->>SF: Valid ✓
        else Fails validation
            VAL-->>SF: Errors (field, reason)
            SF->>DB: Log validation errors
        end
    end

    Note over SF: Step 3 — Generate Export File
    SF->>GEN: Generate CSV (valid products only)
    GEN->>GEN: Map internal fields → marketplace columns
    GEN->>GEN: Apply formatting rules (dates, prices, text)
    GEN->>GEN: Generate CSV with correct encoding
    GEN->>S3: Upload export file
    S3-->>GEN: File URL (pre-signed, 7d expiry)
    GEN-->>SF: Export complete

    Note over SF: Step 4 — Finalise
    SF->>DB: Update export (status: complete, fileUrl)
    SF->>N: Notify seller
    N-->>S: "Export ready — 47/50 products valid"

    S->>FE: Download export file
    FE->>S3: GET (pre-signed URL)
    S3-->>S: CSV file download
```

---

## Export Validation Rules (Example: Takealot)

| Field | Rule | Action on Failure |
|-------|------|-------------------|
| Title | Required, 5–150 chars | Reject product |
| Category | Must exist in taxonomy | Flag for manual mapping |
| Price | Required, > 0, ZAR format | Reject product |
| Image URL | Required, HTTPS, .jpg/.png | Reject product |
| Description | Required, 50–5000 chars | Truncate or reject |
| Barcode | Valid EAN-13 or UPC-A | Warning (not blocking) |
| Stock | Required, integer >= 0 | Reject product |
