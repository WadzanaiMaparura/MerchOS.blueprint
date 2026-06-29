# AWS Well-Architected Framework — MerchOS Review

| Field | Value |
|-------|-------|
| **Document ID** | MERCH-WAF |
| **Title** | Well-Architected Framework Review |
| **Version** | 0.2 |
| **Status** | Draft |
| **Owner** | Wadzanai Maparura |
| **Created** | 2026-06-27 |
| **Last Updated** | 2026-06-27 |

---

## Overview

This document maps MerchOS architecture decisions to the six pillars
of the AWS Well-Architected Framework, demonstrating alignment with
AWS best practices and identifying areas for future improvement.

---

## Pillar 1: Operational Excellence

> *The ability to support development and run workloads effectively,
> gain insight into operations, and continuously improve processes.*

### How MerchOS Achieves This

| Practice | Implementation | Reference |
|----------|---------------|-----------|
| **Infrastructure as Code** | 100% AWS CDK (TypeScript); no console-created resources | MERCH-005 §20 |
| **Automated deployments** | CDK Pipelines (backend) + Amplify CI/CD (frontend) | MERCH-018 |
| **Observability** | CloudWatch Logs + Metrics + X-Ray traces on every Lambda | MERCH-019 |
| **Structured logging** | Lambda Powertools (TypeScript) — JSON structured logs | MERCH-017 |
| **Runbooks** | Documented incident response procedures per service | MERCH-019 |
| **Feature flags** | Controlled rollout; disable features without deployment | MERCH-018 |
| **Small frequent changes** | Continuous deployment; every merge deploys to staging | MERCH-018 |
| **Anticipate failure** | DLQs on all async functions; retry policies; circuit breakers | MERCH-005 |

### Design Decisions Supporting This Pillar

| Decision | Operational Benefit |
|----------|-------------------|
| CDK Nag enabled | Catches configuration drift and security issues pre-deploy |
| Per-function IAM roles | Blast radius limited; easier to audit permissions |
| EventBridge archive | Replay events for debugging and recovery |
| CloudWatch dashboards | Platform health visible at a glance |
| Canary deployments | Detect issues before full traffic shift |

### Gaps / Future Improvements

- [ ] Implement automated rollback on CloudWatch alarm trigger
- [ ] Add synthetic canary monitoring (CloudWatch Synthetics)
- [ ] Establish on-call rotation and escalation procedures
- [ ] Create game days for failure scenario testing

---

## Pillar 2: Security

> *The ability to protect data, systems, and assets while delivering
> business value through risk assessment and mitigation strategies.*

### How MerchOS Achieves This

| Practice | Implementation | Reference |
|----------|---------------|-----------|
| **Identity & access** | Amazon Cognito (MFA, RBAC, tenant isolation) | MERCH-006 |
| **Least privilege** | Per-Lambda IAM roles; no shared service accounts | MERCH-005 §7 |
| **Encryption at rest** | S3 SSE, DynamoDB encryption, Secrets Manager | MERCH-005 |
| **Encryption in transit** | TLS 1.2+ enforced on all endpoints | MERCH-006 |
| **Multi-tenant isolation** | Partition key prefixing + IAM conditions + middleware | MERCH-006, ADR-011 |
| **API protection** | AWS WAF + API Gateway rate limiting + request validation | MERCH-005 §9 |
| **Secrets management** | AWS Secrets Manager with rotation policies | MERCH-005 §21 |
| **Audit trail** | CloudTrail + DynamoDB audit table + structured logs | MERCH-006 |
| **Compliance** | POPIA (South Africa) — data residency, consent, deletion | MERCH-006 |
| **Dependency scanning** | CDK Nag (AwsSolutions pack); npm audit in CI | MERCH-018 |

### Threat Model Summary

| Threat | Control | AWS Service |
|--------|---------|-------------|
| Unauthorised access | MFA + JWT validation + session management | Cognito |
| Cross-tenant leakage | Partition isolation + auth middleware + IAM conditions | DynamoDB + Lambda |
| API abuse | Rate limiting + WAF + throttling | API Gateway + WAF |
| Data exfiltration | Encryption + access logging + anomaly detection | S3 + CloudTrail |
| Credential exposure | Secrets Manager + no hardcoded credentials + rotation | Secrets Manager |

### Gaps / Future Improvements

- [ ] Implement AWS GuardDuty for threat detection
- [ ] Add penetration testing programme (annual)
- [ ] Implement AWS Config rules for continuous compliance
- [ ] Add VPC endpoint policies if VPC is ever introduced

---

## Pillar 3: Reliability

> *The ability of a workload to perform its intended function correctly
> and consistently when expected to.*

### How MerchOS Achieves This

