# MerchOS Engineering Blueprint

## Volume 03 — Functional Requirements

---

| Field | Value |
|-------|-------|
| **Document ID** | MERCH-003 |
| **Title** | Functional Requirements |
| **Version** | 0.1 |
| **Status** | Draft |
| **Owner** | Wadzanai Maparura |
| **Technical Lead** | Kiro AI |
| **Created** | 2026-06-27 |
| **Last Updated** | 2026-06-27 |
| **Next Review** | 2026-07-11 |
| **Classification** | Internal — Confidential |
| **Related Documents** | MERCH-002 (Business & Product), MERCH-014 (Database Design), MERCH-015 (API Specifications) |

---

## Revision History

| Version | Date | Author | Change Description |
|---------|------|--------|-------------------|
| 0.1 | 2026-06-27 | Kiro AI / Wadzanai Maparura | Initial draft — complete functional requirements |

---

## Table of Contents

1. [Purpose](#1-purpose)
2. [Scope](#2-scope)
3. [Requirements Notation](#3-requirements-notation)
4. [Authentication & Authorisation](#4-authentication--authorisation)
5. [Tenant Management](#5-tenant-management)
6. [Product Management](#6-product-management)
7. [AI Intelligence](#7-ai-intelligence)
8. [Marketplace Intelligence](#8-marketplace-intelligence)
9. [Export Engine](#9-export-engine)
10. [Inventory Management](#10-inventory-management)
11. [Supplier Intelligence](#11-supplier-intelligence)
12. [Image Management](#12-image-management)
13. [Notifications & Alerts](#13-notifications--alerts)
14. [Analytics & Reporting](#14-analytics--reporting)
15. [Administration](#15-administration)
16. [Assumptions](#16-assumptions)
17. [Dependencies](#17-dependencies)
18. [References](#18-references)

---


## 1. Purpose

This document defines all **functional requirements** for the MerchOS platform. Each requirement specifies observable system behaviour — what the system must do, not how it does it.

These requirements serve as the contract between:
- Product (what users need)
- Engineering (what to build)
- QA (what to test)
- Stakeholders (what to expect)

Every requirement is traceable to user stories in MERCH-002 and informs API specifications in MERCH-015 and database models in MERCH-014.

---

## 2. Scope

### In Scope
- All user-facing platform functionality
- System-initiated automated behaviours
- Integration points with external systems
- Administrative capabilities

### Out of Scope
- Non-functional requirements (see MERCH-004)
- Implementation details (see MERCH-005, MERCH-017)
- UI/UX specifications (see MERCH-016)

---

## 3. Requirements Notation

| Prefix | Module | Example |
|--------|--------|---------|
| FR-AUTH | Authentication & Authorisation | FR-AUTH-001 |
| FR-TEN | Tenant Management | FR-TEN-001 |
| FR-PRD | Product Management | FR-PRD-001 |
| FR-AI | AI Intelligence | FR-AI-001 |
| FR-MKT | Marketplace Intelligence | FR-MKT-001 |
| FR-EXP | Export Engine | FR-EXP-001 |
| FR-INV | Inventory Management | FR-INV-001 |
| FR-SUP | Supplier Intelligence | FR-SUP-001 |
| FR-IMG | Image Management | FR-IMG-001 |
| FR-NTF | Notifications & Alerts | FR-NTF-001 |
| FR-ANL | Analytics & Reporting | FR-ANL-001 |
| FR-ADM | Administration | FR-ADM-001 |

**Priority Levels:**
- **P0** — Must have for MVP (Phase 1)
- **P1** — Required for Phase 2
- **P2** — Required for Phase 3
- **P3** — Future enhancement

---


## 4. Authentication & Authorisation

### 4.1 User Registration

| ID | Requirement | Priority | Acceptance Criteria |
|----|------------|----------|-------------------|
| FR-AUTH-001 | System SHALL allow users to register with email and password | P0 | User receives verification email; account created in pending state |
| FR-AUTH-002 | System SHALL verify email addresses before activating accounts | P0 | Unverified accounts cannot access platform features |
| FR-AUTH-003 | System SHALL enforce password policy (min 8 chars, 1 upper, 1 number, 1 special) | P0 | Weak passwords rejected with specific feedback |
| FR-AUTH-004 | System SHALL support social login (Google, Microsoft) | P1 | OAuth 2.0 flow completes; user profile populated from provider |
| FR-AUTH-005 | System SHALL associate every new user with a tenant on registration | P0 | Tenant context established before first platform interaction |

### 4.2 Authentication

| ID | Requirement | Priority | Acceptance Criteria |
|----|------------|----------|-------------------|
| FR-AUTH-010 | System SHALL authenticate users via email/password | P0 | Valid credentials return JWT access + refresh tokens |
| FR-AUTH-011 | System SHALL support MFA (TOTP and SMS) | P0 | MFA challenge presented after password verification |
| FR-AUTH-012 | System SHALL issue short-lived access tokens (1 hour) | P0 | Tokens expire; API returns 401 after expiry |
| FR-AUTH-013 | System SHALL support token refresh without re-authentication | P0 | Refresh token (30 days) exchanges for new access token |
| FR-AUTH-014 | System SHALL lock accounts after 5 failed login attempts | P0 | Account locked for 30 minutes; user notified via email |
| FR-AUTH-015 | System SHALL support password reset via email link | P0 | Reset link valid for 1 hour; single use only |
| FR-AUTH-016 | System SHALL log all authentication events | P0 | Login, logout, failed attempts, MFA events recorded with timestamp and IP |

### 4.3 Authorisation (RBAC)

| ID | Requirement | Priority | Acceptance Criteria |
|----|------------|----------|-------------------|
| FR-AUTH-020 | System SHALL enforce role-based access control | P0 | Users can only access resources permitted by their role |
| FR-AUTH-021 | System SHALL support predefined roles: Owner, Admin, Manager, Editor, Viewer | P0 | Each role has distinct permission set |
| FR-AUTH-022 | System SHALL enforce tenant isolation — users cannot access other tenants' data | P0 | Cross-tenant request returns 403; no data leakage |
| FR-AUTH-023 | System SHALL allow Owners to invite users to their tenant | P0 | Invitation email sent; invitee joins correct tenant on registration |
| FR-AUTH-024 | System SHALL allow Owners to assign and modify user roles | P0 | Role changes take effect immediately on next API call |
| FR-AUTH-025 | System SHALL support custom permission sets (Enterprise tier) | P2 | Admins can create custom roles with granular permissions |

### 4.4 Role Permission Matrix

| Permission | Owner | Admin | Manager | Editor | Viewer |
|-----------|:-----:|:-----:|:-------:|:------:|:------:|
| Manage tenant settings | ✓ | ✓ | — | — | — |
| Manage users & roles | ✓ | ✓ | — | — | — |
| Manage billing | ✓ | — | — | — | — |
| Create/edit products | ✓ | ✓ | ✓ | ✓ | — |
| Delete products | ✓ | ✓ | ✓ | — | — |
| Generate exports | ✓ | ✓ | ✓ | ✓ | — |
| Approve AI suggestions | ✓ | ✓ | ✓ | ✓ | — |
| View products | ✓ | ✓ | ✓ | ✓ | ✓ |
| View analytics | ✓ | ✓ | ✓ | ✓ | ✓ |
| Manage inventory | ✓ | ✓ | ✓ | — | — |
| Manage suppliers | ✓ | ✓ | ✓ | — | — |
| Configure marketplace connections | ✓ | ✓ | — | — | — |
| View audit logs | ✓ | ✓ | — | — | — |

---


## 5. Tenant Management

| ID | Requirement | Priority | Acceptance Criteria |
|----|------------|----------|-------------------|
| FR-TEN-001 | System SHALL create a tenant on first user registration | P0 | Tenant record created with unique ID; user assigned as Owner |
| FR-TEN-002 | System SHALL isolate all tenant data using partition key prefixing | P0 | No API call returns data belonging to another tenant |
| FR-TEN-003 | System SHALL support tenant subscription tier (Starter/Growth/Professional/Enterprise) | P0 | Tier determines feature limits and quotas |
| FR-TEN-004 | System SHALL enforce tier-based quotas (products, users, storage, AI credits) | P0 | Quota exceeded returns 429 with upgrade prompt |
| FR-TEN-005 | System SHALL track tenant usage metrics in real-time | P0 | Current usage queryable via API; updated within 5 minutes |
| FR-TEN-006 | System SHALL support tenant suspension (non-payment, abuse) | P1 | Suspended tenants get read-only access; exports blocked |
| FR-TEN-007 | System SHALL support tenant deletion with full data purge (POPIA compliance) | P1 | All data removed within 30 days; confirmation to tenant owner |
| FR-TEN-008 | System SHALL support agency-mode tenants (parent-child relationships) | P2 | Agency tenant can manage multiple child tenants with isolation |
| FR-TEN-009 | System SHALL maintain tenant configuration (connected marketplaces, preferences) | P0 | Configuration persisted and applied to all tenant operations |
| FR-TEN-010 | System SHALL provide tenant onboarding wizard for initial setup | P0 | Guided flow: marketplace selection → first product → first export |

---

## 6. Product Management

### 6.1 Product CRUD

| ID | Requirement | Priority | Acceptance Criteria |
|----|------------|----------|-------------------|
| FR-PRD-001 | System SHALL allow creation of products with title, description, and category | P0 | Product created with unique ID; timestamps set; status = draft |
| FR-PRD-002 | System SHALL support product attributes (key-value pairs, typed) | P0 | Attributes stored with type (text, number, boolean, list); retrievable |
| FR-PRD-003 | System SHALL support product variants (size, colour, material, etc.) | P0 | Variants inherit parent attributes; override specific fields |
| FR-PRD-004 | System SHALL support product status lifecycle: Draft → Active → Archived | P0 | Status transitions validated; archived products excluded from exports |
| FR-PRD-005 | System SHALL auto-generate internal SKU if not provided | P0 | SKU format: {TENANT-PREFIX}-{CATEGORY-CODE}-{SEQUENCE} |
| FR-PRD-006 | System SHALL validate required fields on product creation/update | P0 | Missing required fields return 400 with field-level errors |
| FR-PRD-007 | System SHALL support product duplication (clone with modifications) | P1 | Cloned product gets new ID/SKU; all attributes copied |
| FR-PRD-008 | System SHALL support soft-delete with recovery window (30 days) | P1 | Deleted products recoverable; permanent purge after 30 days |
| FR-PRD-009 | System SHALL calculate and maintain product completeness score | P1 | Score (0–100%) based on filled fields vs. marketplace requirements |
| FR-PRD-010 | System SHALL support product tagging and custom grouping | P1 | Tags are free-form; groups are named collections |

### 6.2 Bulk Operations

| ID | Requirement | Priority | Acceptance Criteria |
|----|------------|----------|-------------------|
| FR-PRD-020 | System SHALL support bulk product import via CSV upload | P0 | CSV parsed; products created; import report generated with success/error counts |
| FR-PRD-021 | System SHALL support bulk product import via Excel upload | P1 | XLSX parsed; multi-sheet support; same outcomes as CSV |
| FR-PRD-022 | System SHALL validate bulk imports against schema before processing | P0 | Validation errors returned per-row before any products created |
| FR-PRD-023 | System SHALL support bulk update of existing products | P1 | Matching by SKU or product ID; only changed fields updated |
| FR-PRD-024 | System SHALL process bulk operations asynchronously for > 100 items | P0 | Job ID returned immediately; status queryable; notification on completion |
| FR-PRD-025 | System SHALL support import template download (pre-formatted CSV/Excel) | P0 | Template includes all supported columns with header descriptions |
| FR-PRD-026 | System SHALL handle bulk imports of up to 50,000 products per job | P1 | Large imports chunked; progress tracking; resume on failure |

### 6.3 Product Search & Filtering

| ID | Requirement | Priority | Acceptance Criteria |
|----|------------|----------|-------------------|
| FR-PRD-030 | System SHALL support full-text search across product title and description | P0 | Results returned in < 500ms; ranked by relevance |
| FR-PRD-031 | System SHALL support filtering by status, category, tags, and completeness score | P0 | Filters combinable (AND logic); results paginated |
| FR-PRD-032 | System SHALL support sorting by date created, date modified, title, completeness | P0 | Ascending and descending; default = most recently modified |
| FR-PRD-033 | System SHALL support pagination with cursor-based navigation | P0 | Page size configurable (10–100); next/previous cursors returned |
| FR-PRD-034 | System SHALL support saved search queries (saved filters) | P2 | Named filter sets persisted per user; quick-access |

---


## 7. AI Intelligence

### 7.1 Description Generation

| ID | Requirement | Priority | Acceptance Criteria |
|----|------------|----------|-------------------|
| FR-AI-001 | System SHALL generate marketplace-ready product descriptions from raw product data | P0 | Description generated within 10 seconds; confidence score included |
| FR-AI-002 | System SHALL support tone/style configuration (professional, casual, technical) | P1 | Generated description matches selected tone; configurable per tenant |
| FR-AI-003 | System SHALL respect marketplace character limits in generated descriptions | P0 | Output truncated/adapted to target marketplace constraints |
| FR-AI-004 | System SHALL generate SEO-optimised descriptions with keyword integration | P1 | Keywords identified and naturally incorporated; SEO score provided |
| FR-AI-005 | System SHALL support description generation in multiple languages | P2 | English, Afrikaans, Zulu initially; language quality validated |
| FR-AI-006 | System SHALL present AI descriptions as suggestions requiring user approval | P0 | Description stored as draft; user must accept/edit/reject before use |

### 7.2 Attribute Extraction

| ID | Requirement | Priority | Acceptance Criteria |
|----|------------|----------|-------------------|
| FR-AI-010 | System SHALL extract structured attributes from unstructured text (supplier descriptions) | P0 | Attributes extracted with confidence scores; mapped to product schema |
| FR-AI-011 | System SHALL extract attributes from product images via OCR | P1 | Text extracted from images; attributes parsed from extracted text |
| FR-AI-012 | System SHALL extract attributes from uploaded PDF documents | P1 | PDF parsed; tables and text extracted; attributes mapped |
| FR-AI-013 | System SHALL provide confidence scores (0.0–1.0) for all extracted attributes | P0 | Scores visible to user; low-confidence (< 0.7) flagged for review |
| FR-AI-014 | System SHALL learn from user corrections to improve future extractions | P2 | Correction patterns tracked; model context updated per tenant |

### 7.3 Category Recommendation

| ID | Requirement | Priority | Acceptance Criteria |
|----|------------|----------|-------------------|
| FR-AI-020 | System SHALL recommend marketplace categories based on product attributes | P1 | Top 3 recommendations with confidence scores; correct category in top 3 > 80% of time |
| FR-AI-021 | System SHALL use marketplace taxonomy from Knowledge Base for recommendations | P1 | Recommendations constrained to valid taxonomy nodes |
| FR-AI-022 | System SHALL explain category recommendation rationale to the user | P2 | Brief explanation of why category was selected shown alongside recommendation |

### 7.4 Batch AI Processing

| ID | Requirement | Priority | Acceptance Criteria |
|----|------------|----------|-------------------|
| FR-AI-030 | System SHALL support batch AI enrichment for multiple products | P1 | Batch job created; products processed sequentially/parallel; report generated |
| FR-AI-031 | System SHALL track AI credit consumption per operation | P0 | Credits deducted per AI call; running total available via API |
| FR-AI-032 | System SHALL halt processing when AI credit budget is exhausted | P0 | Processing stops gracefully; user notified; partial results preserved |
| FR-AI-033 | System SHALL queue batch jobs and process based on tenant priority/tier | P1 | Enterprise > Professional > Growth > Starter processing priority |

---

## 8. Marketplace Intelligence

### 8.1 Knowledge Base Management

| ID | Requirement | Priority | Acceptance Criteria |
|----|------------|----------|-------------------|
| FR-MKT-001 | System SHALL store marketplace schemas as configuration (not code) | P0 | Schema changes require no code deployment |
| FR-MKT-002 | System SHALL version marketplace schemas with change history | P0 | Previous schema versions queryable; rollback supported |
| FR-MKT-003 | System SHALL store per-column metadata: name, type, required, char limit, validation rules, accepted values | P0 | All CSV columns fully documented in knowledge base |
| FR-MKT-004 | System SHALL store marketplace category taxonomies as navigable trees | P0 | Categories queryable by level, parent, search; full path retrievable |
| FR-MKT-005 | System SHALL store marketplace image requirements (dimensions, format, count, size) | P0 | Image validation rules configurable per marketplace |
| FR-MKT-006 | System SHALL support marketplace-specific attribute mappings (product field → CSV column) | P0 | Mappings configurable; support 1:1, 1:many, many:1 relationships |
| FR-MKT-007 | System SHALL notify admins when marketplace schema changes are detected | P1 | Alert triggered; affected exports flagged; update workflow initiated |

### 8.2 Supported Marketplaces

| ID | Requirement | Priority | Acceptance Criteria |
|----|------------|----------|-------------------|
| FR-MKT-010 | System SHALL support Takealot marketplace (CSV + API) | P0 | Valid Takealot CSV exportable; all required columns populated |
| FR-MKT-011 | System SHALL support Amazon marketplace (SP-API flat files) | P0 | Valid Amazon flat file exportable; compliant with SP-API schemas |
| FR-MKT-012 | System SHALL support Makro marketplace (CSV) | P1 | Valid Makro CSV exportable; all required columns populated |
| FR-MKT-013 | System SHALL support Shopify (Admin API product creation/sync) | P1 | Products pushable to Shopify; bidirectional sync supported |
| FR-MKT-014 | System SHALL support WooCommerce (REST API product creation/sync) | P2 | Products pushable to WooCommerce; bidirectional sync supported |
| FR-MKT-015 | System SHALL allow adding new marketplace support via configuration only | P1 | New marketplace added by populating knowledge base — zero code deployment |

### 8.3 Validation

| ID | Requirement | Priority | Acceptance Criteria |
|----|------------|----------|-------------------|
| FR-MKT-020 | System SHALL validate products against marketplace-specific rules before export | P0 | Validation report with pass/fail per field; specific error messages |
| FR-MKT-021 | System SHALL check required fields, data types, character limits, and accepted values | P0 | All rule types enforced; clear error messages per violation |
| FR-MKT-022 | System SHALL provide a pre-export validation preview (dry-run) | P0 | User can validate without generating export file |
| FR-MKT-023 | System SHALL calculate marketplace readiness score per product (0–100%) | P1 | Score reflects % of required/recommended fields populated correctly |
| FR-MKT-024 | System SHALL support custom validation rules (regex, cross-field, conditional) | P1 | Admin can define rules via configuration; rules evaluated at validation time |

---


## 9. Export Engine

### 9.1 Export Generation

| ID | Requirement | Priority | Acceptance Criteria |
|----|------------|----------|-------------------|
| FR-EXP-001 | System SHALL generate marketplace-specific CSV files from product data | P0 | CSV conforms to marketplace schema; downloadable via signed URL |
| FR-EXP-002 | System SHALL generate Amazon SP-API compliant flat files | P0 | Flat file passes Amazon validation; correct template used per category |
| FR-EXP-003 | System SHALL support multi-product export (batch export) | P0 | Selected products exported in single file; up to 50,000 per batch |
| FR-EXP-004 | System SHALL validate all products in an export before file generation | P0 | Invalid products excluded; validation report included with export |
| FR-EXP-005 | System SHALL support export to multiple marketplaces from single selection | P1 | One action generates separate files per marketplace |
| FR-EXP-006 | System SHALL apply marketplace attribute mappings during export | P0 | Product fields correctly mapped to marketplace CSV columns |
| FR-EXP-007 | System SHALL handle character encoding correctly (UTF-8, with BOM where required) | P0 | No encoding errors on marketplace upload |
| FR-EXP-008 | System SHALL include only products matching export filter criteria | P0 | Filters: status, category, tags, marketplace readiness score threshold |

### 9.2 Export Management

| ID | Requirement | Priority | Acceptance Criteria |
|----|------------|----------|-------------------|
| FR-EXP-010 | System SHALL maintain export history with downloadable files | P0 | Previous exports listed with date, marketplace, product count, status |
| FR-EXP-011 | System SHALL provide export status tracking (pending, processing, complete, failed) | P0 | Real-time status via API; webhook notification on completion |
| FR-EXP-012 | System SHALL support scheduled recurring exports | P1 | Cron-based scheduling; export only changed products since last export |
| FR-EXP-013 | System SHALL detect product changes since last export (delta exports) | P1 | Only modified products included; reduces marketplace processing load |
| FR-EXP-014 | System SHALL support export preview (first 10 rows rendered in UI) | P1 | User can inspect export format before committing to full generation |
| FR-EXP-015 | System SHALL retain export files for 90 days (configurable) | P0 | Files auto-deleted after retention period; lifecycle policy enforced |

### 9.3 Direct API Push

| ID | Requirement | Priority | Acceptance Criteria |
|----|------------|----------|-------------------|
| FR-EXP-020 | System SHALL support direct product push to Shopify via Admin API | P1 | Product created/updated in Shopify; sync status tracked |
| FR-EXP-021 | System SHALL support direct product push to WooCommerce via REST API | P2 | Product created/updated in WooCommerce; sync status tracked |
| FR-EXP-022 | System SHALL support direct product push to Amazon via SP-API | P2 | Listing created via Listings API; feed submission tracked |
| FR-EXP-023 | System SHALL handle API rate limits with exponential backoff | P1 | Rate-limited requests retried; no data loss; progress maintained |
| FR-EXP-024 | System SHALL track sync status per product per marketplace | P1 | Status: not synced, syncing, synced, sync failed, out of date |

---

## 10. Inventory Management

| ID | Requirement | Priority | Acceptance Criteria |
|----|------------|----------|-------------------|
| FR-INV-001 | System SHALL track stock quantity per product variant | P0 | Stock level queryable; updated in real-time on adjustment |
| FR-INV-002 | System SHALL support stock adjustments (add, remove, set) with reason codes | P0 | Adjustment recorded with quantity, reason, timestamp, user |
| FR-INV-003 | System SHALL trigger low-stock alerts when quantity falls below threshold | P1 | Alert sent via notification; threshold configurable per product |
| FR-INV-004 | System SHALL support multiple warehouse locations | P2 | Stock tracked per location; total available calculated |
| FR-INV-005 | System SHALL support inventory allocation rules per marketplace | P2 | Allocation policies define % or fixed quantity per channel |
| FR-INV-006 | System SHALL prevent overselling by checking stock before export/push | P1 | Export excludes products with zero stock (configurable) |
| FR-INV-007 | System SHALL support bulk stock updates via CSV import | P1 | CSV with SKU + quantity; adjustments applied in batch |
| FR-INV-008 | System SHALL maintain complete stock movement history (audit trail) | P1 | All adjustments queryable by product, date range, reason |
| FR-INV-009 | System SHALL sync inventory levels to connected marketplaces | P2 | Stock changes propagated within configured interval (default 5 min) |
| FR-INV-010 | System SHALL support stock reservation during export/push process | P2 | Reserved stock not available for other channels until released |

---

## 11. Supplier Intelligence

| ID | Requirement | Priority | Acceptance Criteria |
|----|------------|----------|-------------------|
| FR-SUP-001 | System SHALL allow registration of supplier profiles (name, contact, data format) | P1 | Supplier created with unique ID; format mapping template assigned |
| FR-SUP-002 | System SHALL support supplier catalogue ingestion from CSV/Excel files | P1 | File parsed according to supplier format mapping; products normalised |
| FR-SUP-003 | System SHALL support supplier catalogue ingestion from PDF documents (via OCR) | P2 | PDF processed by Textract; table data extracted and normalised |
| FR-SUP-004 | System SHALL normalise supplier data into MerchOS product schema | P1 | Supplier columns mapped to product attributes; transformations applied |
| FR-SUP-005 | System SHALL detect and handle duplicate products from supplier imports | P1 | Duplicates detected by matching rules (title similarity, barcode, supplier SKU) |
| FR-SUP-006 | System SHALL calculate supplier data quality score | P2 | Score based on completeness, consistency, and accuracy of provided data |
| FR-SUP-007 | System SHALL support scheduled supplier feed ingestion | P2 | Automated pull/push on schedule; delta processing for changes only |
| FR-SUP-008 | System SHALL track price changes from supplier updates | P1 | Price change history maintained; alerts on significant changes (> 10%) |
| FR-SUP-009 | System SHALL support supplier-specific attribute mappings | P1 | Each supplier has custom column mapping; reusable across imports |
| FR-SUP-010 | System SHALL generate supplier ingestion reports (items processed, errors, new, updated) | P1 | Report available after each ingestion; downloadable |

---


## 12. Image Management

| ID | Requirement | Priority | Acceptance Criteria |
|----|------------|----------|-------------------|
| FR-IMG-001 | System SHALL support image upload (JPEG, PNG, WebP) up to 10MB per image | P0 | Image stored in S3; thumbnail generated; metadata recorded |
| FR-IMG-002 | System SHALL support multiple images per product (up to 12) with ordering | P0 | Primary image designated; display order maintained |
| FR-IMG-003 | System SHALL validate images against marketplace requirements (dimensions, format, size) | P0 | Non-compliant images flagged with specific violation details |
| FR-IMG-004 | System SHALL perform OCR on product images to extract text | P1 | Text extracted via Textract; results stored as searchable metadata |
| FR-IMG-005 | System SHALL analyse images to detect product labels and categories (Rekognition) | P1 | Labels returned with confidence; used for category recommendation |
| FR-IMG-006 | System SHALL detect image quality issues (blur, low resolution, wrong aspect ratio) | P1 | Quality score calculated; issues flagged before marketplace export |
| FR-IMG-007 | System SHALL support background removal for product images | P2 | Clean white/transparent background generated; before/after preview |
| FR-IMG-008 | System SHALL auto-resize images to meet marketplace specifications | P1 | Resized versions generated per marketplace; originals preserved |
| FR-IMG-009 | System SHALL detect prohibited content in images (marketplace compliance) | P1 | Rekognition moderation; watermarks, text overlays, logos detected |
| FR-IMG-010 | System SHALL generate optimised image variants (thumbnail, medium, full) | P0 | Three sizes generated on upload; served via CDN |

---

## 13. Notifications & Alerts

| ID | Requirement | Priority | Acceptance Criteria |
|----|------------|----------|-------------------|
| FR-NTF-001 | System SHALL send in-app notifications for key events | P0 | Notification bell with unread count; notification list view |
| FR-NTF-002 | System SHALL send email notifications for critical events | P0 | Export complete, validation failures, low stock, account alerts |
| FR-NTF-003 | System SHALL allow users to configure notification preferences | P1 | Per-event-type toggle for in-app, email, and SMS channels |
| FR-NTF-004 | System SHALL notify on AI enrichment completion | P0 | User alerted when AI processing finishes; link to review results |
| FR-NTF-005 | System SHALL notify on export completion (success or failure) | P0 | Notification includes file download link or error summary |
| FR-NTF-006 | System SHALL notify on marketplace schema changes (admin) | P1 | Platform admin alerted when knowledge base detects format changes |
| FR-NTF-007 | System SHALL notify on inventory low-stock threshold breach | P1 | Product(s) identified; current stock level shown |
| FR-NTF-008 | System SHALL notify on supplier feed ingestion completion | P1 | Summary: items processed, new, updated, errors |
| FR-NTF-009 | System SHALL notify on account security events (login from new device, password change) | P0 | Email sent immediately; includes device/location info |
| FR-NTF-010 | System SHALL support webhook delivery for integration events | P2 | Configurable webhook URL; JSON payload; retry on failure |

---

## 14. Analytics & Reporting

| ID | Requirement | Priority | Acceptance Criteria |
|----|------------|----------|-------------------|
| FR-ANL-001 | System SHALL provide dashboard with key metrics overview | P1 | Total products, exports this month, AI enrichments, marketplace readiness |
| FR-ANL-002 | System SHALL track export history and success rates per marketplace | P1 | Chart: exports over time, pass/fail ratio, trend lines |
| FR-ANL-003 | System SHALL report AI enrichment metrics (usage, accuracy, acceptance rate) | P1 | Credits used, descriptions accepted/edited/rejected, confidence distribution |
| FR-ANL-004 | System SHALL report product catalogue health metrics | P1 | Completeness score distribution, products needing attention, stale products |
| FR-ANL-005 | System SHALL report inventory status summary | P2 | In-stock, low-stock, out-of-stock counts; value at cost |
| FR-ANL-006 | System SHALL provide audit trail for all data modifications | P0 | Who changed what, when, previous value, new value; searchable |
| FR-ANL-007 | System SHALL support data export of analytics (CSV download) | P2 | All reports downloadable as CSV for external analysis |
| FR-ANL-008 | System SHALL track user activity metrics (sessions, actions, feature usage) | P2 | Aggregate analytics; no PII in analytics data |

---

## 15. Administration

| ID | Requirement | Priority | Acceptance Criteria |
|----|------------|----------|-------------------|
| FR-ADM-001 | System SHALL provide platform admin panel (internal MerchOS team only) | P0 | Separate admin interface; super-admin role required |
| FR-ADM-002 | System SHALL allow admins to manage tenant accounts (create, suspend, configure) | P0 | Full tenant lifecycle management without code deployment |
| FR-ADM-003 | System SHALL allow admins to update marketplace knowledge base schemas | P0 | Schema CRUD via admin UI; version control maintained |
| FR-ADM-004 | System SHALL allow admins to monitor platform health and metrics | P0 | Real-time dashboard: active tenants, API calls, error rates, AI usage |
| FR-ADM-005 | System SHALL allow admins to view and search audit logs across tenants | P1 | Cross-tenant audit search for security investigation |
| FR-ADM-006 | System SHALL allow admins to configure AI model parameters | P1 | Model selection, temperature, max tokens, prompt templates editable |
| FR-ADM-007 | System SHALL allow admins to manage feature flags for controlled rollout | P1 | Per-feature, per-tenant, percentage-based feature flag control |
| FR-ADM-008 | System SHALL provide cost tracking dashboard (AWS costs per tenant) | P1 | Cost attribution to tenants; budget alerts; trend analysis |
| FR-ADM-009 | System SHALL support system-wide announcements to all tenants | P2 | Banner notifications; scheduled maintenance notices |
| FR-ADM-010 | System SHALL log all admin actions for security audit | P0 | Admin actions immutably logged; separate from tenant audit logs |

---


## 16. Assumptions

| # | Assumption | Impact if Invalid |
|---|-----------|-------------------|
| A1 | All marketplace CSV specifications can be fully captured as configuration | Some marketplaces may need custom code adapters |
| A2 | AI confidence scores are reliable enough for user decision-making | Users may distrust AI suggestions; higher review overhead |
| A3 | Bulk operations (50K products) complete within 30 minutes | May need to increase async processing capacity or adjust SLAs |
| A4 | Single DynamoDB table handles all access patterns defined here | May require additional indexes or table splits |
| A5 | Users prefer AI suggestions over fully automated enrichment | UX may need automated mode for power users |

---

## 17. Dependencies

| Dependency | Required For | Status |
|-----------|-------------|--------|
| MERCH-002 (Business & Product) | Personas, user stories | Complete |
| MERCH-005 (AWS Architecture) | Technical implementation | Planned |
| MERCH-014 (Database Design) | Data model support | Planned |
| MERCH-015 (API Specifications) | API contracts | Planned |
| Amazon Cognito | FR-AUTH requirements | Available |
| Amazon Bedrock | FR-AI requirements | Available |
| Amazon Textract | FR-IMG-004, FR-SUP-003 | Available |
| Amazon Rekognition | FR-IMG-005, FR-IMG-009 | Available |

---

## 18. References

| # | Reference | Relevance |
|---|-----------|-----------|
| 1 | MERCH-002 (Business & Product) | User stories and personas that drive requirements |
| 2 | MERCH-004 (Non-Functional Requirements) | Quality attributes constraining functional behaviour |
| 3 | MERCH-014 (Database Design) | Data structures supporting these requirements |
| 4 | MERCH-015 (API Specifications) | API contracts implementing these requirements |
| 5 | IEEE 830 (SRS Standard) | Requirements documentation best practice |

---

## Requirements Traceability Matrix

| Requirement Group | User Story Source (MERCH-002) | Implementing Module | API Specification |
|-------------------|------------------------------|--------------------|--------------------|
| FR-AUTH (16 reqs) | PA-001, PA-005 | Identity & Access Management | /auth/* endpoints |
| FR-TEN (10 reqs) | PA-001, PA-002 | Identity & Access Management | /tenants/* endpoints |
| FR-PRD (34 reqs) | PM-001 through PM-010 | Product Hub | /products/* endpoints |
| FR-AI (14 reqs) | AI-001 through AI-010 | Product Intelligence Engine | /intelligence/* endpoints |
| FR-MKT (15 reqs) | EX-001, EX-010 | Marketplace Intelligence Engine | /marketplaces/* endpoints |
| FR-EXP (24 reqs) | EX-001 through EX-010 | Export Engine | /exports/* endpoints |
| FR-INV (10 reqs) | IV-001 through IV-006 | Inventory Engine | /inventory/* endpoints |
| FR-SUP (10 reqs) | SP-001 through SP-005 | Supplier Intelligence Engine | /suppliers/* endpoints |
| FR-IMG (10 reqs) | PM-002, AI-005 | Image Intelligence Engine | /images/* endpoints |
| FR-NTF (10 reqs) | Cross-cutting | Notification Service | /notifications/* endpoints |
| FR-ANL (8 reqs) | Cross-cutting | Analytics Service | /analytics/* endpoints |
| FR-ADM (10 reqs) | PA-001 through PA-006 | Administration Module | /admin/* endpoints |

**Total Functional Requirements: 161**

---

*End of Volume 03 — Functional Requirements*

*Previous: Volume 02 — Business & Product (MERCH-002)*
*Next: Volume 04 — Non-Functional Requirements (MERCH-004)*
