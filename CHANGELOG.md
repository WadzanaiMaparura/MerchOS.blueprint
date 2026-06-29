# Changelog

All notable changes to the MerchOS Engineering Blueprint are documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/), and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html) adapted for documentation.

---

## Versioning Strategy

| Version | Meaning | Criteria |
|---------|---------|----------|
| 0.1 | Initial draft | First written version; not yet reviewed |
| 0.2 | Reviewed | Feedback incorporated; diagrams added; ADRs expanded |
| 0.3 | Diagrams complete | All architecture diagrams finalised |
| 0.4 | Mentor/stakeholder review complete | External review incorporated |
| 0.5 | Technical validation | Architecture validated against requirements |
| 1.0 | Approved baseline | Ready for implementation; signed off by stakeholders |
| 1.x | Implementation updates | Changes discovered during development |
| 2.0 | Major revision | Significant architecture change post-implementation |

---

## [0.2] — 2026-06-27

### Added
- **README.md** — Complete project overview with architecture diagram, tech stack, repository structure, blueprint index, status, roadmap, contributing guide, and license
- **docs/diagrams/** — New directory structure for visual architecture documentation
  - `high-level/component-architecture.md` — Internal component structure
  - `high-level/system-context.md` — External system interactions
  - `aws/aws-infrastructure.md` — Complete AWS service topology and account strategy
  - `sequence/product-ingestion-flow.md` — Product upload and AI enrichment pipeline
  - `sequence/ai-processing-pipeline.md` — AI request management and processing
  - `sequence/marketplace-export-workflow.md` — Export generation and validation
  - `data-flow/event-driven-architecture.md` — EventBridge event catalogue and routing
  - `data-flow/ci-cd-pipeline.md` — Deployment pipeline (backend CDK + frontend Amplify)
  - `ui/user-journey.md` — End-to-end seller journey map
- **docs/adr/** — Individual Architecture Decision Records (expanded from blueprint appendix)
  - `ADR-001-why-aws.md` — AWS as sole cloud provider
  - `ADR-002-why-serverless.md` — Serverless-first compute strategy
  - `ADR-003-why-amplify.md` — Amplify over EC2/ECS for frontend hosting
  - `ADR-004-why-bedrock.md` — Bedrock for managed AI inference
  - `ADR-005-why-marketplace-knowledge-base.md` — Configuration-driven marketplace schemas
  - `ADR-006-ai-vs-deterministic-boundary.md` — AI recommends, rules decide
- **docs/well-architected/well-architected-review.md** — AWS Well-Architected Framework mapping across all 6 pillars
- **CHANGELOG.md** — This file; version history tracking

### Changed
- README.md upgraded from placeholder to comprehensive project documentation

---

## [0.1] — 2026-06-27

### Added
- Initial blueprint creation — 24 volumes covering complete platform architecture
- Volumes 01–21: Executive Summary through Implementation Roadmap
- Appendix A: Glossary (Volume 22)
- Appendix B: Architecture Decision Records — 12 ADRs (Volume 23)
- Appendix C: Document Index with cross-reference matrix (Volume 24)
- Placeholder README.md

### Blueprint Statistics (v0.1)
- 21 volumes + 3 appendices
- ~9,500 lines of documentation
- ~45 Mermaid diagrams (embedded in volumes)
- ~350 data tables
- 161 functional requirements
- 129 non-functional requirements
- 12 architecture decision records
- 57 API endpoints documented
- 16 AWS services documented
- 5 marketplace specifications

---

## Upcoming

### [0.3] — Planned
- [ ] Finalise all architecture diagrams (standalone + embedded)
- [ ] Add deployment architecture diagram
- [ ] Add data model entity-relationship diagram
- [ ] Complete marketplace-specific export format diagrams

### [0.4] — Planned
- [ ] Mentor/stakeholder review
- [ ] Incorporate external feedback
- [ ] Validate cost model with AWS pricing calculator
- [ ] Security architecture peer review

### [1.0] — Planned
- [ ] All volumes reviewed and approved
- [ ] Implementation-ready baseline
- [ ] Sprint planning aligned to roadmap
- [ ] Sign-off from all stakeholders