| Practice | Implementation | Reference |
|----------|---------------|-----------|
| **Multi-AZ by default** | DynamoDB, Lambda, S3 — all natively multi-AZ | MERCH-005 |
| **Retry with backoff** | Exponential backoff on all service calls (2x, max 60s) | MERCH-005 §11 |
| **Dead-letter queues** | Every async Lambda + SQS queue has a DLQ | MERCH-005 §7 |
| **Circuit breakers** | Graceful degradation when downstream services fail | MERCH-017 |
| **Idempotency** | Idempotency keys on all write operations | MERCH-017 |
| **Point-in-time recovery** | DynamoDB PITR enabled (35-day window) | MERCH-005 §8 |
| **S3 versioning** | Object recovery from accidental deletion/overwrite | MERCH-005 §6 |
| **Step Functions retry** | Built-in retry + catch states in all workflows | MERCH-005 §11 |
| **Event replay** | EventBridge archive enables event reprocessing | MERCH-005 §12 |
| **Health checks** | API Gateway health endpoint; CloudWatch synthetic | MERCH-019 |

### Availability Target

| Metric | Target | Achieved By |
|--------|--------|-------------|
| Platform availability | 99.9% (8.7h downtime/year) | Multi-AZ serverless + retry + DLQ |
| API latency (p95) | < 500ms | Provisioned concurrency + DynamoDB |
| Recovery time (MTTR) | < 30 minutes | Automated alerting + runbooks |
| Data durability | 99.999999999% (11 9s) | S3 + DynamoDB multi-AZ replication |

### Gaps / Future Improvements

- [ ] Implement cross-region DR (af-south-1 → eu-west-1)
- [ ] Add chaos engineering (AWS Fault Injection Simulator)
- [ ] Load test to validate scaling limits
- [ ] Implement bulkhead pattern for tenant isolation under load

---

## Pillar 4: Performance Efficiency

> *The ability to use computing resources efficiently to meet system
> requirements and maintain that efficiency as demand changes.*

### How MerchOS Achieves This

| Practice | Implementation | Reference |
|----------|---------------|-----------|
| **Right-sized compute** | Lambda memory benchmarked per function (Power Tuning) | MERCH-005 §7 |
| **arm64 architecture** | Graviton2 — 20% cheaper, 10–20% faster | MERCH-005 §7 |
| **On-demand scaling** | Lambda concurrent scaling; DynamoDB on-demand | MERCH-005 |
| **Async processing** | Long operations processed via SQS/Step Functions | MERCH-005 §11–13 |
| **CDN caching** | CloudFront (via Amplify) for static assets | MERCH-005 §19 |
| **Minimal cold starts** | esbuild tree-shaking; provisioned concurrency on critical paths | MERCH-005 §7 |
| **Efficient data access** | DynamoDB single-table design; GSIs for access patterns | MERCH-014 |
| **Prompt optimisation** | AI prompts reviewed for token efficiency; caching | MERCH-007 §7 |
| **Batch processing** | SQS batch size tuning; parallel Lambda invocation | MERCH-005 §13 |

### Performance Targets

| Operation | Target (p95) | Strategy |
|-----------|-------------|----------|
| API response (sync) | < 500ms | Provisioned concurrency + optimised Lambda |
| Product enrichment (AI) | < 15s | Direct Bedrock inference |
| Bulk export (100 products) | < 10min | Parallel Step Functions |
| Image processing | < 5s | Rekognition sync API |
| Database query | < 20ms | DynamoDB single-digit ms latency |

### Gaps / Future Improvements

- [ ] Implement response caching (API Gateway cache or ElastiCache)
- [ ] Add Lambda Power Tuning automated runs in CI
- [ ] Evaluate DynamoDB Accelerator (DAX) for hot access patterns
- [ ] Implement request deduplication for AI calls

---

## Pillar 5: Cost Optimisation

> *The ability to run systems to deliver business value at the lowest
> price point.*

### How MerchOS Achieves This

| Practice | Implementation | Reference |
|----------|---------------|-----------|
| **Pay-per-use** | Lambda, DynamoDB on-demand, S3 — zero idle cost | MERCH-020 |
| **Serverless-first** | No servers running when platform has no traffic | ADR-002 |
| **arm64 compute** | 20% cost reduction vs. x86 Lambda | MERCH-005 §7 |
| **S3 lifecycle policies** | Auto-transition to cheaper tiers (IA, Glacier) | MERCH-005 §6 |
| **AI cost control** | Per-tenant token budgets; model routing by tier | MERCH-007 §11 |
| **Right-sized memory** | Each Lambda function memory-optimised | MERCH-005 §7 |
| **TTL on DynamoDB** | Auto-delete expired records (sessions, temp data) | MERCH-005 §8 |
| **Cost alerts** | AWS Budgets + CloudWatch billing alarms | MERCH-020 |
| **Prompt caching** | Cache identical AI prompts to avoid duplicate costs | MERCH-007 §7 |
| **Intelligent-Tiering** | S3 auto-optimises storage class based on access | MERCH-005 §6 |

