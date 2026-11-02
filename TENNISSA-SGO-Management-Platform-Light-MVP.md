# TENNISSA Scholarship MVP — Recommendation and PRD

Oct 30, 2026 · @Kyle Anderson

## Summary and recommendation

Build the TENNISSA MVP as one web app on **Next.js (Vercel) + Supabase + Stripe ACH + Resend**, run by a small team for roughly **$45–$80 a month** plus Stripe's per-donation fees. Ship it in four phases so donor giving and acknowledgments are live by **January 1, 2027**, when §25F credit-eligible gifts begin, and the student, review, award and disbursement modules follow in time for the 2027–28 award cycle.

**What this MVP is.** A secure, mobile-friendly app with four portals (student/parent, reviewer, donor, administrator) plus a read-only auditor view on one database. It replaces spreadsheets for TENNISSA's core workflow: apply → review → recommend → approve → disburse, with donor gifts, receipts and a financial dashboard alongside.

**Key recommendations**

1. **One managed platform, not many.** Supabase gives the database, login, role-based security, file storage and backups in one service; Vercel hosts the app with no servers to manage.
2. **ACH only, through Stripe.** Stripe holds the bank details, verifies accounts and tells the app when a pending payment settles, so acknowledgments go out only for completed gifts.
3. **Record disbursements, don't automate them yet.** Pay schools from TENNISSA's bank as today; the app records each payment, its reference and status, and tracks awarded vs. disbursed per student and school.
4. **QuickBooks by export first.** Monthly CSV exports in QuickBooks import format in the MVP; a direct QuickBooks Online sync is a phase-two upgrade.
5. **Build in the §25F basics now.** A few compliance items (designation, donor numbers, the 300% income test, priority rules, the 90/10 split, January 31 acknowledgments) cost little to include now and a lot to retrofit. They are listed in the next section.

**What it is not.** Not the full SGO Management Platform PRD: no multi-tenant SaaS, automated payouts, IRS transcript integration, digital wallets or SOC 2 audit of TENNISSA itself. The data model is designed so those can be added later without a rebuild.

## MVP scope and §25F minimums

The MVP covers the seven priorities in TENNISSA's request and nothing else, plus eight §25F items that the program requires from the first gift.

**In scope (v1)**

1. Student application and document submission
2. Reviewer access and application evaluation
3. Administrator award approval
4. Scholarship disbursement tracking
5. Donor registration and Stripe ACH donations
6. Automated donation acknowledgments (PDF + email)
7. Administrative dashboard and financial reporting

**Out of scope (later phases)**

- Automated payouts to schools (payments are made from the bank and recorded in the app)
- Card payments, recurring gifts, donor CRM campaigns
- Direct QuickBooks Online sync (CSV export in v1)
- IRS transcript and payroll income verification (documents reviewed by staff in v1)
- School self-service portal (TENNISSA staff maintain the school list in v1)
- Multi-organization (SaaS) features

**§25F minimums the MVP must include**

These come from IRC §25F and Treasury's [proposed regulations](https://www.federalregister.gov/documents/2026/10/02/2026-20277/federal-scholarship-tax-credit). Leaving them out would put donors' credits or TENNISSA's SGO status at risk.

| Requirement | What the MVP does |
| --- | --- |
| Donor designation | Donor confirms at checkout that the gift is a §25F qualified contribution; the designation is stored and cannot be changed |
| Unique donor number | Generated automatically per donor in the IRS-prescribed format once the IRS SGO portal publishes it; shown on every receipt |
| Annual acknowledgment and IRS report | Per-gift receipts plus a year-end acknowledgment by January 31 (EIN, total, donor number, $1,700 cap and no-double-benefit statement); a February 28 export of donor totals for the IRS SGO portal |
| No earmarking | Giving form has no student field; donors cannot direct gifts to a student |
| Student eligibility | Application captures household size and income; reviewer confirms household income is at or below 300% of area median gross income for the county, with documents attached |
| Priority and conflicts | Flags renewal students and siblings for priority; blocks awards to board members, reviewers, large donors and their families |
| Segregated funds and 90/10 | All §25F gifts settle to a dedicated bank account; the dashboard tracks scholarship spending against at least 90% of income and operating costs against the 10% |
| Direct payment to schools | Tuition is recorded as paid to the school, never to families |

## Users, roles and access

