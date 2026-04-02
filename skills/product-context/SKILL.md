---
name: product-context
description: >
  Static product knowledge base for SparkIQ ERP: module inventory, CRE domain model,
  business rules, personas, competitive positioning, and product vision.
  Load when creating epics, answering product questions, or planning with the CEO.
user-invocable: true
---

# SparkIQ ERP — Product Context

This is the durable product knowledge base. It complements the technical status reports in `profile/projects/sparkiq-erp/` with business context, domain knowledge, and strategic positioning.

**Last updated:** 2026-04-02

---

## 1. Product Vision & North Star

**SparkIQ ERP** is the first modern, API-first commercial real estate ERP built for institutional operators who manage multi-entity portfolios. It replaces the legacy systems (Yardi, MRI) that dominate the market with a clean, traceable general ledger, automated lease billing, and native multi-entity accounting — all accessible through a modern UI and a complete REST API.

**First principles that guide every product decision:**

- **API-first**: Every feature is an API endpoint first, UI second. Enables integrations, automation, and future portals without retrofitting.
- **Clean ledger from day one**: Double-entry enforced at the database level. Full audit trail. No "reconciliation surprises" after go-live.
- **No legacy baggage**: Greenfield architecture. No decades of bolt-on modules, no acquired product sprawl, no on-prem deployment model.
- **Domain-native**: Purpose-built for CRE — lease terms, CAM, escalations, intercompany, and property accounting are first-class concepts, not customizations on a generic ERP.

---

## 2. Target Market & Personas

**Market segment:** Institutional CRE operators — mid-market firms managing $500M–$5B in assets under management, with multi-entity structures (separate legal entities per property or fund), 5–50 properties, and a need for consolidated reporting and intercompany accounting.

### Asset Manager
- **Goals:** Maximize portfolio NOI, make hold/sell decisions, report to investors
- **Pain points:** Scattered data across spreadsheets and legacy systems, no real-time property financials, manual investor reporting
- **SparkIQ value:** Property financials (cap rate, NOI, current value), consolidated reporting across entities, clean GL for auditable investor packages

### Property Accountant
- **Goals:** Accurate GL, timely period close, correct tenant billing, clean CAM reconciliation, audit-ready books
- **Pain points:** Manual journal entries, billing errors from spreadsheet-based schedules, painful period-end close, audit findings from inconsistent data
- **SparkIQ value:** Automated billing engine (contract → events → invoices → GL), double-entry enforcement, document lifecycle (draft → posted → void/reverse), accounting periods (open/close/lock)

### Property Manager / Operations
- **Goals:** Lease administration, tenant relations, vendor management, space occupancy
- **Pain points:** Lease data buried in PDFs, no visibility into upcoming expirations or escalations, manual rent roll generation
- **SparkIQ value:** Contract engine with term types (base rent, escalation, CAM, abatement), space management, party multi-role system, rent roll reporting

### CFO / Controller
- **Goals:** Consolidated financial statements, multi-entity compliance, intercompany elimination, tax reporting
- **Pain points:** Manual consolidation across entities, intercompany transactions tracked in spreadsheets, no role-based access control
- **SparkIQ value:** Native multi-entity with intercompany module (IC transactions, settlements, fee rules, position/aging reports), financial reporting endpoints, RBAC (planned)

---

## 3. Module & Feature Inventory

Organized by business domain. Status reflects the product capability, not code completeness (see status reports for technical detail).

### Core Accounting
| Module | Business Purpose | Status |
|--------|-----------------|--------|
| Chart of Accounts | Hierarchical account structure with system roles and detail types | Built (read-only API; write endpoints planned) |
| General Ledger | Double-entry journal entries with dimension tagging | Built |
| GL Activity Report | Transaction-level ledger filtered by date, entity, account | Built |
| Accounting Periods | Monthly open/close/lock cycle to prevent retroactive posting | Planned (schema exists) |
| Financial Reporting (P&L, BS, TB) | Standard financial statements | Planned |

### Lease Management
| Module | Business Purpose | Status |
|--------|-----------------|--------|
| Contracts — Lease | Commercial/multifamily/mixed-use leases with complex term structures | Built |
| Contracts — Management | Property management agreements with fee rules | Built |
| Contract Terms | Base rent, escalation, abatement, CAM, percentage rent, late fee, renewal options | Built |
| Lease Terms | Possession date, rent commencement, sqft, units, security deposit | Partial (schema exists; not fully exposed via API) |
| Billing Schedules | 12-month event horizon generated on contract activation | Built |
| Financial Events | Automated document generation from billing schedules (cron every minute) | Built |

