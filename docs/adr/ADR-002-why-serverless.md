# ADR-002: Why Serverless?

| Field | Value |
|-------|-------|
| **Status** | Approved |
| **Date** | 2026-06-27 |
| **Decision Makers** | Wadzanai Maparura |
| **Category** | Compute Architecture |

---

## Context

MerchOS needs a compute and data platform that:
- Costs nothing when idle (startup economics)
- Scales automatically with demand
- Requires minimal operational overhead
- Supports event-driven processing patterns
- Handles both synchronous API calls and long-running batch jobs

## Decision

Adopt a **serverless-first** architecture using AWS Lambda for all compute, DynamoDB for data, S3 for objects, and managed services for orchestration.

## Rationale

| Requirement | Serverless Solution |
|-------------|-------------------|
| Zero idle cost | Lambda: $0 when not invoked; DynamoDB on-demand: $0 when unused |
| Auto-scaling | Lambda: 0 to 1,000+ concurrent; DynamoDB: automatic |
| No ops | No patching, no capacity planning, no server management |
| Event-driven | Native triggers from S3, SQS, EventBridge, DynamoDB Streams |
| Batch processing | Step Functions + Lambda for long workflows (up to 1 year) |
| Pay-per-use | Aligned with SaaS revenue — cost scales with customers |

## Alternatives Considered

| Alternative | Reason Rejected |
|-------------|----------------|
| **ECS Fargate (Containers)** | Higher idle cost (~$30/month minimum); over-engineering at current scale; more operational complexity |
| **EC2 Instances** | Maximum operational overhead; fixed cost regardless of usage; requires patching and scaling configuration |
| **AWS App Runner** | Less AWS service integration; limited event triggers; less mature |
| **Kubernetes (EKS)** | Extreme operational complexity for a small team; significant idle cost; overkill for this workload |

## Consequences

### Positive
- Zero cost when platform has no traffic (critical for startup phase)
- Linear cost growth — cost scales proportionally with revenue
- No infrastructure team required (single developer can operate)
- Built-in high availability (Lambda runs across multiple AZs)
- arm64 (Graviton2) support = 20% cost reduction
- Automatic patching and security updates

### Negative
- Cold starts add latency (mitigated: provisioned concurrency on critical paths)
- 15-minute Lambda timeout (mitigated: Step Functions for long jobs)
- 6MB response payload limit (mitigated: S3 pre-signed URLs for large files)
- No persistent connections (mitigated: DynamoDB — no connection pool needed)
- Harder to debug locally (mitigated: LocalStack, SAM CLI)

### Trade-off Analysis

| Concern | Impact | Mitigation |
|---------|--------|-----------|
| Cold starts | ~500ms on first call | Provisioned concurrency (auth: 5, product API: 10) |
| Execution limit | 15 min max | Step Functions chains for longer workflows |
| Vendor lock-in | AWS-specific | Hexagonal architecture at boundaries |
| Local dev | Different from production | SAM CLI + DynamoDB Local + LocalStack |

---

## References

- AWS Lambda Developer Guide
- AWS Well-Architected Serverless Applications Lens
- MerchOS MERCH-005 (AWS Architecture)
