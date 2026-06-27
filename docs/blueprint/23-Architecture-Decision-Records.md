# MerchOS Engineering Blueprint

## Appendix B — Architecture Decision Records (ADRs)

---

| Field | Value |
|-------|-------|
| **Document ID** | MERCH-ADR |
| **Title** | Architecture Decision Records |
| **Version** | 0.1 |
| **Status** | Draft |
| **Owner** | Wadzanai Maparura |
| **Technical Lead** | Kiro AI |
| **Created** | 2026-06-27 |
| **Last Updated** | 2026-06-27 |

---

## ADR Index

| ADR | Title | Status | Date |
|-----|-------|--------|------|
| ADR-001 | DynamoDB as primary database | Approved | 2026-06-27 |
| ADR-002 | AWS Lambda for all compute | Approved | 2026-06-27 |
| ADR-003 | Amazon Bedrock for AI inference | Approved | 2026-06-27 |
| ADR-004 | EventBridge for event bus | Approved | 2026-06-27 |
| ADR-005 | Step Functions for orchestration | Approved | 2026-06-27 |
| ADR-006 | Cognito for authentication | Approved | 2026-06-27 |
| ADR-007 | Knowledge-base-driven marketplace schemas | Approved | 2026-06-27 |
| ADR-008 | AI recommends, rules decide | Approved | 2026-06-27 |
| ADR-009 | Next.js on Amplify for frontend | Approved | 2026-06-27 |
| ADR-010 | AWS CDK for Infrastructure as Code | Approved | 2026-06-27 |
| ADR-011 | Multi-tenant via partition key isolation | Approved | 2026-06-27 |
| ADR-012 | S3 for all object storage | Approved | 2026-06-27 |

---

## ADR-001: DynamoDB as Primary Database

**Status:** Approved

**Context:** MerchOS needs a primary database that supports multi-tenant isolation, serverless scaling, single-digit millisecond latency, and no connection pool management.

**Decision:** Use Amazon DynamoDB with single-table design as the primary operational database.

**Consequences:**
- (+) Serverless; scales from zero to millions without capacity planning
- (+) No connection pooling (critical for Lambda)
- (+) Single-digit millisecond latency at any scale
- (+) Multi-tenant isolation via partition key prefixing
- (+) On-demand billing = pay only for what you use
- (-) Requires upfront access pattern analysis (no ad-hoc queries)
- (-) No joins; requires denormalisation
- (-) 400KB item size limit
- (-) Single-table design has learning curve

**Alternatives Rejected:**
- PostgreSQL (Aurora Serverless): connection pool issues with Lambda; higher cost at low scale
- MongoDB Atlas: additional vendor; not AWS-native; connection management
- Amazon OpenSearch: overkill for primary data; better as secondary search index

---

## ADR-002: AWS Lambda for All Compute

**Status:** Approved

**Context:** MerchOS needs a compute platform that scales to zero, handles event-driven workloads, and requires no server management.

**Decision:** Use AWS Lambda for all compute — API handlers, event processors, async workers, and scheduled tasks.

**Consequences:**
- (+) Zero cost when idle; pay per invocation
- (+) Auto-scaling (no capacity planning)
- (+) No server management or patching
- (+) Native integration with all AWS services
- (+) arm64 (Graviton2) for 20% cost reduction
- (-) Cold starts (mitigated by provisioned concurrency for critical paths)
- (-) 15-minute max execution time (mitigated by Step Functions for long workflows)
- (-) 10GB max package size; 6MB response payload (not limiting for our use case)

**Alternatives Rejected:**
- ECS Fargate: over-engineering for current scale; higher idle cost
- EC2: operational overhead; not serverless
- App Runner: less AWS service integration; limited event triggers

---

## ADR-003: Amazon Bedrock for AI Inference

**Status:** Approved

**Context:** MerchOS needs LLM capabilities for product enrichment without hosting or managing ML infrastructure.

**Decision:** Use Amazon Bedrock with Claude 3.5 Sonnet (primary) and Claude 3 Haiku (cost-optimised fallback).