Five roles, each seeing only its own data; every rule is enforced in the database (row-level security), not just hidden in the screens.

| Role | Who | Can do | Can see |
| --- | --- | --- | --- |
| Applicant | Student or parent/guardian | Create account, apply, upload documents, answer info requests, view decision | Own household's applications, documents, award and balance |
| Reviewer | TENNISSA staff or volunteer | Review assigned applications, comment, score, recommend an award amount, submit | Only applications assigned to them, with their documents |
| Donor | Individual taxpayer | Manage profile, give by ACH, download receipts and year-end acknowledgments | Own profile, donor number, gifts and receipts |
| Administrator | Executive director, finance lead | Assign reviewers, approve/modify/reject awards, record disbursements and expenses, manage schools and users, run reports and exports | Everything |
| Auditor | Independent CPA, board audit committee member, or state reviewer | View records, run reports, export CSVs and evidence files, leave notes for administrators; no create, edit, approve or delete anywhere | Read-only: donations, acknowledgments, applications, reviews, awards, disbursements, schools, expenses, dashboard, audit log; documents only when the admin turns on document access for that auditor |

**Access rules**

- Administrators can be split later into **Program admin** (applications, awards) and **Finance admin** (disbursements, expenses, exports); the MVP ships one admin role with a second approver required to record any disbursement over a set amount.
- Reviewers who are related to an applicant must declare it; a declared conflict removes the assignment.
- Applicants and donors sign in with email and password or a magic link; reviewers and admins must use multi-factor authentication.
- Applicant documents are viewable only through short-lived signed links.
- **Auditor access** is granted by an administrator for a set period (for example the audit window, 30–90 days) and expires automatically; MFA is required. Every page an auditor opens and every file they export is written to the audit log. Donor bank details never appear (Stripe holds them), and family documents are hidden unless the admin enables document access for that engagement.
- **Audit sample view:** auditors can filter any period and open a single award or donation with its full trail (application, reviews, approval, disbursements, Stripe reference, acknowledgment) on one page.

## Functional requirements

Six modules, each with numbered requirements. All are MVP unless marked **Later**.

### 1. Student application portal

| ID | Requirement |
| --- | --- |
| STU-1 | Sign up and sign in with email; password reset; one account per household that can hold several students |
| STU-2 | Application form: household members and size, Tennessee address and county, household income, student grade and school, prior TENNISSA award, siblings applying |
| STU-3 | Upload documents (PDF, JPG, PNG; up to 10 MB each): income proof (tax return, W-2, pay stubs, or SNAP/TANF/WIC/SSI letter), proof of Tennessee residence, school acceptance or enrollment |
| STU-4 | Save draft and submit; application locks on submission |
| STU-5 | Status page: Draft, Submitted, Under review, Info requested, Recommended, Approved, Declined |
| STU-6 | Email when more information is requested, with a link to upload; status returns to Under review after response |
| STU-7 | View final decision, award amount, school and payments made once approved |
| STU-8 | Mobile-friendly; English first, Spanish **Later** |

**STU-9 · Automatic student folders.** Every upload is filed by the app into the right folder, with no manual sorting. Household-wide proof (income, residency) goes to a `shared` folder used by all siblings; each student gets their own folder with one subfolder per application cycle; award letters generated on approval land in the same place. Files are renamed on upload to a standard name (document type + date), and the original file name is kept in the database.

```
applicant-documents/
  household-{household_id}/
    shared/                      income proof, residency proof
    student-{student_id}/
      2027-28/                   school acceptance, info-request replies, award letter
      2028-29/
```

### 2. Application review portal

| ID | Requirement |
| --- | --- |
| REV-1 | Reviewer sees only applications an administrator assigned to them |
| REV-2 | Side-by-side view of application answers and uploaded documents |
| REV-3 | Eligibility checklist: Tennessee resident, household income ≤ 300% of the county limit (income table loaded by admin), required documents present, not a disqualified person |
| REV-4 | Request more information from the family (sends STU-6 email) |
| REV-5 | Comments, a score against TENNISSA's criteria, a recommended award amount and a recommend/decline choice |
| REV-6 | Submit recommendation to the administrator; editable until the admin acts |
| REV-7 | Conflict declaration on each assigned application |

### 3. Scholarship award management

