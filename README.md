# Tennessee SGO Management Platform

Software for Tennessee Scholarship Granting Organizations (SGOs) to run the federal Education Freedom Tax Credit (IRC §25F) end to end: accept credit-eligible gifts, award income-verified scholarships, pay schools and vendors directly, and prove compliance from one ledger.

> **Status:** pre-build. The full product spec is in [`Tennessee-SGO-Management-Platform-PRD.md`](./Tennessee-SGO-Management-Platform-PRD.md). Hard deadline: credit-eligible giving live on **January 1, 2027**.

---

## What it does

| Module | Highlights |
| --- | --- |
| **Contributions & donor credits** | Branded giving pages, ACH-first checkout, irrevocable §25F designation, anti-earmarking at checkout, unique donor numbers, per-gift receipts with the $1,700 cap and §170 double-benefit notice, Jan 31 acknowledgments, Feb 28 IRS SGO portal report |
| **Applications & eligibility** | Bilingual (English/Spanish) applications, secure document upload, 300% AMGI income ceiling check, IRS transcript verification (Form 4506-C), renewal and sibling priority, disqualified-person screening, committee dockets |
| **Award management** | Award cycles and budgets, award letters with e-acceptance, live student balances, amendments, waitlists, renewals |
| **Payments routing** | Tuition paid directly to verified schools, vendor and reimbursement requests, approval workflow with conflict-of-interest guardrails, enrollment-gated payouts, overpayment recovery |
| **School & vendor registration** | Self-service onboarding, KYB and sanctions checks, bank account verification, school portal |
| **§25F compliance** | Restricted-funds ledger for the Tennessee §25F account, 90/10 monitoring with alerts and reports, 10-student counter, annual certification, audit package, hash-chained audit log |
| **SGO dashboard & portals** | One dashboard for contributions, applications, disbursements and reports; branded portals for families, contributors and schools |

**Scope:** Tennessee only. SGOs located in and listed by the Tennessee Department of Education (TDOE), funding Tennessee-resident students. Multistate SGOs are out of scope.

---

## Architecture

A modular monolith on managed Postgres. Every module writes to one double-entry ledger and one append-only audit log; external vendors are reached only through adapters.

```
Portals (families, donors, schools, SGO staff, auditors)
        │
        ▼
Core platform
  ├── applications/     eligibility, documents, priority, committee
  ├── contributions/    gifts, designation, donor numbers, receipts
  ├── awards/           cycles, letters, balances
  ├── payments/         requests, approvals, payouts, recovery
  ├── registry/         schools and vendors, KYB, enrollment
  ├── compliance/       90% cohorts, certification, audit package
  ├── rules/            versioned §25F and Tennessee parameters
  ├── ledger/           restricted-funds double-entry ledger
  └── audit/            hash-chained event log, WORM evidence
        │
        ▼
Vendor adapters (Stripe, Modern Treasury, Plaid, Truv, Ocrolus, Middesk, Persona, IRS)
```

Design principles:

- **Ledger is the source of truth.** Nothing moves money without a ledger entry and an audit event first.
- **Rules are data.** Every statutory number and date (cap, 300% AMGI, 90%, deadlines) lives in a versioned rules registry with effective dates and a citation, so final regulations ship as a config release.
- **Evidence by reference.** Each award and payment links to its documents, rule version and approvers so an auditor can replay the decision.
- **No custody.** Funds stay in each SGO's own FDIC-insured accounts; the platform orchestrates movement through API rails.

---

## Tech stack

| Layer | Choice |
| --- | --- |
| App | Next.js + TypeScript on Vercel, Tailwind + shadcn/ui, next-intl |
| Database | Supabase Postgres (row-level security) or Neon |
| Workflows | Temporal Cloud |
| Auth | Clerk (MFA, passkeys) + WorkOS (SAML SSO) |
| Donations | Stripe Checkout + Stripe Connect (Standard), ACH via Financial Connections |
| Banking & payouts | Modern Treasury (payments, reconciliation, Ledgers) over SGO-owned accounts at Column or Increase |
| Verification | Plaid (bank), Truv/Argyle (payroll income), IVES provider (IRS transcripts), Persona (identity), Middesk (KYB) |
| Documents | Ocrolus, Claude API / AWS Textract, AWS S3 with Object Lock |
| Sensitive data | VGS or Skyflow vault |
| Messaging | Postmark, Twilio, Lob |
| Reporting | Metabase Cloud (embedded) |
| Ops | Vanta (SOC 2), Sentry + Datadog, LaunchDarkly, GitHub Actions, Terraform |

