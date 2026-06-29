# AI Processing Pipeline

> Detailed sequence showing how AI requests are managed, queued, and processed.

```mermaid
sequenceDiagram
    participant T as Trigger (Event/API)
    participant Q as SQS (AI Queue)
    participant SF as Step Functions
    participant R as Model Router
    participant P as Prompt Builder
    participant S3 as S3 (Prompt Cache)
    participant KB as Knowledge Base (RAG)
    participant B as Bedrock (Claude)
    participant V as Output Validator
    participant DB as DynamoDB
    participant M as CloudWatch Metrics

    T->>Q: Enqueue AI request<br/>(productId, tenantId, tasks)
    Q->>SF: Trigger workflow (batch size: 5)
    
    Note over SF,R: Model Selection
    SF->>R: Determine model for task + tenant tier
    R->>R: Check tier (Starter→Haiku, Pro→Sonnet)
    R->>R: Check budget remaining
    R-->>SF: Model config (model, temp, maxTokens)

    Note over SF,S3: Prompt Assembly
    SF->>S3: Load prompt template (versioned)
    S3-->>SF: System prompt template
    SF->>KB: Query for relevant category context
    KB-->>SF: RAG results (top 5 similar)
    SF->>P: Build final prompt<br/>(template + product + RAG + schema)
    P-->>SF: Assembled prompt (token count: ~3000)

    Note over SF,B: AI Inference
    SF->>B: InvokeModel (Claude 3.5 Sonnet)
    B-->>SF: Response (JSON + tokens used)

    Note over SF,V: Validation
    SF->>V: Validate output against JSON Schema
    V->>V: Check confidence scores
    V->>V: Verify no hallucinated specs
    
    alt Valid Output (confidence >= 0.7)
        V-->>SF: Validation passed
        SF->>DB: Save enriched data (status: draft)
        SF->>M: Log: success, tokens, latency, confidence
    else Invalid Output
        V-->>SF: Validation failed
        SF->>SF: Retry with adjusted parameters
        alt Retry succeeds
            SF->>DB: Save enriched data
        else Retry fails (max 3)
            SF->>DB: Log failure + raw response
            SF->>M: Log: failure, error type
        end
    end
```

---

## Model Routing Decision Tree

```mermaid
graph TD
    REQUEST[AI Request] --> CHECK_BUDGET{Tenant has<br/>AI credits?}
    CHECK_BUDGET -->|No| REJECT[Reject — notify user<br/>to upgrade or wait]
    CHECK_BUDGET -->|Yes| CHECK_TIER{Tenant Tier?}
    
    CHECK_TIER -->|Enterprise/Pro| SONNET[Claude 3.5 Sonnet<br/>~$0.031/product]
    CHECK_TIER -->|Growth| CHECK_TASK{Task Type?}
    CHECK_TIER -->|Starter| HAIKU[Claude 3 Haiku<br/>~$0.0026/product]
    
    CHECK_TASK -->|Description / Extract| SONNET
    CHECK_TASK -->|Classify / Summarise| HAIKU
    
    SONNET --> CHECK_AVAIL{Model Available?}
    HAIKU --> CHECK_AVAIL
    
    CHECK_AVAIL -->|Yes| INVOKE[Invoke Model]
    CHECK_AVAIL -->|Rate Limited| QUEUE[Re-queue with backoff]
    CHECK_AVAIL -->|Down| FALLBACK[Use fallback model]
    FALLBACK --> INVOKE
```

---

## Token Cost Tracking

| Model | Input Cost | Output Cost | Avg per Product | Monthly Budget (Growth) |
|-------|-----------|-------------|-----------------|------------------------|
| Claude 3.5 Sonnet | $3.00/1M tokens | $15.00/1M tokens | ~$0.031 | 500 products |
| Claude 3 Haiku | $0.25/1M tokens | $1.25/1M tokens | ~$0.0026 | ~6,000 products |