| ID | Requirement |
| --- | --- |
| AWD-1 | Admin list of recommended applications, filterable by status, school, county and reviewer |
| AWD-2 | Compare multiple reviewers' recommendations and amounts side by side, with comments |
| AWD-3 | Approve, decline, or modify the amount; a reason is required when modifying or declining |
| AWD-4 | Priority flags: renewal student, sibling of a recipient |
| AWD-5 | Show funds available to award before approving, so awards cannot exceed available scholarship funds |
| AWD-6 | Record final decision, approver and date; email the family |
| AWD-7 | Award history per student across years |

### 4. Scholarship disbursement management

| ID | Requirement |
| --- | --- |
| DIS-1 | School list: name, address, EIN, contact, remittance details (kept by admins) |
| DIS-2 | Create a disbursement against an approved award: school, student, amount, term, method (ACH from bank, check), planned date |
| DIS-3 | Record payment reference (bank trace number or check number), paid date and status: Scheduled, Sent, Cleared, Returned, Voided |
| DIS-4 | Second-admin approval for disbursements above a set amount; disbursements cannot exceed the award balance |
| DIS-5 | Per student: awarded, disbursed, remaining balance; per school: totals by term |
| DIS-6 | Every disbursement links back to the award, the student, the school and the scholarship fund it drew from |
| DIS-7 | Disbursement report and CSV export by date range, school and student |

### 5. Donor portal and Stripe ACH donations

| ID | Requirement |
| --- | --- |
| DON-1 | Donor sign-up and profile (name, address, email, phone). Spouses: one login can hold two donor records. "Add spouse" creates a linked donor with their own donor number, folder and acknowledgments; at checkout the donor picks who the gift is from (me or my spouse; a "split between us" option that creates two separate gifts is switched on once counsel confirms joint-account treatment); either spouse can later be invited to their own login |
| DON-2 | Unique donor ID assigned automatically on sign-up (DON-YYYY-NNNNN internally, mapped to the IRS donor-number format when published) |
| DON-3 | Give by ACH only, through Stripe Payment Element with bank verification; §25F designation checkbox and disclosures; no student field |
| DON-4 | Gift status visible to donor: Pending (ACH processing), Completed, Failed, Refunded |
| DON-5 | On settlement (Stripe payment succeeded), generate a PDF acknowledgment, store it on the donor's profile, and email it |
| DON-6 | Pending gifts get a "we received your gift" email only, never an acknowledgment |
| DON-7 | Donation history with downloadable receipts; year-end acknowledgment PDF by January 31 |
| DON-8 | Running total for the year against the $1,700 federal credit cap, shown per spouse side by side when a spouse is linked |
| DON-9 | Permanent transaction record; failed or refunded gifts are kept and marked, never deleted |

**DON-10 · Automatic donor folders.** Each donor gets a dedicated folder with one subfolder per tax year. The app saves a receipt there when each gift settles (pending ACH gifts get an email only, no file) and adds the year-end acknowledgment by January 31. Refunded or disputed gifts keep their receipt, stamped VOID; a corrected year-end acknowledgment is saved beside the original. Spouses have separate folders because each has their own donor number and $1,700 cap. Donors see the folder as **My documents** and can download any file at any time.

```
donor-documents/
  donor-{donor_number}/
    2027/
      gift-2027-03-14.pdf          receipt for each completed gift
      gift-2027-09-02.pdf
      year-end-acknowledgment-2027.pdf   issued by Jan 31, 2028
    2028/
```

### 6. Administrative dashboard and reporting

| ID | Requirement |
| --- | --- |
| ADM-1 | Donation summary: total received (settled), pending ACH total, totals by donor, transaction list |
| ADM-2 | Scholarship summary: funds available, awards approved, disbursed, remaining; by school and by student |
| ADM-3 | Operating expenses: record expenses with category (administrative, fundraising, other), amount, date, vendor, reference |
| ADM-4 | Funds remaining after disbursements and expenses |
| ADM-5 | 90/10 tracker: scholarship spending vs. 90% of income received, and operating costs vs. 10% |
| ADM-6 | Exports (CSV): donations, disbursements, expenses, awards; QuickBooks import format |
| ADM-7 | Year-end exports: donor totals for the IRS SGO portal (by February 28) and annual report data |
| ADM-8 | Audit log view of administrative actions; auditor user management (invite, set access window and document access, revoke) |

