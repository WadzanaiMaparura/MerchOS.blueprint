# ADR-001: Why AWS?

| Field | Value |
|-------|-------|
| **Status** | Approved |
| **Date** | 2026-06-27 |
| **Decision Makers** | Wadzanai Maparura |
| **Category** | Infrastructure |

---

## Context

MerchOS requires a cloud platform that provides:
- Managed AI/ML services (LLM inference without GPU management)
- An Africa-based region for low-latency access to South African users
- Serverless compute and storage that scales to zero
- Deep integration between services (events, queues, orchestration)
- Enterprise-grade security and compliance features
- Mature Infrastructure as Code tooling

## Decision

Use **Amazon Web Services (AWS)** as the sole cloud provider for all MerchOS infrastructure.

## Rationale

| Factor | AWS Advantage |
|--------|--------------|
| **Africa Region** | `af-south-1` (Cape Town) — local latency for SA users |
| **AI Services** | Amazon Bedrock (Claude, Titan), Textract, Rekognition — no GPU management |
| **Serverless** | Lambda, DynamoDB, S3, API Gateway — mature serverless ecosystem |
| **Integration** | EventBridge, Step Functions, SQS — native service-to-service |
| **Security** | IAM, Cognito, KMS, Secrets Manager — defense in depth |
| **IaC** | AWS CDK (TypeScript) — high-level constructs, type safety |
| **Scale** | Proven at massive scale; auto-scaling built in |
| **Ecosystem** | Largest cloud market share; most documentation/community |

## Alternatives Considered

| Alternative | Reason Rejected |
|-------------|----------------|
| **Microsoft Azure** | No South Africa East region equivalent for all needed services; weaker serverless ecosystem |
| **Google Cloud** | No Africa region; less mature AI integration compared to Bedrock |
| **Multi-cloud** | Unnecessary complexity; reduces integration depth; increases cost |
| **Self-hosted** | Operational burden; capital expenditure; doesn't scale to zero |

## Consequences

### Positive
- Single vendor relationship simplifies operations and billing
- Deep service integration reduces glue code
- Cape Town region provides <50ms latency for SA users
- Bedrock eliminates need to manage ML infrastructure
- CDK provides type-safe IaC in the same language as application code

### Negative
- Vendor lock-in risk (mitigated by hexagonal architecture at boundaries)
- Some AI services may require cross-region calls (us-east-1)
- AWS pricing complexity requires ongoing cost monitoring
- Team must develop AWS-specific expertise

### Risk Mitigation
- Hexagonal architecture with ports/adapters at service boundaries
- Data export capabilities built into the platform
- Standard API contracts (REST/GraphQL) at application layer
- Multi-region design documented for future if needed

---

## References

- AWS Well-Architected Framework
- AWS Global Infrastructure — Region availability
- Amazon Bedrock documentation