### Receivables & Billing
| Module | Business Purpose | Status |
|--------|-----------------|--------|
| Invoices | Tenant billing with line items, tax, auto-numbering | Built |
| Payment Applications | Directional payment matching (payment_in → invoices) | Built |
| Tax Rates | VAT/GST/sales tax on invoices | Planned (stub — blocking compliant invoicing) |

### Payables
| Module | Business Purpose | Status |
|--------|-----------------|--------|
| Bills | Vendor bills with line-by-line property/space allocation | Built |
| Payments Out | Vendor payment processing (payment_out → bills) | Built |

### Banking
| Module | Business Purpose | Status |
|--------|-----------------|--------|
| Bank Accounts | Linked to GL cash accounts | Built |
| Transactions | Import, categorize, match/unmatch/revert full lifecycle | Built |

### Property & Space Management
| Module | Business Purpose | Status |
|--------|-----------------|--------|
| Properties | Portfolio with status tracking (active/inactive/sold/under_contract) | Built |
| Spaces | Nested under properties — types, vacancy, sqft | Built |
| Property Financials | Cap rate, NOI, acquisition cost, current value | Planned (schema exists) |

### Entity & Party Management
| Module | Business Purpose | Status |
|--------|-----------------|--------|
| Legal Entities | Hierarchical parent/child with entity types, `is_management` flag | Built |
| Parties | Multi-role (tenant/vendor/owner/investor/bank), addresses | Built |
| Org Management | Members, roles, settings | Planned (minimal stub) |

### Intercompany
| Module | Business Purpose | Status |
|--------|-----------------|--------|
| IC Transactions | Cross-entity transactions (e.g., management fees) | Built |
| IC Settlements | Approval/dispute/post workflow for intercompany balances | Built |
| Fee Rules | Per-account commission percentages on management contracts | Built |
| IC Reports | Position report + aging report | Built |

### Items & Catalog
| Module | Business Purpose | Status |
|--------|-----------------|--------|
| Items | Revenue/expense item catalog for billing line items | Built (read-only API; write endpoints planned) |

### Automation & Events
| Module | Business Purpose | Status |
|--------|-----------------|--------|
| Rule Templates & Instances | Billing rule configuration | Partial (backend exists; no UI) |
| Event Overrides | Skip/modify/reschedule individual billing events | Planned (schema exists; no API) |

### Reporting & Analytics
| Module | Business Purpose | Status |
|--------|-----------------|--------|
| Rent Roll | Active leases, spaces, tenants, rates, expirations | Planned |
| P&L Statement | Revenue minus expenses by period and entity | Planned |
| Balance Sheet | Assets, liabilities, equity at a point in time | Planned |
| Trial Balance | Account balances for period-end verification | Planned |
| Dashboard | KPI widgets, activity feed, operational overview | Planned (FE stub only) |

### Settings & Admin
| Module | Business Purpose | Status |
|--------|-----------------|--------|
| User Management | Invite, roles, permissions | Planned |
| RBAC | Endpoint-level role checks (owner/admin/member) | Planned |
| Org Settings | Company preferences, defaults | Planned |

---

## 4. CRE Domain Glossary

**Lease-to-Cash Cycle:** The core revenue flow. Lease execution → billing schedule generation → financial event triggers → invoice creation → payment application → GL posting → financial reporting. SparkIQ automates the middle steps (billing schedule through GL posting).

**CAM Reconciliation (Common Area Maintenance):** Tenants pay estimated CAM charges monthly. At year-end, actual costs are tallied and compared to estimates. Difference becomes a true-up invoice (tenant owes more) or credit memo (overpaid). SparkIQ models this through contract terms of type `CAM` with billing schedules.

**NOI (Net Operating Income):** Property revenue minus operating expenses, excluding debt service and capital expenditures. The primary valuation metric for CRE. NOI / Cap Rate = Property Value.

**Cap Rate (Capitalization Rate):** NOI divided by property value. Represents the yield on a CRE investment. A 6% cap rate means the property generates 6% annual return on its value. Stored in `property_financials`.