### Cost Model

| Scale | Products | Estimated Monthly AWS Cost | Per Product |
|-------|----------|---------------------------|-------------|
| Startup | 1,000 | $50–$150 | $0.05–$0.15 |
| Growth | 10,000 | $200–$600 | $0.02–$0.06 |
| Scale | 100,000 | $1,500–$4,000 | $0.015–$0.04 |
| Enterprise | 1,000,000 | $8,000–$20,000 | $0.008–$0.02 |

### Cost Drivers (Ranked)

| # | Service | % of Total | Optimisation |
|---|---------|-----------|--------------|
| 1 | Amazon Bedrock (AI) | 30–40% | Model routing; caching; budget caps |
| 2 | AWS Lambda | 20–25% | arm64; right-sized; batch processing |
| 3 | Amazon DynamoDB | 15–20% | On-demand; TTL; efficient access patterns |
| 4 | Amazon S3 | 10–15% | Lifecycle policies; Intelligent-Tiering |
| 5 | Other (API GW, SQS, etc.) | 10–15% | Usage-based; minimal optimisation needed |

### Gaps / Future Improvements

- [ ] Implement Savings Plans for predictable Lambda usage at scale
- [ ] Add per-tenant cost attribution dashboard
- [ ] Evaluate DynamoDB provisioned mode at steady-state scale
- [ ] Implement AI response caching to reduce Bedrock costs by ~30%

---

## Pillar 6: Sustainability

> *Minimising the environmental impacts of running cloud workloads.*

### How MerchOS Achieves This

| Practice | Implementation | Reference |
|----------|---------------|-----------|
| **Scale to zero** | Serverless architecture consumes no resources when idle | ADR-002 |
| **Efficient compute** | arm64 (Graviton2) — more efficient per watt | MERCH-005 §7 |
| **Data lifecycle** | Auto-delete unused data (TTL, lifecycle policies) | MERCH-005 §6 |
| **Right-sized resources** | No over-provisioned infrastructure; on-demand only | MERCH-020 |
| **Managed services** | AWS optimises underlying infrastructure efficiency | All services |
| **Minimal data transfer** | VPC-less architecture; no NAT Gateway; local region | MERCH-005 §4 |
| **Efficient AI usage** | Prompt caching; model routing; avoid unnecessary inference | MERCH-007 §11 |
| **Code efficiency** | esbuild tree-shaking; minimal dependencies; fast execution | MERCH-017 |

### Sustainability Decisions

| Decision | Environmental Benefit |
|----------|---------------------|
| VPC-less architecture | No NAT Gateway running 24/7 consuming resources |
| On-demand DynamoDB | No provisioned capacity sitting idle |
| S3 Intelligent-Tiering | Data automatically moved to lowest-energy storage |
| Lambda arm64 | Graviton2 processors are more energy-efficient |
| Event-driven processing | Work happens only when needed; no polling loops |

### Gaps / Future Improvements

- [ ] Track carbon footprint via AWS Customer Carbon Footprint Tool
- [ ] Evaluate S3 Glacier Deep Archive for old exports/backups
- [ ] Measure and report per-request energy consumption
- [ ] Consider renewable energy regions for cross-region AI calls

---

## Summary Matrix

| Pillar | Score | Key Strength | Key Gap |
|--------|-------|-------------|---------|
| Operational Excellence | Strong | 100% IaC + full observability | Automated rollback |
| Security | Strong | Multi-layer tenant isolation + encryption | Penetration testing |
| Reliability | Strong | Multi-AZ + DLQ + retry + PITR | Cross-region DR |
| Performance Efficiency | Strong | Serverless scaling + right-sized compute | Response caching layer |
| Cost Optimisation | Strong | Pay-per-use + zero idle cost | Per-tenant attribution |
| Sustainability | Good | Scale-to-zero + arm64 + lifecycle policies | Carbon tracking |

---

## Review Schedule

| Review | Frequency | Scope |
|--------|-----------|-------|
| Architecture review | Quarterly | All 6 pillars |
| Security review | Monthly | Pillar 2 focus + vulnerability scan |
| Cost review | Weekly | Pillar 5 + billing analysis |
| Performance review | Monthly | Pillar 4 + latency metrics |
| Reliability review | After incidents | Pillar 3 + post-mortem |

---

## References

- [AWS Well-Architected Framework](https://docs.aws.amazon.com/wellarchitected/latest/framework/welcome.html)
- [Serverless Applications Lens](https://docs.aws.amazon.com/wellarchitected/latest/serverless-applications-lens/welcome.html)
- [Machine Learning Lens](https://docs.aws.amazon.com/wellarchitected/latest/machine-learning-lens/machine-learning-lens.html)
- [SaaS Lens](https://docs.aws.amazon.com/wellarchitected/latest/saas-lens/saas-lens.html)

---

*End of Well-Architected Framework Review*