**ADM-9 · Student file page.** One page per student listing everything in their folders (shared household documents, each year's documents, award letters) alongside the award and disbursement history, with a **Download all (ZIP)** button for audits and records requests. Folders are not copied to Google Drive or SharePoint, so sensitive documents exist in one place only.

**ADM-10 · Donor file page.** One page per donor showing their profile, donor number, gift history by year, and every receipt and acknowledgment in their folder, with a **Download all (ZIP)** button for audits and donor requests.

## Recommended technology stack and architecture

One Next.js web app on Vercel, backed by Supabase for data, login and files; Stripe, Resend and QuickBooks connect at the edges. Nothing to patch, no servers, one bill per service.

&#91;embedded content: MVP architecture · 4 user groups, 1 app, 5 managed services\]

Every request passes through the app's role check and the database's row-level security; dashed lines are file exports and manual bank payments, not live connections.

| Layer | Choice | Why it fits TENNISSA | Alternative |
| --- | --- | --- | --- |
| Web app | Next.js + TypeScript, Tailwind CSS | One codebase for all four portals; mobile-friendly; large hiring pool | Remix |
| Hosting | Vercel Pro | Serverless, automatic HTTPS and deploys, no servers | Netlify |
| Database | Supabase Postgres | Relational data fits awards, schools and payments; row-level security; SQL reports | Firebase (weaker for financial reporting joins) |
| Authentication | Supabase Auth | Email login, password reset, magic links, MFA, roles tied to database rules | Clerk |
| File storage | Supabase Storage (private buckets) | Same security rules as the database; signed links | AWS S3 |
| Payments | Stripe ACH Direct Debit + Financial Connections | Bank data stays at Stripe; settlement webhooks drive receipts | None needed |
| Email | Resend | Simple API, verified sending domain, delivery logs | Postmark |
| PDF receipts | React-PDF in the app | No extra service or cost | PDF service API |
| Scheduled jobs | Vercel Cron | Year-end acknowledgments, exports, reminders | Supabase scheduled functions |
| Error alerts | Sentry | Free tier covers the MVP | Vercel logs only |
| Accounting | QuickBooks Online (existing) | CSV import now, API sync later | — |

**Why Supabase over Firebase:** scholarship data is relational (a disbursement belongs to an award, a student, a school and a fund), and the dashboard is mostly sums across those links. Postgres handles that in plain SQL; Firebase's document model makes those reports harder and costlier to build.

**Modular by design:** each module is its own folder and set of tables, so later features (school portal, automated payouts, QuickBooks API sync, IRS transcript checks) are added beside the MVP rather than rebuilt into it.

## Database design and relationships

Seventeen tables in Supabase Postgres (13 drawn below, plus helpers and `profiles` and `audit_log`); the award is the hinge that ties a student's application to a school, a fund and the payments made.

&#91;embedded content: data model · 13 core tables, 3 groups\]

A disbursement always points to its award, school and fund, so awarded vs. disbursed, balances by school and the 90/10 split are simple sums.

| Table | Key fields | Relationships |
| --- | --- | --- |
| profiles | user ID, name, email, phone, role (applicant, reviewer, donor, admin, auditor), access expiry for auditors, MFA on | One per login; owns households, reviews or donor records by role |
| households | address, county, household size, reported income, income check result | Belongs to an applicant profile; has many students |
| students | name, date of birth, grade, current/intended school, renewal flag, sibling flag | Belongs to a household; has many applications and awards |
| applications | cycle (e.g. 2027–28), status, submitted date, eligibility checklist | Belongs to a student; has many documents and reviews; at most one award |
| documents | type, folder (household shared or student + cycle), storage path, original file name, uploaded date, verified by | Belongs to an application |
| reviewer\_assignments | reviewer, application, assigned date, conflict declared | Links reviewers to the applications they may see |
| reviews | score, comments, recommendation, recommended amount, submitted date | Belongs to an application and a reviewer |
| awards | amount, status, approver, approved date, reason if modified, school year | One per approved application; belongs to a student, a school and a fund; has many disbursements |
| schools | name, EIN, address, contact, remittance details, active | Has many awards and disbursements |
| disbursements | amount, term, method, reference, planned/paid date, status, approved by | Belongs to an award, a school and a fund |
| donors | donor number, name, address, email, spouse link, account owner (the login that manages this donor) | Belongs to a donor profile; has many donations |
| donations | amount, §25F designation, Stripe PaymentIntent ID, status, settled date, Stripe fee | Belongs to a donor and a fund; has acknowledgments |
| acknowledgments | type (gift or year-end), folder (donor + tax year), PDF path, issued date, emailed date, void flag, replaced-by link for corrected acknowledgments | Belongs to a donation (or a donor and year for year-end) |
| funds | name (§25F scholarship, operating), bank account label | Has many donations, disbursements and expenses |
| expenses | category, amount, date, vendor, reference, fund | Belongs to the operating fund |
| income\_limits | county, household size, 300% limit, year | Read by the eligibility check |
| audit\_log | actor, action, table, record, before, after, time | Written by database triggers; read-only |

**Totals the dashboard computes**

- Funds available = settled §25F donations − approved awards not yet disbursed − disbursements
- Student balance = award amount − sum of disbursements (Cleared or Sent)
- 90/10 = disbursements ÷ settled §25F income, and operating expenses ÷ income

## Security architecture and TENNISSA's responsibilities

The cloud providers secure the infrastructure; TENNISSA still owns who gets access, what data it collects, and how it responds when something goes wrong.

**Built-in controls**

| Control | How it is implemented |
| --- | --- |
| Encryption in transit | HTTPS everywhere (Vercel and Supabase enforce TLS); HSTS on the app domain |
| Encryption at rest | Supabase encrypts the database and file storage (AES-256); Stripe stores all bank data |
| Authentication | Supabase Auth: email + password with reset, magic links; MFA (authenticator app) required for reviewers and admins |
| Authorization | Row-level security policies on every table: applicants see own household, reviewers see assigned applications, donors see own gifts, admins see all, auditors read all within their access window and change nothing |
| Documents | Private storage bucket organized as one folder per household and per student; storage policies check the folder path, so families reach only their household folder, reviewers only assigned students' folders, admins all, auditors only with document access on; donor receipts sit in a separate bucket, one folder per donor, open to that donor, admins, and auditors by default; files opened only through signed links that expire in minutes; file type and size checks on upload |
| Payments | Stripe Payment Element and Financial Connections; the app stores only Stripe IDs and the bank's last four digits, never account or routing numbers |
| Audit log | Database triggers record who changed awards, disbursements, expenses, school bank details and user roles, with before/after values; log rows cannot be edited |
| Backups | Supabase Pro daily backups kept 7 days; monthly export of the database and documents to TENNISSA-owned storage |
| Secrets | Stripe, Resend and Supabase service keys kept in Vercel environment settings, never in code or the browser |
| Monitoring | Error tracking (Sentry free tier) and Stripe webhook failure alerts emailed to the admin |

**Data minimization**

- Collect no Social Security numbers in forms. Ask families to black out SSNs on tax returns and W-2s before uploading.
- Keep only what eligibility and IRS reporting need; set a retention period (for example 7 years for financial records, 3 years after the last award for application documents) and delete on schedule.

**What TENNISSA must do itself**

- [ ] Name a data owner and review user access every quarter; remove reviewers and staff the day they leave
- [ ] Require MFA on the Supabase, Vercel, Stripe, Resend and QuickBooks admin accounts, owned by TENNISSA (not a contractor's personal account)
- [ ] Publish a privacy notice for families and donors and get consent at sign-up
- [ ] Sign each vendor's data processing terms and keep copies
- [ ] Write a one-page incident response plan, including Tennessee data-breach notification duties
- [ ] Train reviewers on handling family documents; no downloads to personal devices
- [ ] Test a backup restore once before launch and once a year

## Stripe ACH and QuickBooks integration

Stripe tells the app when an ACH gift settles; the app issues the acknowledgment only then. QuickBooks gets monthly exports in v1 and a direct sync later.

### Stripe ACH donation flow

1. Donor signs in, enters an amount, ticks the §25F designation box and accepts the ACH debit authorization shown by Stripe.
2. Donor links a bank through Stripe Financial Connections (instant) or enters account details for micro-deposit verification.
3. The app creates a Stripe PaymentIntent (ACH Direct Debit) with the donor ID and donation ID in its metadata, and records the donation as **Pending**.
4. Stripe sends a `payment_intent.processing` webhook; the app emails "we received your gift, it is processing."
5. About 4 business days later Stripe sends `payment_intent.succeeded`: the app marks the gift **Completed**, generates the PDF acknowledgment, saves it to the donor's profile and emails it.
6. If Stripe sends `payment_intent.payment_failed`, the gift is marked **Failed** and the donor is asked to try again; no acknowledgment is issued.
7. Refunds and disputes (`charge.refunded`, `charge.dispute.created`) mark the gift and void its acknowledgment; a corrected year-end acknowledgment reflects the change.

**Setup notes**

- One Stripe account for TENNISSA, with payouts going only to the dedicated §25F scholarship bank account; operating costs are paid from a separate account.
- Standard ACH settlement (not 2-day) to keep the fee at 0.8% capped at $5.
- Webhooks are verified with Stripe's signing secret and processed once per event ID, so a repeated webhook never creates a second receipt.
- Stripe's bank-verification and ACH mandate screens handle authorization; TENNISSA's app never sees full account numbers.

### QuickBooks approach

| Option | What it does | When |
| --- | --- | --- |
| CSV export (MVP) | Monthly files in QuickBooks Online import format: donations (deposits by settlement date, net of Stripe fees), disbursements (payments to schools), and expenses recorded in the app | Launch |
| Stripe-to-QuickBooks sync app | Third-party connector posts Stripe payouts and fees to QuickBooks automatically | If month-end reconciliation takes more than a few hours |
| QuickBooks Online API sync | App posts journal entries for donations, awards and disbursements directly, with the scholarship fund and operating fund kept apart | Phase 2, once volumes justify it |

In QuickBooks, track the §25F scholarship fund and operating funds separately (classes or separate bank accounts) so the books match the app's 90/10 numbers. QuickBooks stays the system of record for the general ledger; the app is the system of record for donors, awards and disbursements.

## Implementation phases and timeline

Four phases over about eight months, each usable on its own: donations first because January 1, 2027 is the only fixed deadline, then the scholarship cycle in the order TENNISSA will run it.

&#91;embedded content: MVP roadmap · 4 phases, 4 gates\]

Phase 1 has seven weeks, so it holds only what donors need on day one; the final two weeks of December are for testing with real ACH gifts of a few dollars.

| Phase | Weeks | Effort (developer-weeks) | Done when |
| --- | --- | --- | --- |
| 1 · Donations live | 7 | 7–8 | A test donor gives by ACH, sees Pending, then gets the PDF acknowledgment by email after settlement |
| 2 · Apply, review | 8 | 7–8 | A family submits a full application with documents; an assigned reviewer recommends an amount |
| 3 · Award and pay | 8 | 6–7 | Admin approves an award, records a school payment, and the student balance updates |
| 4 · Report, harden | 9 | 5–6 | Dashboard totals match QuickBooks for a test month; backup restore and access review completed |

**Team:** one experienced full-stack developer (Next.js and Supabase) full time, plus about 10% of a designer for screens and receipts and a TENNISSA staff member for testing each phase. Total about **25–29 developer-weeks**. A contractor at $75–$125 an hour would put the build at roughly $75,000–$145,000; a volunteer or in-house developer changes that, not the stack.

**After the MVP (phase 2 platform):** school self-service portal, QuickBooks Online API sync, automated school payouts, Spanish, IRS transcript checks, and the year-end acknowledgment and IRS export run for the first time in January–February 2028.

## Estimated monthly costs

Fixed platform costs run about **$45–$80 a month**; Stripe adds a few dollars per donation. Prices are list prices checked October 2026; confirm before signing up.

| Service | Plan | Monthly cost | Notes |
| --- | --- | --- | --- |
| Vercel (hosting) | Pro, 1 developer seat | $20 | Hobby is personal, non-commercial only ([Vercel pricing](https://dev.to/nayankyada/vercel-pricing-2026-what-you-actually-pay-for-a-real-nextjs-project-33md)) |
| Supabase (database, auth, storage, backups) | Pro | $25 + small usage | Includes daily backups; Team plan at $599/month adds SOC 2 reports and SSO, not needed for the MVP ([Supabase pricing](https://www.cloudzero.com/blog/supabase-pricing/)) |
| Resend (email) | Free, then Pro | $0–$20 | Free covers 3,000 emails/month at 100/day; Pro is $20 for 50,000/month, needed for January year-end acknowledgments ([Resend pricing](https://superdupr.com/blog/resend-pricing)) |
| Sentry (error alerts) | Developer (free) | $0 | Upgrade only if alerts outgrow it |
| PDF generation | Open-source library in the app | $0 | No separate service |
| QuickBooks Online | TENNISSA's existing plan | $0 added | Class tracking may need a higher QuickBooks tier |
| Domain and DNS | Existing or new | \~$2 | About $20–$25 a year |
| **Total fixed** |  | **\~$45–$80** | Plus usage overages if volumes grow well beyond the MVP |

**Stripe per-donation fees (no monthly fee)**

| Fee | Amount |
| --- | --- |
| ACH Direct Debit, standard settlement | 0.8%, capped at $5 per gift |
| Instant bank verification (Financial Connections) | $1.50 per verification |
| Failed ACH payment | $4 |
| Disputed ACH payment | $15 |

A $1,700 gift costs $5 plus a one-time $1.50 bank verification for a new donor ([Stripe ACH fees](https://superdupr.com/blog/ach-fees)).

**Not included:** development labor (see the timeline), legal review of donor disclosures, and the annual §25F audit, which are TENNISSA's separate budget lines.

## Risks, open questions and sources

| Risk | Impact | Mitigation |
| --- | --- | --- |
| Phase 1 not live by January 1, 2027 | Early-year §25F gifts missed | Phase 1 holds donations and acknowledgments only; use Stripe's hosted components rather than custom screens |
| IRS donor-number format or SGO portal file format published late | Receipts or the February 28 report need rework | Internal donor ID now, mapped to the IRS format when released; export built as a separate, swappable file |
| ACH returns after acknowledgment | Wrong receipt in a donor's hands | Acknowledge only on `succeeded`; void and reissue on later returns or disputes |
| Income documents contain SSNs and sensitive data | Privacy exposure | Ask families to redact SSNs; private storage, signed links, MFA for reviewers, retention schedule |
| One developer or contractor holds all knowledge | Stalled support | TENNISSA owns every account; code in a TENNISSA GitHub organization; a short runbook in the repo |
| §25F rules change when regulations are finalized | Feature changes | Keep limits (300%, $1,700, 90%) in a settings table, not hard-coded |

**Open questions**

- [ ] Is TENNISSA approved, or applying, for the Tennessee SGO list, and is its 501(c)(3) status final or pending?
- [ ] Which bank holds the dedicated §25F scholarship account, and who can send ACH payments to schools from it?
- [ ] What award criteria and scoring should reviewers use beyond the income test?
- [ ] Will scholarships cover only tuition at schools, or also tutoring and other qualified expenses in the first year?
- [ ] Who builds it: an in-house developer, a contractor, or an agency? (Changes the timeline's labor cost, not the stack.)
- [ ] Which QuickBooks Online plan does TENNISSA use, and does it have class tracking?
- [ ] Should the donor portal also accept non-§25F general gifts, or §25F gifts only? (General gifts would need a separate bank account and receipt wording.)
- [ ] Can a gift from a joint bank account be designated as one spouse's gift, or split between spouses? The proposed rules treat spouses as separate taxpayers but don't address joint accounts; until counsel confirms, "split between us" stays off and each spouse gives from their own bank account or chooses with a short disclosure.

**Sources**

- [Federal Register: proposed §25F regulations (Oct 2, 2026)](https://www.federalregister.gov/documents/2026/10/02/2026-20277/federal-scholarship-tax-credit)
- [IRS: Federal Scholarship Tax Credit](https://www.irs.gov/government-entities/federal-state-local-governments/federal-scholarship-tax-credit-fstc)
- [TDOE: Scholarship Granting Organizations](https://www.tn.gov/education/about-the-tdoe/scholarship-granting-organizations.html)
- [Stripe: US bank payments](https://stripe.com/payments/us-bank-payments)
- [Superdupr: Stripe ACH fees 2026](https://superdupr.com/blog/ach-fees)
- [CloudZero: Supabase pricing](https://www.cloudzero.com/blog/supabase-pricing/)
- [DEV: Vercel pricing 2026](https://dev.to/nayankyada/vercel-pricing-2026-what-you-actually-pay-for-a-real-nextjs-project-33md)
- [Superdupr: Resend pricing 2026](https://superdupr.com/blog/resend-pricing)

Not tax or legal advice; confirm §25F requirements with TENNISSA's tax counsel.
