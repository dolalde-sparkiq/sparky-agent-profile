# SparkIQ ERP — Backend Status Report
**Repo:** `sparkiq-gh/sparkiq-erp-be`
**Stack:** NestJS 11 · TypeScript · Drizzle ORM · PostgreSQL (Supabase) · Fastify · BullMQ · Redis
**Report Date:** 2026-03-25
**Primary Engineer:** Daniel Antonini (dantonini@sparkiq.io / @Antonini2207)

---

## 1. Repository Snapshot

| Field | Value |
|---|---|
| Last Commit | 2026-03-25 (PR #10 merged — management contract enhancements) |
| Total Commits | ~100+ (single-developer, consistent cadence) |
| Active Branch | `main` |
| Stale Branches | `feature/mejoras`, `billing` |
| Open PRs | 0 |
| Open Issues | 0 |
| Migrations Applied | 23 (0000–0022) |
| Total API Endpoints | ~70 |

---

## 2. Milestones Achieved ✅

### M1 — Infrastructure & Auth Foundation
- NestJS + Fastify scaffold with global Supabase JWT guard
- Multi-tenant org isolation (all queries scoped by `org_id`)
- Swagger/OpenAPI UI at `/docs`
- Sentry error monitoring + profiling
- Rate limiting (short / medium / long windows via `@nestjs/throttler`)
- Health checks (`/health` via Terminus)
- BullMQ job queues (`posting`, `tasks`, `posting-recovery`)

### M2 — Core Entity Management
- **Legal Entities:** Full CRUD with hierarchical parent/child support, entity types, `is_management` flag
- **Parties:** Full CRUD, 21 party types, multi-role support (tenant/vendor/owner/investor/management_company/bank), addresses
- **Properties:** Full CRUD + soft status (active/inactive/sold/under_contract), acquisition/sale date tracking
- **Spaces:** Full CRUD nested under properties, space types, vacancy status

### M3 — Chart of Accounts & General Ledger
- Account types + detail types reference data
- Hierarchical accounts with system roles (`system_role` field)
- Journal entries with double-entry lines (enforced CHECK: exactly one of debit/credit > 0)
- GL Activity report with date/entity/account filtering
- Accounting dimensions (analytics tagging on entries and lines)

### M4 — Document Engine
- **Document types:** invoice, credit_memo, bill, expense, payment_in, payment_out, journal_entry, intercompany_transaction, intercompany_settlement, sales_receipt
- Full lifecycle: create → post (via BullMQ posting queue) → void/reverse
- Auto-numbering sequences by document type + year
- Payment applications with directional validation
- Full CQRS implementation (`CommandBus` / `QueryBus`)
- Posting recovery processor for failed posts

### M5 — Banking Module
- Bank accounts linked to GL cash accounts
- Manual bank transaction import
- Full categorization workflow: uncategorized → excluded / needs_review / accounted
- Transaction processing: creates documents + journals automatically
- Match / unmatch / revert full lifecycle
- Multi-direction (in/out) processing options
- Documented in `docs/bank-transactions-match-and-revert.md`

### M6 — Contract Engine
- **Lease contracts:** commercial/multifamily/mixed_use, possession date, rent commencement, rentable sqft, units, security deposit
- **Management contracts:** property-scoped, counterparty legal entity, fee rules with per-account commission percentages
- Complex term types: rent_base, rent_escalation, abatement, CAM, percentage_rent, late_fee, option_renewal
- Zod-validated term schemas per term type
- Anniversary-based annual term splitting for multi-year leases
- Escalation support (percentage / fixed_amount)
- Contract files (upload URL + SHA256 tracking)

### M7 — Billing Schedule / Event Engine
- `financial_events` table drives automated document generation
- `DocumentGeneratorCron` runs every minute via `@nestjs/schedule`
- Contract activation generates 12-month event horizon
- Recalculate / initialize / run-maintenance endpoints
- Draft → document pipeline
- Billing frequencies: monthly, weekly, biweekly

### M8 — Intercompany Module
- IC transactions (`intercompany_transaction` document type)
- IC settlements with approval / dispute / post workflow
- Intercompany account mappings for fee rules
- IC position report + aging report
- Management fee rule engine (per-account commission percentages on management contracts)
- `counterparty_legal_entity_id` on journal lines for intercompany tracing

---

## 3. Pending Milestones / Gaps 🔴

### P1 — Tax Rates Module (BLOCKING for compliant invoicing)
- `TaxRatesController.list()` returns hardcoded empty array
- No `tax_rates` database table
- `tax_amount` column exists on documents but cannot be populated via API
- **Impact:** Any jurisdiction requiring VAT/GST/sales tax on invoices is broken

### P2 — Chart of Accounts Write Operations
- No `POST /accounting/accounts`, `PUT`, or `DELETE` endpoints
- Accounts must be seeded via SQL scripts
- **Impact:** Onboarding new clients requires direct DB access; not scalable

### P3 — Items Write Operations
- No `POST /items`, `PUT /items/:id`, `DELETE /items/:id`
- Item catalog is read-only via API; only manageable via seed SQL
- **Impact:** PMs and operations cannot manage billing items without engineering support

### P4 — Contracts: Missing Lifecycle Transitions
- No `DELETE` / terminate / expire endpoint
- Only `PATCH` available, and only on draft contracts
- **Impact:** Cannot programmatically transition active leases to terminated/expired status

### P5 — Lease Terms Not Fully Exposed
- `lease_terms` table (sqft, units, security deposit, possession date, rent commencement) exists in schema
- No dedicated endpoint to read or write lease-specific term fields
- **Impact:** Key lease data is in the DB but not surfaced to the frontend

### P6 — Org Management
- Only `GET /org/current` exists (returns only `org_id`)
- No: create org, invite members, manage member roles, update org settings
- **Impact:** All org provisioning requires direct DB or Supabase console access

### P7 — Accounting Periods
- `accounting_periods` table (open/closed/locked) is in schema
- No API endpoints to open, close, or lock periods
- **Impact:** Period-end close workflow is entirely manual

### P8 — Financial Reporting Endpoints
- No balance sheet, P&L, trial balance, or rent roll endpoints
- Only GL activity (a raw transaction ledger) is exposed
- **Impact:** Finance team cannot generate reports from the API; requires external BI tool

### P9 — Event Overrides Not Exposed
- `event_overrides` table (skip/modify_payload/reschedule) exists in schema
- No API endpoint to create or manage overrides
- **Impact:** Operators cannot override individual billing events without DB access

### P10 — Dimensions Not Exposed
- `dimensions` table exists (custom analytics tagging)
- No endpoints to manage dimensions
- **Impact:** Analytics customization requires direct DB access

### P11 — Property Financials Not Exposed
- `property_financials` table (cap_rate, NOI, acquisition_cost, current_value) is in schema
- No API endpoint
- **Impact:** Asset performance data cannot be read or updated via API

### P12 — Contract Files Endpoint Missing
- `contract_files` table exists (file_url, sha256)
- No endpoint to upload, list, or delete contract files

### P13 — Role-Based Authorization
- Global JWT guard confirms org membership only
- No endpoint-level role checks (owner vs. admin vs. member)
- **Impact:** All authenticated org members have full read/write access to everything

---

## 4. Areas Needing Immediate Attention 🚨

### CRITICAL — Security: Credentials Committed to Repo
- `var.txt` in the repo root contains **live credentials in plaintext**:
  - Supabase database connection string including password (`##Spark@adm1n!!`)
  - Supabase project URL and anon key
- This file is **NOT in `.gitignore`**
- **Action required immediately:** rotate the DB password and Supabase anon key, then add `var.txt` to `.gitignore` or delete it

### HIGH — Pagination Metadata Missing
- All list endpoints return `{ data: [...] }` with no `total`, `page`, or `hasNextPage`
- Frontend cannot render pagination controls

### HIGH — `getById` Returns `null` with HTTP 200
- Several controllers return `null` instead of throwing `NotFoundException` when a record is not found
- Affected: `PropertiesController`, `BankAccountsController`, `BankingController`
- Frontend must distinguish `200 null` from `200 {data}` — error-prone

### MEDIUM — Dual Event Systems Need Reconciliation
- Two separate scheduling tables: `events` (general, JSONB payload, idempotency keys) and `financial_events` (billing-specific)
- Billing cron uses `financial_events` only; `events` table capabilities (overrides, idempotency) are underutilized
- Creates architectural ambiguity for future event types

### MEDIUM — BullMQ Disabled in Dev
- `QUEUE_ENABLED=false` in `var.txt` (dev environment)
- Document posting falls back to a non-queued path in dev, creating behavior divergence from production
- Should be documented and tested explicitly

### MEDIUM — Hardcoded Currency in Banking
- `currency: 'USD'` hardcoded in `BankingController.list()` response
- `bank_accounts.currency_code` field exists but is ignored here

### LOW — No Test Coverage
- Only one spec file found (`contracts.service.spec.ts`)
- `test/e2e/` and `test/integration/` directories appear sparse
- No evidence of CI running tests on PRs

---

## 5. API Surface Summary

| Module | Endpoints | Status |
|---|---|---|
| Auth | 1 | ✅ Complete |
| System / Health | 6 | ✅ Complete |
| Legal Entities | 4 | ✅ Complete (no DELETE) |
| Parties | 4 | ✅ Complete (no DELETE) |
| Properties + Spaces | 10 | ✅ Complete |
| Accounting — Accounts | 3 | ⚠️ Read-only (no write) |
| Accounting — Journals | 2 | ✅ Complete |
| GL Activity | 1 | ✅ Complete |
| Documents | 5 | ✅ Complete |
| Payment Applications | 3 | ✅ Complete |
| Banking — Bank Accounts | 4 | ✅ Complete |
| Banking — Transactions | 11 | ✅ Complete |
| Contracts | 4 | ⚠️ No lifecycle transitions |
| Billing Schedules | 8 | ✅ Complete |
| Intercompany | 11 | ✅ Complete |
| Items | 1 | ⚠️ Read-only |
| Tax Rates | 1 | 🔴 Stub — not implemented |
| Org | 1 | ⚠️ Minimal stub |
| **Total** | **~70** | |

---

## 6. Database Schema Summary

**35+ tables across 23 migrations**

| Domain | Tables |
|---|---|
| Organization | `orgs`, `org_members` |
| Entities & Parties | `legal_entities`, `parties`, `party_roles`, `party_addresses`, `party_legal_entities` |
| Properties | `properties`, `spaces`, `property_financials` |
| Accounting | `accounts`, `account_types`, `account_detail_types`, `dimensions`, `journal_entries`, `journal_lines`, `journal_entry_dimensions`, `journal_line_dimensions`, `accounting_periods` |
| Documents | `documents`, `document_lines`, `document_dimensions`, `document_line_dimensions`, `document_sequences` |
| Contracts | `contracts`, `contract_terms`, `lease_terms`, `contract_files` |
| Billing Engine | `financial_events`, `rule_templates`, `rule_instances`, `events`, `event_overrides` |
| Banking | `bank_accounts`, `bank_transactions` |
| Intercompany | `intercompany_account_mappings`, `ic_settlement_applications`, `ic_settlement_adjustments` |
| Items & Tax | `items`, ~~`tax_rates`~~ (not created) |

---

## 7. Recommended Sprint Priorities

### Sprint 1 — Security & Stability (Must ship before any external access)
1. 🔴 Rotate leaked credentials; remove `var.txt` from repo / add to `.gitignore`
2. 🔴 Add pagination metadata to all list endpoints (`total`, `page`, `hasNextPage`)
3. 🔴 Fix `getById` endpoints — return `404 NotFoundException` instead of `200 null`
4. 🔴 Implement role-based authorization guards (owner / admin / member) on write endpoints

### Sprint 2 — Core Missing Functionality
5. Tax Rates module (table + CRUD + wire to `documents.tax_amount`)
6. Chart of Accounts write endpoints (CREATE / UPDATE / DELETE with soft-delete guard)
7. Items write endpoints (CREATE / UPDATE / DELETE)
8. Expose `lease_terms` data through contracts GET endpoint

### Sprint 3 — Operational Completeness
9. Contract lifecycle transitions (terminate, expire, assign)
10. Accounting Periods endpoints (open / close / lock)
11. Org management endpoints (members, roles, settings)
12. Property Financials endpoints (GET + POST/PUT)

### Sprint 4 — Reporting & Analytics
13. Trial Balance endpoint
14. Rent Roll endpoint
15. Basic P&L and Balance Sheet endpoints
16. Dimensions management endpoints

### Sprint 5 — Quality & Reliability
17. Reconcile dual event systems (`events` vs `financial_events`)
18. Fix hardcoded USD currency in banking response
19. Add test coverage (unit + integration + e2e) with CI enforcement
20. Add `hasNextPage` cursor-based pagination to high-volume endpoints

---

## 8. Key Technical Decisions & Context

- **CQRS pattern** used throughout the document engine — commands and queries are fully separated. Future modules should follow this pattern for consistency.
- **BullMQ** used for async document posting — ensures retries and recovery but requires Redis. Queue can be disabled (`QUEUE_ENABLED=false`) for serverless/edge environments.
- **Zod** used for contract term validation at the application layer (in addition to class-validator at the DTO layer) — this is intentional for the complex nested term schemas.
- **Supabase RLS** is enabled on key tables (`orgs`, `properties`, `accounts`, etc.) — application-level org scoping AND DB-level RLS provide defense in depth.
- **Fastify** adapter used instead of Express — better performance but some Express-specific middleware is incompatible.
- **`@nestjs/schedule`** cron (`DocumentGeneratorCron`) runs every minute — monitor for overlap/lock issues at scale.