**Consequences:**
- (+) No model hosting or GPU management
- (+) Pay-per-token (no idle infrastructure cost)
- (+) Multiple model choice (Claude, Titan) for fallback
- (+) Guardrails for content safety
- (+) Knowledge Bases for RAG
- (-) Cross-region calls if not available in af-south-1
- (-) Rate limits require queuing and backpressure management
- (-) Token costs are the #1 cost driver; requires careful management

**Alternatives Rejected:**
- Self-hosted models (SageMaker): operational complexity; GPU cost even when idle
- OpenAI API: non-AWS; additional vendor; data residency concerns
- Cohere: smaller model selection; less capable for our use cases

---

## ADR-007: Knowledge-Base-Driven Marketplace Schemas

**Status:** Approved

**Context:** MerchOS must support multiple marketplaces (Takealot, Amazon, Makro, Shopify, WooCommerce) with different CSV formats, validation rules, and category taxonomies. Marketplace formats change periodically.

**Decision:** Store all marketplace schemas, validation rules, and category taxonomies as configuration in DynamoDB/S3. Never hardcode marketplace logic into application code.

**Consequences:**
- (+) Add new marketplaces without code deployment (configuration only)
- (+) Schema changes handled by admin (no developer required)
- (+) Version-controlled schemas with rollback capability
- (+) Schemas consumable by AI (RAG for category recommendation)
- (+) Clear separation: engine logic vs marketplace knowledge
- (-) More complex initial setup (schema definition tooling needed)
- (-) Admin UI required for schema management
- (-) Validation rule expressions need careful design for flexibility

**Alternatives Rejected:**
- Hardcoded per-marketplace adapters: code deployment for every format change; poor scalability
- External marketplace API only: not all marketplaces have APIs; CSV still required
- Plugin system: over-engineering for 5 marketplaces; runtime loading complexity

---

## ADR-008: AI Recommends, Rules Decide

**Status:** Approved

**Context:** AI-generated content (descriptions, attributes, categories) needs to be marketplace-compliant. AI can hallucinate or produce non-compliant output.

**Decision:** Enforce strict separation: AI provides suggestions with confidence scores; deterministic business rules validate, format, and decide what gets exported.

**Consequences:**
- (+) Marketplace compliance guaranteed (rules are deterministic)
- (+) AI failures don't corrupt exports or break business operations
- (+) Users maintain control (human-in-the-loop for approval)
- (+) Auditable (every decision logged with source: AI or rule)
- (+) AI improves over time without risk to production correctness
- (-) Additional UX step (user reviews AI suggestions)
- (-) Not fully autonomous (can't "set and forget")
- (-) Complexity of dual-path (AI generates, rules validate)

**Alternatives Rejected:**
- AI-first (auto-apply all AI output): unacceptable marketplace rejection risk
- Rules-only (no AI): doesn't differentiate product; manual effort too high
- Hybrid auto-approve above confidence threshold: Phase 4 consideration (not Phase 1)

---

## ADR-011: Multi-Tenant via Partition Key Isolation

**Status:** Approved

**Context:** MerchOS is multi-tenant SaaS. Need complete data isolation between tenants while sharing infrastructure for cost efficiency.

**Decision:** Implement tenant isolation via DynamoDB partition key prefixing (`TENANT#{tenantId}`) combined with application-layer middleware enforcement and IAM conditions.

**Consequences:**
- (+) Cost-efficient (shared tables, no per-tenant provisioning)
- (+) Simple operational model (one table to monitor/backup)
- (+) Scalable (DynamoDB handles partition distribution)
- (+) Strong isolation (middleware + IAM conditions + testing)
- (-) Isolation is logical, not physical (defence in depth required)
- (-) Cross-tenant queries require admin-only GSI scan (acceptable)
- (-) Single noisy tenant could affect partition performance (mitigated by DynamoDB auto-splitting)

**Alternatives Rejected:**
- Table-per-tenant (silo): operational nightmare at 1,000+ tenants; CDK stack limits
- Database-per-tenant: excessive cost; complex management
- Pool model with row-level security: DynamoDB has no native row-level security (must be in app layer anyway)

---

*End of Appendix B — Architecture Decision Records*
