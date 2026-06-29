# Product Ingestion Flow

> Sequence diagram showing the complete product upload and AI enrichment pipeline.

```mermaid
sequenceDiagram
    participant S as Seller
    participant FE as Frontend (Next.js)
    participant API as API Gateway
    participant AUTH as Cognito
    participant L as Lambda (Product API)
    participant DB as DynamoDB
    participant S3 as S3 (Media)
    participant EB as EventBridge
    participant SF as Step Functions
    participant T as Textract (OCR)
    participant R as Rekognition
    participant B as Bedrock (Claude)
    participant N as SNS (Notifications)

    S->>FE: Upload product (data + images)
    FE->>API: POST /products (JWT token)
    API->>AUTH: Validate JWT
    AUTH-->>API: Token valid (tenantId, role)
    API->>L: Invoke with tenant context
    
    L->>DB: Save raw product (status: pending)
    L->>S3: Store product images
    L->>EB: Emit "product.created" event
    L-->>API: 201 Created (productId)
    API-->>FE: Product created
    FE-->>S: "Processing with AI..."

    EB->>SF: Trigger enrichment workflow

    Note over SF: Step 1 — Image Analysis
    SF->>T: Extract text from images (OCR)
    T-->>SF: Extracted text + confidence
    SF->>R: Analyse images (labels, quality)
    R-->>SF: Labels, categories, quality scores

    Note over SF: Step 2 — AI Enrichment
    SF->>B: Generate description + attributes
    Note right of B: System prompt + product data<br/>+ marketplace context<br/>+ RAG category knowledge
    B-->>SF: AI output + confidence scores

    Note over SF: Step 3 — Validate & Save
    SF->>SF: Validate output schema
    SF->>SF: Check confidence thresholds
    SF->>DB: Save enriched product (status: draft)
    SF->>EB: Emit "product.enriched" event

    EB->>N: Send notification
    N-->>S: "Your product is ready for review"

    S->>FE: Review AI suggestions
    S->>FE: Accept / Edit / Reject
    FE->>API: PATCH /products/{id}/approve
    API->>L: Update product status
    L->>DB: Status: approved
```

---

## Key Decision Points

| Step | Decision | Outcome |
|------|----------|---------|
| Confidence >= 0.9 | Auto-suggest as "Recommended" | Green badge in UI |
| Confidence 0.7–0.89 | Suggest with review flag | Yellow badge |
| Confidence < 0.7 | Require manual review | Orange/red badge |
| AI unavailable | Graceful degradation | Manual mode enabled |