**Rent Roll:** Point-in-time snapshot of all active leases showing: tenant, space, base rent, escalation schedule, lease start/end, security deposit. The most-requested report for asset managers and lenders.

**Escalation:** Contractual rent increase applied on lease anniversary. Types: fixed amount ($500/year), percentage (3%/year), or CPI-indexed. SparkIQ supports percentage and fixed_amount escalations with anniversary-based annual term splitting.

**Triple Net (NNN):** Lease structure where tenant pays base rent plus property taxes, insurance, and CAM. Most common in commercial CRE. Each component is a separate contract term in SparkIQ.

**Abatement:** Rent-free period, typically at lease start as a tenant incentive. Must still be tracked for straight-line rent accounting (GAAP). Modeled as a contract term of type `abatement`.

**Security Deposit:** Cash held by landlord as collateral. Recorded as a liability on the GL. Returned at lease end minus damages. Tracked in `lease_terms`.

**Intercompany:** Transactions between legal entities under the same organization. Common example: management company charges property entity a management fee (percentage of collected rent). SparkIQ has a full IC module with fee rules, settlements, and elimination reporting.

**Accounting Period:** Monthly cycle: Open (accepting entries) → Closed (period-end review complete) → Locked (no further changes). Prevents retroactive posting that would invalidate financial statements.

**Double-Entry Accounting:** Every debit has an equal credit. Journal entries must balance — enforced by a database CHECK constraint in SparkIQ. This is non-negotiable.

**Document Lifecycle:** Draft → Posted → Void/Reverse. Only draft documents can be edited. Posting creates journal entries (GL impact). Voiding creates reversing entries (not deletion). This preserves the audit trail.

**Document Sequencing:** Auto-numbering by document type and year (e.g., INV-2026-0001). Gaps in sequence numbers are preserved (voided documents keep their number). Ensures audit compliance.

---

## 5. Key Business Rules & Data Model

### Validation & Lifecycle Rules

- **JE balance enforcement:** Sum of debits must equal sum of credits on every journal entry. Enforced by DB CHECK constraint — not just application code.
- **Document lifecycle:** Draft → Posted → Void/Reverse. Only drafts are editable. Posting is async via BullMQ queue with recovery processor for failures.
- **Auto-numbering:** Sequential per document type per year. Managed by `document_sequences` table. Idempotent — no duplicates on retry.
- **Org scoping:** Every table has `org_id`. All queries scoped by org. Supabase RLS provides defense-in-depth at the database level.
- **Contract term validation:** Each term type (rent_base, escalation, CAM, etc.) has a dedicated Zod schema. Terms are validated at the application layer in addition to DTO-level class-validator.
- **Billing horizon:** Contract activation generates a 12-month financial event horizon. Events trigger document generation via cron (every minute).
- **Payment directionality:** `payment_in` applies to invoices (AR). `payment_out` applies to bills (AP). Cross-application is blocked.
- **IC settlement workflow:** Created → Approved → Posted (or Disputed). Settlements create offsetting journal entries across both entities.
- **Management fee rules:** Per-account commission percentages on management contracts. Fee calculation runs against collected revenue per the account mapping.

### Entity Relationship Model

```
Organization (org)
├── Legal Entities (hierarchical parent/child)
│   ├── Properties
│   │   └── Spaces (nested, typed, vacancy-tracked)
│   └── Contracts (lease or management)
│       ├── Contract Terms (typed: rent, escalation, CAM, etc.)
│       ├── Lease Terms (sqft, units, security deposit, dates)
│       └── Financial Events → Documents → Journal Entries → GL
├── Parties (multi-role: tenant/vendor/owner/investor/bank)
│   └── Party Roles + Addresses
├── Bank Accounts → Bank Transactions → matched to Documents
├── Items (revenue/expense catalog for billing line items)
└── Intercompany Account Mappings → IC Transactions → IC Settlements
```

**Key relationships:**
- A Party can hold multiple roles (e.g., both tenant and vendor). Roles are separate from the party record.
- A Contract belongs to a Legal Entity and references a Party (the counterparty). Management contracts also reference a counterparty legal entity (for intercompany).
- Financial Events are children of Billing Schedules, which are children of Contracts. Events generate Documents, which create Journal Entries.
- Journal Lines carry optional `counterparty_legal_entity_id` for intercompany tracing.

---

## 6. Competitive Positioning

### Incumbent Landscape

