# MerchOS Engineering Blueprint

## Appendix C — Document Index

---

| Field | Value |
|-------|-------|
| **Document ID** | MERCH-IDX |
| **Title** | Blueprint Document Index |
| **Version** | 0.1 |
| **Status** | Draft |
| **Owner** | Wadzanai Maparura |
| **Technical Lead** | Kiro AI |
| **Created** | 2026-06-27 |
| **Last Updated** | 2026-06-27 |

---

## Complete Document Registry

| Vol | Document ID | File | Title | Pages (est.) | Status |
|-----|-------------|------|-------|--------------|--------|
| 01 | MERCH-001 | 01-Executive-Summary.md | Executive Summary | 825 lines | Draft |
| 02 | MERCH-002 | 02-Business-Product.md | Business & Product | 1,450 lines | Draft |
| 03 | MERCH-003 | 03-Functional-Requirements.md | Functional Requirements | ~450 lines | Draft |
| 04 | MERCH-004 | 04-Non-Functional-Requirements.md | Non-Functional Requirements | ~400 lines | Draft |
| 05 | MERCH-005 | 05-AWS-Architecture.md | AWS Architecture | ~550 lines | Draft |
| 06 | MERCH-006 | 06-Security-Architecture.md | Security Architecture | ~450 lines | Draft |
| 07 | MERCH-007 | 07-AI-Architecture.md | AI Architecture | ~500 lines | Draft |
| 08 | MERCH-008 | 08-Marketplace-Intelligence-Engine.md | Marketplace Intelligence Engine | ~600 lines | Draft |
| 09 | MERCH-009 | 09-Product-Intelligence-Engine.md | Product Intelligence Engine | ~400 lines | Draft |
| 10 | MERCH-010 | 10-Image-Intelligence-Engine.md | Image Intelligence Engine | ~350 lines | Draft |
| 11 | MERCH-011 | 11-Supplier-Intelligence.md | Supplier Intelligence | ~400 lines | Draft |
| 12 | MERCH-012 | 12-Inventory-Engine.md | Inventory Engine | ~380 lines | Draft |
| 13 | MERCH-013 | 13-Export-Engine.md | Export Engine | ~400 lines | Draft |
| 14 | MERCH-014 | 14-Database-Design.md | Database Design | ~400 lines | Draft |
| 15 | MERCH-015 | 15-API-Specifications.md | API Specifications | ~420 lines | Draft |
| 16 | MERCH-016 | 16-Frontend-Architecture.md | Frontend Architecture | ~400 lines | Draft |
| 17 | MERCH-017 | 17-Backend-Architecture.md | Backend Architecture | ~420 lines | Draft |
| 18 | MERCH-018 | 18-DevOps-CICD.md | DevOps & CI/CD | ~400 lines | Draft |
| 19 | MERCH-019 | 19-Monitoring-Operations.md | Monitoring & Operations | ~400 lines | Draft |
| 20 | MERCH-020 | 20-Cost-Optimisation.md | Cost Optimisation | ~380 lines | Draft |
| 21 | MERCH-021 | 21-Implementation-Roadmap.md | Implementation Roadmap | ~380 lines | Draft |
| A | MERCH-GLO | 22-Glossary.md | Glossary | ~120 lines | Draft |
| B | MERCH-ADR | 23-Architecture-Decision-Records.md | Architecture Decision Records | ~200 lines | Draft |
| C | MERCH-IDX | 24-Index.md | Document Index (this file) | ~100 lines | Draft |

---

## Cross-Reference Matrix

| Topic | Primary Volume | Also Referenced In |
|-------|---------------|-------------------|
| Authentication | MERCH-006 (Security) | MERCH-005 (Cognito), MERCH-015 (API Auth), MERCH-016 (Frontend Auth) |
| DynamoDB | MERCH-005 (AWS Arch) | MERCH-014 (Database Design), MERCH-017 (Backend Repos) |
| AI / Bedrock | MERCH-007 (AI Arch) | MERCH-005 (AWS), MERCH-009 (PIE), MERCH-020 (Cost) |
| Marketplace schemas | MERCH-008 (MIE) | MERCH-013 (Export), MERCH-014 (DB), MERCH-009 (PIE - RAG) |
| Export flow | MERCH-013 (Export) | MERCH-008 (Validation), MERCH-012 (Inventory), MERCH-015 (API) |
| Multi-tenancy | MERCH-006 (Security) | MERCH-005 (DDB), MERCH-014 (DB Keys), MERCH-017 (Middleware) |
| Cost management | MERCH-020 (Cost) | MERCH-007 (AI Credits), MERCH-002 (Pricing), MERCH-005 (per-service) |
| Monitoring | MERCH-019 (Monitoring) | MERCH-005 (CloudWatch), MERCH-004 (NFR-Observability) |
| CI/CD | MERCH-018 (DevOps) | MERCH-005 (CDK), MERCH-016 (Amplify), MERCH-019 (Deploy alerts) |
| Product data model | MERCH-014 (Database) | MERCH-003 (Requirements), MERCH-015 (API), MERCH-017 (Repos) |

---

## Blueprint Statistics

| Metric | Value |
|--------|-------|
| Total volumes | 21 + 3 appendices |
| Total estimated lines | ~9,500+ |
| Total Mermaid diagrams | ~45 |
| Total tables | ~350+ |
| Functional requirements | 161 |
| Non-functional requirements | 129 |
| Architecture Decision Records | 12 |
| API endpoints documented | 57 |
| AWS services documented | 16 |
| Marketplace specifications | 5 (complete CSV/API column-level) |

---

## Document Lifecycle

| Status | Meaning | Next Action |
|--------|---------|-------------|
| Draft | Initial version; review pending | Review by owner + stakeholders |
| Review | Under active review | Incorporate feedback |
| Approved | Accepted as authoritative | Reference for implementation |
| Superseded | Replaced by newer version | Archive; reference new version |

---

## How to Use This Blueprint

1. **New team member?** Start with Volume 01 (Executive Summary) for full context, then read Volume 02 (Business & Product) for domain understanding.

2. **Implementing a feature?** Find the relevant engine volume (08–13), then reference the Database Design (14) and API Specifications (15) for data models and contracts.

3. **Making an architecture decision?** Check existing ADRs (Appendix B), then reference the relevant architecture volume (05–07).

4. **Setting up infrastructure?** Follow Volume 05 (AWS Architecture), Volume 18 (DevOps), and Volume 21 (Implementation Roadmap) for sprint-by-sprint guidance.

5. **Reviewing for compliance/security?** Volume 06 (Security Architecture) and Volume 04 (Non-Functional Requirements) are the primary references.

---

*End of Appendix C — Document Index*

*This concludes the MerchOS Engineering Blueprint.*