---

## Getting started

> Proposed setup; update as the codebase lands.

### Prerequisites

- Node.js 20+ and pnpm
- Docker (local Postgres and Temporal)
- Test-mode accounts for Stripe, Modern Treasury and Plaid

### Install and run

```bash
git clone <repo-url>
cd tn-sgo-platform
pnpm install
cp .env.example .env.local      # fill in test-mode keys
docker compose up -d            # Postgres + Temporal dev server
pnpm db:migrate
pnpm db:seed                    # demo SGO, donors, households, schools
pnpm dev                        # http://localhost:3000
```

### Environment variables

| Variable | Purpose |
| --- | --- |
| `DATABASE_URL` | Postgres connection string |
| `CLERK_SECRET_KEY`, `NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY` | Authentication |
| `STRIPE_SECRET_KEY`, `STRIPE_WEBHOOK_SECRET`, `STRIPE_CONNECT_CLIENT_ID` | Donations and platform fee |
| `MODERN_TREASURY_API_KEY`, `MODERN_TREASURY_ORG_ID` | Payouts, reconciliation, ledgers |
| `PLAID_CLIENT_ID`, `PLAID_SECRET` | Bank account verification |
| `TEMPORAL_ADDRESS`, `TEMPORAL_NAMESPACE` | Durable workflows |
| `S3_EVIDENCE_BUCKET` | WORM document storage |
| `VAULT_API_KEY` | Tokenizing SSNs and bank numbers |

Never commit real keys. Live keys belong in the secrets manager only.

### Tests

```bash
pnpm test           # unit tests, including rules-registry and 90% cohort math
pnpm test:e2e       # end-to-end flows: give → receipt, apply → award → payout
pnpm test:tenancy   # row-level security isolation tests
```

---

## Compliance and security

- SOC 2 Type I before the first live gift; Type II within 12 months
- PCI DSS SAQ-A: card data stays in Stripe's hosted fields
- HTTPS for all traffic; AES-256 at rest; field-level encryption for tax identifiers, bank details and payment keys
- No donor SSNs collected; applicant SSN/ITIN only with signed Form 4506-C consent, tokenized in a vault
- Role-based access, MFA for staff, schools and auditors; dual approval for payouts and bank-detail changes
- 7-year retention; append-only, hash-chained audit log

Rule changes that affect compliance (anything in `rules/`) need review from the compliance lead and a citation to the regulation or TDOE guidance.

---

## Pricing model

2% of each processed donation, capped at $25 per gift, collected as a Stripe Connect application fee. ACH is the default payment method. Software only; no setup or monthly fees.

---

## Roadmap

| Phase | Dates | Deliverable |
| --- | --- | --- |
| 0 · Launch giving | Nov–Dec 2026 | Giving pages, §25F designation, donor numbers, ledger, receipts |
| 1 · Award | Jan–Apr 2027 | Applications, AMGI and IRS transcripts, committee, school registration |
| 2 · Pay schools | May–Aug 2027 | Payment requests, approvals, payouts, enrollment checks |
| 3 · Prove it | Sep 2027–Feb 2028 | 90% tracker, certification, audit package, Jan 31 / Feb 28 filings |

---

## Key references

- [IRS: Federal Scholarship Tax Credit](https://www.irs.gov/government-entities/federal-state-local-governments/federal-scholarship-tax-credit-fstc)
- [Proposed regulations, REG-117199-25](https://www.federalregister.gov/documents/2026/10/02/2026-20277/federal-scholarship-tax-credit)
- [Temporary regulations, T.D. 10057](https://www.federalregister.gov/documents/2026/10/02/2026-20264/federal-scholarship-tax-credit)
- [TDOE: Scholarship Granting Organizations](https://www.tn.gov/education/about-the-tdoe/scholarship-granting-organizations.html)

---

## Disclaimer

This software supports §25F compliance but does not provide tax or legal advice. SGOs should confirm program rules with tax counsel.

## License

TBD.