| Competitor | Weakness SparkIQ Exploits |
|-----------|--------------------------|
| **Yardi Voyager/Breeze** | Decades-old monolith. On-prem or hosted (not cloud-native). Expensive implementation ($500K+). Long contracts. UI from 2005. No real API — integrations are batch file exports. |
| **MRI Software** | Acquired sprawl of 30+ products. Integration between modules is painful. High TCO from professional services. Not a unified platform. |
| **AppFolio / Buildium** | Residential-focused. Lack institutional CRE features: no multi-entity, no intercompany, no CAM reconciliation, no complex lease terms (escalations, percentage rent). |
| **Generic ERPs (NetSuite, Sage Intacct)** | Not CRE-native. Lease management, property accounting, and CAM require heavy customization modules. Ongoing maintenance burden. |

### SparkIQ Differentiators

1. **API-first architecture** — every feature is an endpoint. Enables custom integrations, investor portals, and automation without waiting for vendor roadmap.
2. **Clean GL from day one** — double-entry enforced at DB level, full audit trail, document lifecycle with reversing entries (not deletion).
3. **Native multi-entity + intercompany** — not a bolt-on. IC transactions, settlements, fee rules, and elimination reporting are core modules.
4. **Automated billing engine** — contract → billing schedule → events → documents → GL. Zero manual steps from lease activation to revenue recognition.
5. **Modern stack, modern UX** — NestJS + React 19 + Next.js. Fast iteration. No Java monolith. No 15-minute page loads.
6. **Transparent pricing model** — no implementation fees, no per-seat gouging, no multi-year lock-in (planned GTM positioning).

---

## 7. Product Roadmap Phases

### Phase 1 — Core ERP (Current)
**Goal:** Can we run a portfolio on this?
- Core accounting (GL, JE, COA, documents)
- Lease management (contracts, terms, billing schedules, events)
- Receivables (invoices, payments) + Payables (bills, payments)
- Banking (accounts, transactions, matching)
- Intercompany (transactions, settlements, fee rules)
- **Success criteria:** A single operator can manage a 10-property portfolio end-to-end without touching a spreadsheet for core accounting and billing.

### Phase 2 — Operational Completeness
**Goal:** Production-ready for paying customers.
- Financial reporting (P&L, Balance Sheet, Trial Balance, Rent Roll)
- Tax rates module (compliant invoicing)
- COA + Items write endpoints (self-service onboarding)
- Contract lifecycle transitions (terminate, expire)
- Accounting periods (open/close/lock)
- Org management + RBAC (multi-user, role-based access)
- **Success criteria:** Finance team can do month-end close, generate statements, and operate without engineering support.

### Phase 3 — Operator Experience
**Goal:** Operators love using this daily.
- Dashboard (KPIs, activity feed, operational overview)
- Automation UI (rule management, event overrides)
- Property financials (cap rate, NOI, asset performance)
- Rent roll report
- Lease expiration tracking + alerts
- **Success criteria:** Daily active usage by property accountants and managers. NPS > 40.

### Phase 4 — Scale & Intelligence
**Goal:** Institutional grade — ready for $5B+ portfolios.
- Multi-currency support
- CPI-indexed escalations
- Investor portal (read-only reporting for LPs)
- Budgeting & forecasting
- AI-assisted bank reconciliation
- Bulk import/migration tools (from Yardi/MRI)
- **Success criteria:** Win a competitive deal against Yardi with a firm managing 50+ properties.

---

## 8. When to Use This Skill

**Load `product-context` when:**
- Creating a new epic — use the module inventory, roadmap phases, and personas to write business-grounded stories with clear acceptance criteria
- CEO asks about product direction, competitive positioning, or "what does the product do today"
- Dev is blocked on a business rule — check here before escalating (the domain glossary and business rules sections may have the answer)
- Planning session where you need to propose next priorities — reference roadmap phases and the module inventory to ground your proposal

**Do NOT load for:**
- Routine state machine operations (status pings, label updates, PR management)
- Technical implementation questions (those belong in the status reports or the codebase itself)
- Simple GitHub issue management (assign, label, comment)

**This skill complements — does not replace:**
- `profile/projects/sparkiq-erp/fe-status-report.md` — technical FE status
- `profile/projects/sparkiq-erp/be-status-report.md` — technical BE status
- `memory/goals.qmd` — one-line vision (this skill expands on it)
