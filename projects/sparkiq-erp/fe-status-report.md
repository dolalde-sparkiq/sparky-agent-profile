# SparkIQ ERP — Frontend Status Report
**Repo:** `sparkiq-gh/sparkiq-erp`
**Stack:** Next.js 16 · React 19 · TypeScript · Tailwind CSS v4 · Supabase Auth · TanStack Table v8 · Radix UI · Sentry · Vercel
**Report Date:** 2026-03-25
**Engineers:** Daniel Antonini (lead), Williams Verde, JCibeiraSparkiq

---

## 1. Repository Snapshot

| Field | Value |
|---|---|
| Last Commit | 2026-03-19 (PR #6 merged — feature/financial) |
| Total Commits | 59 |
| Development Period | 2026-02-04 → 2026-03-19 (~6.5 weeks) |
| Active Branch | `main` |
| Stale Branches | `feature/financial` (merged), `billing` (open/stale) |
| Open PRs | 0 |
| Open Issues | 0 |
| Active Engineers | 3 (Daniel Antonini, Williams Verde, JCibeiraSparkiq) |

---

## 2. Milestones Achieved ✅

### M1 — Application Shell & Navigation
- Full collapsible sidebar with SparkIQ branding
- Breadcrumb header derived from route metadata (single source of truth in `lib/routes.ts`)
- Responsive layout with mobile breakpoint detection
- Global drawer/dialog state machine (`LayoutProvider`) enabling cross-page panel management
- Quick Create menu in sidebar header
- User avatar dropdown with logout

### M2 — Authentication
- Email/password login via Supabase `signInWithPassword`
- Server-side session management with `@supabase/ssr` (cookie-based)
- Auth middleware (`proxy.ts`) protecting all routes except `/auth/sign-in`, `/auth/callback`
- Post-login redirect via `?next=` query param
- `UserProvider` context for session-aware components throughout the tree
- Supabase OAuth callback route (`/auth/callback/route.ts`)

### M3 — Reusable Data Infrastructure
- Custom `fetch`-based API client (`lib/api/client.ts`) with Bearer token auth
- `useInfiniteResource` — generic infinite-scroll data fetching hook with cursor pagination
- `useFilters` — URL-synchronized filter state manager (filters survive page refresh)
- `useIntersectionLoadMore` — Intersection Observer for scroll-triggered data loading
- `usePaginatedQuery` — page-based query alternative
- Domain hook pattern: every module has `use[Domain]Resource`, `use[Domain]Filters`, `use[Domain]Table` hooks
- Generic table system: `TableView`, `InfiniteTable`, `TableHeaders`, `TableCells`, `TableSkeleton`, `EmptyState`

### M4 — Documents Module (Invoices, Bills, Journal Entries)
- **Invoices:** 46KB creation wizard with line items, party/legal entity/property selection, tax, document sequencing; detail drawer; status filters; void/reverse actions
- **Bills:** 33KB creation wizard with vendor selection, line-by-line property/space allocation; detail drawer; actions
- **Journal Entries:** 27KB creation dialog with debit/credit line entry table; post/reverse/delete actions
- All three share consistent patterns: list page → client component → detail drawer → action menu

### M5 — Banking Module
- **Bank Accounts:** Full CRUD — create/edit dialog with parent cash account linking, filter, row actions
- **Transactions:** List with advanced filters, inline transaction processor, split transaction dialog, full process dialog, popover processor; covers categorize/process/unmatch/revert full lifecycle

### M6 — Accounting Module
- **Chart of Accounts (COA):** Account list with filters, create account dialog with hierarchical parent selection
- **Journals:** Journal list, detail drawer showing linked documents (invoices/bills)
- **GL Activity:** GL activity report with date/entity/account filter controls

### M7 — Contracts Module
- Contracts list page with `ContractsClient`
- Contract detail page (`/contracts/[id]`) with `ContractDetailClient`
- 46KB `ContractSheet` — comprehensive create/edit side sheet with: contract types (lease/management), term types, escalation details, billing term types (`BILLABLE_TERM_TYPES`), destination account field, draft-to-active terminology

### M8 — Properties Module
- Properties list with filters and create dialog (14KB)
- Property detail page (21KB) with spaces submodule
- Space create dialog (8KB)
- Properties and spaces hooks wired to backend

### M9 — Parties & Organization
- Parties list with type-filtered views (Tenants, Vendors)
- Party create/edit dialog
- Legal Entities list with `is_management` filter, 13KB create/edit dialog
- Legal entity and party hooks wired to backend

### M10 — Items Module
- Full CRUD: create, edit, delete, detail drawer
- Real-time insert notification via Supabase Realtime (`RealtimeNotifications.tsx`)
- Filter controls and sorting

### M11 — Monitoring & Analytics
- Sentry integrated at all three levels: client (`instrumentation-client.ts`), server (`sentry.server.config.ts`), edge (`sentry.edge.config.ts`)
- Vercel Analytics integrated in root layout

---

## 3. Pending Milestones / Gaps 🔴

### P1 — Dashboard (CRITICAL — first screen users see)
- `app/page.tsx` is 5 lines: a heading "Welcome to Modern CRE ERP" and a placeholder sentence
- No KPIs, no summary widgets, no charts, no activity feed
- **Impact:** Product feels unfinished at first login; no operational overview for users

### P2 — Reports Module (Not built)
- `/reports/pl` (P&L) and `/reports/balance-sheet` are defined in `routes.ts` and sidebar navigation
- No `app/reports/` directory exists — clicking these nav items will 404
- **Impact:** Finance team has no reporting surface in the app

### P3 — Automation Module (Not built)
- `/automation/rules`, `/automation/events`, `/automation/overrides` are in `routes.ts`
- No `app/automation/` directory exists
- **Impact:** Billing rule management and event overrides have no UI

### P4 — Billing Schedules (Stub)
- Route `/billing-schedules/contracts/[id]` exists as a directory
- Content is empty or minimal — the billing schedule detail view is not built
- **Impact:** Cannot view or manage billing events from the frontend after activating a contract

### P5 — Settings Module (Not built)
- `/settings` and `/settings/users` defined in navigation
- No `app/settings/` directory
- **Impact:** No user management, org settings, or preferences UI

### P6 — Support Page (Not built)
- `/support` in nav but no page built

### P7 — Properties Spaces Dedicated Route (Partial)
- `app/properties/spaces/` directory exists but content not confirmed as a working, complete page
- Spaces within a property are accessible from property detail, but the global spaces list view (`/properties/spaces`) may be incomplete

---

## 4. Areas Needing Immediate Attention 🚨

### CRITICAL — Post-Login Redirect Bug
- `login-form.tsx` defaults the `next` redirect param to `/leads/lead-matches`
- This route does not exist in the app — users will land on a 404 after logging in unless `?next=` is explicitly set
- **Fix:** Change default redirect to `/` or `/banking/transactions`

### HIGH — Missing `.env.example`
- No environment variable documentation file found in the repo
- Required vars (`NEXT_PUBLIC_SUPABASE_URL`, `NEXT_PUBLIC_SUPABASE_ANON_KEY`, `SUPABASE_SERVICE_ROLE_KEY`, `NEXT_PUBLIC_API_URL`, `SENTRY_*`) must be reverse-engineered from code
- **Fix:** Add `.env.example` with all required variables documented

### HIGH — Multiple Package Manager Lockfiles
- `pnpm-lock.yaml`, `yarn.lock`, and `package-lock.json` all coexist in the repo
- Creates dependency drift risk and onboarding confusion
- **Fix:** Delete two lockfiles; standardize on pnpm (per CLAUDE.md / memory)

### HIGH — `data.ts` Static Files May Be Unresolved Mock Data
- `app/items/data.ts`, `app/banking/accounts/data.ts`, `app/banking/transactions/data.ts`, `app/documents/invoices/data.ts`, `app/documents/bills/data.ts` all exist as small static files
- May indicate these modules are partially wired to live API but still using local fallbacks
- **Fix:** Audit each `data.ts` — if it contains mock data, remove and ensure module is fully API-connected

### HIGH — No Test Suite
- No testing framework installed (no Jest, Vitest, Playwright, Cypress)
- No `test` script in `package.json`
- Zero test coverage
- **Fix:** Install Vitest + React Testing Library for unit tests; Playwright for E2E

### MEDIUM — No Branch Protection on `main`
- Direct pushes to `main` are allowed
- No required PR reviews or status checks
- **Fix:** Enable branch protection: require 1 review + status checks before merge

### MEDIUM — No Pre-commit Hooks
- No Husky, no `lint-staged` in `package.json`
- Inconsistent commit messages in history ("done", "improve", "Temp", "wv: test")
- **Fix:** Add Husky + lint-staged for ESLint + Prettier on staged files

### MEDIUM — Hardcoded Fallback API URL
- `lib/api/client.ts` defaults to `https://sparkiq-erp-api.vercel.app` if `NEXT_PUBLIC_API_URL` is not set
- Dev environments without `.env.local` will silently hit the production API
- **Fix:** Remove hardcoded fallback; throw an error if `NEXT_PUBLIC_API_URL` is not set in non-production environments

### LOW — Stale README
- `README.md` is 236 bytes — just a template description + test line ("test = test")
- **Fix:** Document the project, setup steps, env vars, and architecture

---

## 5. Feature Module Status Summary

| Module | Route(s) | Status | Notes |
|---|---|---|---|
| Authentication | `/auth/sign-in`, `/auth/callback` | ✅ Complete | Supabase SSR auth, middleware guard |
| Dashboard | `/` | 🔴 Stub | 5-line placeholder; no widgets |
| Banking — Accounts | `/banking/accounts` | ✅ Complete | Full CRUD + filters |
| Banking — Transactions | `/banking/transactions` | ✅ Complete | Full lifecycle processing |
| Documents — Invoices | `/documents/invoices` | ✅ Complete | 46KB wizard, drawer, actions |
| Documents — Bills | `/documents/bills` | ✅ Complete | 33KB wizard, drawer, actions |
| Documents — Journal Entries | `/documents/journal-entries` | ✅ Complete | 27KB dialog, table, actions |
| Accounting — COA | `/accounting/coa` | ✅ Complete | List + create with hierarchy |
| Accounting — Journals | `/accounting/journals` | ✅ Complete | List + detail drawer |
| Accounting — GL Activity | `/accounting/gl-activity` | ✅ Complete | Report with filters |
| Contracts | `/contracts`, `/contracts/[id]` | ⚠️ Partial | List + detail + create/edit sheet built; billing schedule view missing |
| Billing Schedules | `/billing-schedules/contracts/[id]` | 🔴 Stub | Route exists; no content |
| Properties | `/properties`, `/properties/[id]` | ✅ Complete | List + 21KB detail |
| Properties — Spaces | `/properties/spaces` | ⚠️ Partial | May be stub |
| Parties — Tenants | `/parties/tenants` | ✅ Complete | |
| Parties — Vendors | `/parties/vendors` | ✅ Complete | |
| Organization — Legal Entities | `/organization/legal-entities` | ✅ Complete | |
| Items | `/items` | ✅ Complete | Full CRUD + realtime |
| Reports — P&L | `/reports/pl` | 🔴 Not built | Nav link 404s |
| Reports — Balance Sheet | `/reports/balance-sheet` | 🔴 Not built | Nav link 404s |
| Automation — Rules | `/automation/rules` | 🔴 Not built | Nav link 404s |
| Settings | `/settings` | 🔴 Not built | Nav link 404s |
| Support | `/support` | 🔴 Not built | Nav link 404s |

---

## 6. Architecture Overview

### Data Flow
```
Next.js Page (Server Component)
  └─ [Module]Client.tsx (Client Component)
       └─ use[Domain]Resource.ts (hook)
            └─ useInfiniteResource.ts (generic hook)
                 └─ lib/api/client.ts (fetch + Bearer token)
                      └─ sparkiq-erp-api.vercel.app (NestJS BE)
```

### Auth Flow
```
Browser Request
  → middleware.ts (proxy.ts)
      → Supabase session check
          → Authenticated: pass through + inject x-pathname header
          → Unauthenticated: redirect to /auth/sign-in?next={path}
```

### State Management
- **Server state:** custom `useInfiniteResource` + `usePaginatedQuery` (no React Query / SWR)
- **Filter state:** URL-synchronized via `useFilters` (persists on refresh, shareable links)
- **Dialog/drawer state:** `LayoutProvider` context (global) + local `useState` per component
- **Auth state:** `UserProvider` context (Supabase `onAuthStateChange`)
- **Realtime:** Supabase Realtime subscriptions dispatched as window `CustomEvent`

### Key Design Decisions
- **Route constants** centralized in `lib/routes.ts` — all nav, breadcrumbs, and quick-create items derive from this single file
- **No React Query / SWR** — custom hooks built on native `fetch`. Pro: no extra dep. Con: no caching, no background refetch, no stale-while-revalidate
- **Tailwind CSS v4** — cutting-edge, PostCSS-based. Some ecosystem tooling may not fully support v4 yet
- **React 19** — concurrent features available; ensure Suspense boundaries are in place for all async operations

---

## 7. Component Complexity Map

| Component | Size | Complexity | Risk |
|---|---|---|---|
| `ContractSheet.tsx` | 46KB | 🔴 Very High | Hardest to maintain; critical path for lease creation |
| `CreateInvoiceDialog.tsx` | 46KB | 🔴 Very High | Core billing flow; most-used creation wizard |
| `CreateBillDialog.tsx` | 33KB | 🔴 High | Core AP flow |
| `CreateJournalEntryDialog.tsx` | 27KB | 🟡 High | Manual JE entry |
| `ui/sidebar.tsx` | 21KB | 🟡 Medium | Infrastructure; rarely changes |
| `properties/[id]/...` | 21KB | 🟡 Medium | Property detail; growing complexity |
| `BankAccountDialog.tsx` | 16.7KB | 🟡 Medium | Cash account management |
| `LegalEntityDialog.tsx` | 13KB | 🟢 Medium | Entity management |

> **Note:** The three largest components (ContractSheet, CreateInvoiceDialog, CreateBillDialog) are all creation wizards over 30KB. These should be monitored for performance (bundle size) and considered for splitting into sub-components or multi-step wizard patterns.

---

## 8. Recommended Sprint Priorities

### Sprint 1 — Critical Fixes (Before any user demo or external access)
1. 🔴 Fix post-login redirect bug (`/leads/lead-matches` → `/`)
2. 🔴 Add `.env.example` with all required environment variables documented
3. 🔴 Standardize on pnpm — remove `yarn.lock` and `package-lock.json`
4. 🔴 Remove hardcoded fallback API URL; throw on missing `NEXT_PUBLIC_API_URL`
5. 🔴 Audit all `data.ts` files — replace any remaining mock data with live API calls

### Sprint 2 — Dashboard (Highest visible impact)
6. Build dashboard with key widgets:
   - Outstanding invoice total (AR balance)
   - Unpaid bills total (AP balance)
   - Recent journal entries feed
   - Properties occupancy snapshot
   - Upcoming billing events (from billing schedule)

### Sprint 3 — Complete CRE Core
7. Billing Schedules detail page (`/billing-schedules/contracts/[id]`) — show event horizon, draft events, generated documents
8. Properties Spaces dedicated list page if not complete
9. Properties financial data panel on property detail page (cap rate, NOI, acquisition cost)

### Sprint 4 — Reports Module
10. `/reports/pl` — P&L statement (date range, entity filter)
11. `/reports/balance-sheet` — Balance Sheet (as-of date, entity)
12. `/reports/rent-roll` — Rent Roll (active leases, spaces, tenants, rates)
13. `/reports/trial-balance` — Trial Balance

### Sprint 5 — Automation & Settings
14. `/automation/rules` — Rule instance management (view active billing rules per contract)
15. `/automation/events` — Financial events list with override capability
16. `/settings` — Org settings, user management, member roles

### Sprint 6 — Quality & Reliability
17. Install Vitest + React Testing Library; write tests for top 5 components
18. Add Playwright E2E tests for critical paths: login → create invoice → post
19. Add Husky + lint-staged pre-commit hooks
20. Enable GitHub branch protection on `main` (require PR review + CI pass)

---

## 9. Key Technical Debt Items

| Item | Priority | Effort | Impact |
|---|---|---|---|
| Post-login redirect default is wrong route | P0 | XS | Breaks all fresh logins |
| No `.env.example` | P0 | XS | Blocks new developer onboarding |
| Multiple lockfiles | P1 | XS | Dependency drift risk |
| Hardcoded fallback API URL | P1 | XS | Dev → prod data bleed |
| `ContractSheet.tsx` 46KB monolith | P2 | L | Maintainability + bundle size |
| `CreateInvoiceDialog.tsx` 46KB monolith | P2 | L | Same as above |
| No test suite | P2 | XL | Zero regression safety |
| No branch protection | P2 | XS | Regressions can merge directly |
| No pre-commit hooks | P3 | S | Code quality drift |
| Mock `data.ts` files (unresolved) | P2 | S | Silent fake data in production |
| No React Query / SWR (custom hooks) | P3 | XL | No caching, stale data UX |
