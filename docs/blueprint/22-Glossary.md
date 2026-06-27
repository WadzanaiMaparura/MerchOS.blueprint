# MerchOS Engineering Blueprint

## Appendix A — Glossary

---

| Field | Value |
|-------|-------|
| **Document ID** | MERCH-GLO |
| **Title** | Glossary |
| **Version** | 0.1 |
| **Status** | Draft |
| **Owner** | Wadzanai Maparura |
| **Technical Lead** | Kiro AI |
| **Created** | 2026-06-27 |
| **Last Updated** | 2026-06-27 |

---

## Terms & Definitions

| Term | Definition |
|------|-----------|
| **ADR** | Architecture Decision Record — formal documentation of a significant technical decision |
| **AI Credit** | Unit of AI usage; 1 credit = 1 full product enrichment on Claude 3.5 Sonnet |
| **Allocation Rules** | Configuration defining how inventory is distributed across marketplaces |
| **ARPU** | Average Revenue Per User — MRR divided by paying customer count |
| **Attribute Extraction** | AI process of identifying structured product attributes from unstructured text |
| **Bedrock** | Amazon Bedrock — managed service for accessing foundation models (LLMs) |
| **Browse Node** | Amazon's category identifier (numeric ID) in their taxonomy tree |
| **Bulk Export** | Generating a marketplace file containing 1,000+ products in a single operation |
| **CAC** | Customer Acquisition Cost — total sales/marketing spend per new customer |
| **CDK** | AWS Cloud Development Kit — Infrastructure as Code using TypeScript |
| **CDK Nag** | Security and best-practice linter for CDK-defined infrastructure |
| **Completeness Score** | Percentage (0–100%) indicating how ready a product is for marketplace export |
| **Confidence Score** | AI-generated value (0.0–1.0) indicating certainty of a prediction or extraction |
| **Correlation ID** | Unique identifier propagated across all services for request tracing |
| **CSV** | Comma-Separated Values — file format for marketplace product uploads |
| **DAU/MAU** | Daily Active Users / Monthly Active Users ratio — engagement metric |
| **Dead Letter Queue (DLQ)** | SQS queue that captures messages that fail processing after max retries |
| **Delta Export** | Export containing only products changed since the last export |
| **DynamoDB** | Amazon DynamoDB — serverless NoSQL database service |
| **EAN-13** | European Article Number — 13-digit product barcode standard |
| **Enrichment** | AI process of adding/improving product data (descriptions, attributes, categories) |
| **EventBridge** | Amazon EventBridge — serverless event bus for inter-service communication |
| **Export Engine** | MerchOS module that generates marketplace-specific export files |
| **Feature Flag** | Configuration toggle enabling/disabling features without code deployment |
| **Flat File** | Amazon's term for their product data upload template (tab-delimited) |
| **GSI** | Global Secondary Index — DynamoDB secondary index for alternative query patterns |
| **Guardrails** | Amazon Bedrock feature for content filtering and safety controls on AI output |
| **Human-in-the-Loop** | Design pattern where AI suggestions require human approval before use |
| **IaC** | Infrastructure as Code — defining cloud resources in version-controlled code |
| **Idempotency Key** | Client-generated ID ensuring duplicate requests produce the same result |
| **IIE** | Image Intelligence Engine — MerchOS module for image analysis and OCR |
| **JWT** | JSON Web Token — signed token used for API authentication |
| **Knowledge Base** | Configuration store for marketplace schemas, rules, and taxonomies |
| **Lambda** | AWS Lambda — serverless compute service (functions-as-a-service) |
| **LLM** | Large Language Model — AI model for text generation and understanding |
| **LTV** | Lifetime Value — predicted total revenue from a customer over their lifetime |
| **Marketplace Intelligence Engine (MIE)** | MerchOS module managing marketplace schemas and validation rules |
| **Middy** | Node.js middleware framework for AWS Lambda functions |
| **MRR** | Monthly Recurring Revenue — predictable monthly subscription income |
| **Multi-Tenant** | Architecture where single infrastructure serves multiple isolated customers |
| **NRR** | Net Revenue Retention — revenue from existing customers including expansion |
| **On-Demand** | DynamoDB billing mode that auto-scales without capacity planning |
| **Partition Key (PK)** | DynamoDB primary key component that determines data distribution |
| **PIE** | Product Intelligence Engine — MerchOS AI module for product enrichment |
| **PITR** | Point-in-Time Recovery — DynamoDB continuous backup feature |
| **PLG** | Product-Led Growth — strategy where the product drives acquisition |
| **POPIA** | Protection of Personal Information Act — South African data protection law |
| **Pre-signed URL** | Temporary S3 URL granting time-limited access to upload/download objects |
| **Provisioned Concurrency** | Lambda feature pre-warming instances to eliminate cold starts |
| **RAG** | Retrieval-Augmented Generation — combining search with LLM generation |
| **RBAC** | Role-Based Access Control — permissions determined by user role |
| **Readiness Score** | Marketplace-specific score (0–100%) for export readiness |
| **Rekognition** | Amazon Rekognition — image analysis service (labels, moderation, text) |
| **SaaS** | Software as a Service — cloud-delivered software on subscription model |
| **Schema** | Marketplace CSV column specification (names, types, rules, limits) |
| **Single-Table Design** | DynamoDB pattern storing all entities in one table with composite keys |
| **SKU** | Stock Keeping Unit — unique identifier for a specific product variant |
| **Sort Key (SK)** | DynamoDB secondary key component enabling range queries within a partition |
| **SP-API** | Amazon Selling Partner API — Amazon's marketplace integration API |
| **SRP** | Secure Remote Password — Cognito authentication protocol |
| **SSE** | Server-Side Encryption — data encrypted at rest by the storage service |
| **Step Functions** | AWS Step Functions — visual workflow orchestration service |
| **Taxonomy** | Hierarchical category classification system used by marketplaces |
| **Tenant** | An isolated customer account in the multi-tenant platform |
| **Textract** | Amazon Textract — OCR and document analysis service |
| **TTL** | Time-To-Live — DynamoDB feature for automatic item expiry |
| **Validation Engine** | Component checking products against marketplace-specific rules |
| **WAF** | Web Application Firewall — API protection against common attacks |
| **X-Ray** | AWS X-Ray — distributed tracing service |
| **Zod** | TypeScript-first schema validation library |

---

*End of Appendix A — Glossary*
