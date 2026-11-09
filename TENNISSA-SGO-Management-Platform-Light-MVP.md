# TENNISSA Scholarship MVP — Recommendation and PRD

Oct 30, 2026 · @Kyle Anderson

## Summary and recommendation

Build the TENNISSA MVP as one web app on **Next.js (Vercel) + Supabase + Stripe ACH and Connect + Resend + QuickBooks Online**, run by a small team for roughly **$120–$155 a month** plus Stripe's per-gift and per-payout fees. Ship it in four phases so donor giving and acknowledgments are live by **January 1, 2027**, when §25F credit-eligible gifts begin, and the student, review, award and disbursement modules follow in time for the 2027–28 award cycle.

**What this MVP is.** A secure, mobile-friendly app with five portals (student/parent, reviewer, donor, partner school, administrator) plus a read-only auditor view on one database. It replaces spreadsheets for TENNISSA's core workflow: apply → review → recommend → approve → disburse, with donor gifts, receipts and a financial dashboard alongside.

**Key recommendations**

1. **One managed platform, not many.** Supabase gives the database, login, role-based security, file storage and backups in one service; Vercel hosts the app with no servers to manage.
2. **ACH only, through Stripe.** Stripe holds the bank details, verifies accounts and tells the app when a pending payment settles, so acknowledgments go out only for completed gifts.
3. **Let schools onboard themselves; pay them through Stripe Connect.** Schools register, sign the MOU through Dropbox Sign and connect a bank account that Stripe verifies; the app sends each disbursement as a Stripe transfer and tracks awarded vs. disbursed per student and school.
4. **QuickBooks Online is the books.** The app posts every money event to QuickBooks as a balanced journal entry; bank reconciliation, financial statements and the outside CPA's login live there, and the app adds the 90% tracker and an examiner view.
5. **Build in the §25F basics now.** A few compliance items (designation, donor numbers, the 300% income test, priority rules, the 90/10 split, January 31 acknowledgments) cost little to include now and a lot to retrofit. They are listed in the next section.

**What it is not.** Not the full SGO Management Platform PRD: no multi-tenant SaaS, IRS transcript integration, digital wallets or SOC 2 audit of TENNISSA itself. The data model is designed so those can be added later without a rebuild.

## MVP scope and §25F minimums

The MVP covers the seven priorities in TENNISSA's request plus partner school registration and books, plus eight §25F items that the program requires from the first gift.

**In scope (v1)**

1. Student application and document submission
2. Reviewer access and application evaluation
3. Administrator award approval
4. Scholarship disbursement tracking
5. Donor registration and Stripe ACH donations
6. Automated donation acknowledgments (PDF + email)
7. Administrative dashboard and financial reporting
8. Partner school registration, MOU, verified bank account, students by school year and payments received
9. Books: double-entry ledger, bank reconciliation, reports, 90% ratio and examiner view

**Out of scope (later phases)**

- Card payments, recurring gifts, donor CRM campaigns
- IRS transcript and payroll income verification (documents reviewed by staff in v1)
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

Six roles, each seeing only its own data; every rule is enforced in the database (row-level security), not just hidden in the screens.

| Role | Who | Can do | Can see |
| --- | --- | --- | --- |
| Applicant | Student or parent/guardian | Create account, apply, upload documents, answer info requests, view decision | Own household's applications, documents, award and balance |
| Reviewer | TENNISSA staff or volunteer | Review assigned applications, comment, score, recommend an award amount, submit | Only applications assigned to them, with their documents |
| Donor | Individual taxpayer | Manage profile, give by ACH, download receipts and year-end acknowledgments | Own profile, donor number, gifts and receipts |
| Administrator | Executive director, finance lead | Assign reviewers, approve/modify/reject awards, record disbursements and expenses, approve partner schools, confirm school bank changes, close periods, manage users, run reports and exports | Everything |
| School administrator | Head of school or business office at a partner school | Register the school, sign the MOU, connect a bank account through Stripe, enter tuition, verify enrollment, report withdrawals, invite one more school user | Own school's profile, applicants who named the school (name and grade only), awarded students by school year, payments received; never household income or documents |
| Auditor | Independent CPA, board audit committee member, or state reviewer | View records, run reports, export CSVs and evidence files, leave notes for administrators; no create, edit, approve or delete anywhere | Read-only: donations, acknowledgments, applications, reviews, awards, disbursements, schools and their MOUs, expenses, books (journal, trial balance, reconciliations, 90% workpaper), dashboard, audit log; documents only when the admin turns on document access for that auditor |

**Access rules**

- Administrators can be split later into **Program admin** (applications, awards) and **Finance admin** (disbursements, expenses, exports); the MVP ships one admin role with a second approver required to record any disbursement over a set amount.
- Reviewers who are related to an applicant must declare it; a declared conflict removes the assignment.
- Applicants and donors sign in with email and password or a magic link; reviewers and admins must use multi-factor authentication.
- Applicant documents are viewable only through short-lived signed links.
- **Auditor access** is granted by an administrator for a set period (for example the audit window, 30–90 days) and expires automatically; MFA is required. Every page an auditor opens and every file they export is written to the audit log. Donor bank details never appear (Stripe holds them), and family documents are hidden unless the admin enables document access for that engagement.
- **Audit sample view:** auditors can filter any period and open a single award or donation with its full trail (application, reviews, approval, disbursements, Stripe reference, acknowledgment) on one page.

## Functional requirements

Eight modules, each with numbered requirements. All are MVP unless marked **Later**.

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

### 7. Partner school portal

Schools register themselves, sign TENNISSA's MOU and connect a verified bank account through third-party services, so TENNISSA never handles a school's bank numbers or a paper signature.

| ID | Requirement |
| --- | --- |
| SCH-1 | **Create account.** A school registers from TENNISSA's "Become a partner school" page or an emailed invitation; the first user becomes the school administrator; MFA required; the administrator can invite a second user (e.g. business office) |
| SCH-2 | **School profile.** Legal name, name families know, EIN, TDOE school category and ID, accreditation, physical address and county, grades served, enrollment, head of school, finance contact, website, non-discrimination statement on file |
| SCH-3 | **Tuition schedule** per school year: tuition and mandatory fees by grade, so awards can be capped at actual cost |
| SCH-4 | **Accept the MOU.** TENNISSA's partner MOU is sent through Dropbox Sign to the school's authorized signer; the executed PDF and signature audit trail are saved to the school's folder; the MOU version is recorded, and a new version or a new school year triggers re-signing |
| SCH-5 | **Bank account for disbursement.** The school completes Stripe Connect onboarding (hosted by Stripe): Stripe verifies the school's identity and EIN, screens it, and verifies the bank account. The app stores only the Stripe account ID and the last four digits |
| SCH-6 | **Bank change hold.** When Stripe reports a new or changed bank account, payments to that school are held until a TENNISSA administrator confirms the change by calling the school at a number already on file, and records how it was confirmed |
| SCH-7 | **Approval.** Status: Registered → Under review → Approved → Suspended. An administrator approves only when the checklist is complete: on the TDOE list, MOU signed, Stripe payouts enabled, tuition entered |
| SCH-8 | **Enrollment verification.** The school sees applicants who named it and confirms enrollment and tuition with one click, or declines with a reason; it never sees household income or documents |
| SCH-9 | **Students by school year.** A tab per school year lists each awarded student: name, grade, award amount, amount disbursed, each disbursement date, remaining balance and status, with year totals and CSV export |
| SCH-10 | **Payments received.** Every payment with its date, amount, Stripe transfer reference, the students it covers and their amounts, and status (Sent, Paid, Returned); downloadable remittance advice (PDF and CSV) to match the school's bank statement |
| SCH-11 | **Withdrawals and refunds.** The school reports a student who withdraws; the app calculates the unused amount owed back under the MOU and tracks the refund until received |

Changes to module 4: schools come from this portal instead of DIS-1's manual list, and DIS-2's default payment method becomes a Stripe transfer to the school's connected account, with check as the fallback. The disbursement record keeps the Stripe transfer ID as its reference.

### 8. Books and examiner view

QuickBooks Online is the official general ledger and the place bank reconciliation happens; the app writes every money event to it as a balanced journal entry and keeps an identical, append-only copy so an examiner can trace any number back to its source.

| ID | Requirement |
| --- | --- |
| BK-1 | **Chart of accounts by fund.** Two funds kept apart in both the app and QuickBooks (as classes): the §25F scholarship fund and the operating fund. Accounts include §25F bank, operating bank, Stripe clearing, contributions received, Stripe fees, scholarships paid, school refunds receivable, operating expenses |
| BK-2 | **Double-entry journal.** Every money event creates one journal entry whose debits equal credits (enforced by a database check): gift settled, Stripe fee, ACH return or dispute, Stripe payout to the bank, transfer to a school, school refund, expense. Entries are append-only; a mistake is fixed by a reversing entry with a reason |
| BK-3 | **Post to QuickBooks.** Each entry is posted to QuickBooks Online through its API with the app's entry ID in the memo; the app shows Posted, Pending or Failed with retry, and a nightly check confirms app and QuickBooks balances agree by account and class |
| BK-4 | **Stripe reconciliation (automatic).** Daily, the app matches each Stripe payout and transfer to the gifts, fees and school payments inside it, using Stripe's balance transaction data; anything unmatched is listed for an administrator |
| BK-5 | **Bank reconciliation.** QuickBooks bank feeds pull both bank accounts; each Stripe payout lands as one deposit that matches the app's payout entry. TENNISSA's bookkeeper reconciles each account monthly in QuickBooks; the app shows each month as Reconciled, In progress or Not started, with the statement balance and who signed off |
| BK-6 | **Period close.** After reconciliation an administrator closes the month: the app blocks back-dated entries and sets the QuickBooks closing date with a password |
| BK-7 | **Reports.** From QuickBooks: statement of financial position and statement of activities by fund. From the app: donations by period, scholarships by school and school year, students and schools served, expenses by category, and annual report data for the IRS SGO portal and the State of Tennessee; each as CSV and PDF |
| BK-8 | **90% ratio.** Per tax year: scholarships paid to schools (net of refunds) ÷ §25F income received (net of returns), shown against the 90% target, with the dollar amount still to be paid out to reach it and a warning when the year is behind pace. The exact formula and timing rule are settings, confirmed with TENNISSA's CPA once the regulations are final |
| BK-9 | **Examiner view.** The Auditor role gains a Books area: trial balance, journal, reconciliation status, 90% workpaper and reports for any closed period. Each entry opens its full trail (gift → Stripe ID → receipt PDF, or disbursement → award → application → school MOU). Read-only, time-limited, every view and export logged |
| BK-10 | **Audit package.** One click builds a ZIP for a fiscal year: journal and postings, trial balance, donations, acknowledgments, awards, disbursements, school list with MOUs, reconciliations, 90% workpaper, and QuickBooks financial statements |
| BK-11 | **Examiner access in QuickBooks.** The outside CPA is invited as an Accountant user in QuickBooks, the standard way auditors work, alongside the app's Auditor login |

**Example posting rules**

| Event | Debit | Credit |
| --- | --- | --- |
| $1,700 §25F gift settles, $5 Stripe fee | Stripe clearing $1,695; Stripe fees $5 | Contributions received, §25F $1,700 |
| Stripe pays out to §25F bank | §25F bank | Stripe clearing |
| $3,000 transfer to a school | Scholarships paid, §25F | Stripe clearing |
| ACH gift returned | Contributions received, §25F | Stripe clearing |

## Recommended technology stack and architecture

One Next.js web app on Vercel, backed by Supabase for data, login and files; Stripe, Resend and QuickBooks connect at the edges. Nothing to patch, no servers, one bill per service.

&#91;embedded content: MVP architecture · 4 user groups, 1 app, 5 managed services\]

Every request passes through the app's role check and the database's row-level security; dashed lines are file exports and manual bank payments, not live connections.

| Layer | Choice | Why it fits TENNISSA | Alternative |
| --- | --- | --- | --- |
| Web app | Next.js + TypeScript, Tailwind CSS | One codebase for all five portals; mobile-friendly; large hiring pool | Remix |
| Hosting | Vercel Pro | Serverless, automatic HTTPS and deploys, no servers | Netlify |
| Database | Supabase Postgres | Relational data fits awards, schools and payments; row-level security; SQL reports | Firebase (weaker for financial reporting joins) |
| Authentication | Supabase Auth | Email login, password reset, magic links, MFA, roles tied to database rules | Clerk |
| File storage | Supabase Storage (private buckets) | Same security rules as the database; signed links | AWS S3 |
| Payments | Stripe ACH Direct Debit + Financial Connections | Bank data stays at Stripe; settlement webhooks drive receipts | None needed |
| School payouts | Stripe Connect, Express accounts | Stripe verifies each school and its bank account and sends the money; TENNISSA never holds school bank numbers | Plaid bank verification + ACH from TENNISSA's bank |
| E-signature | Dropbox Sign API | Binding MOU signatures with an audit trail; webhook saves the executed PDF | DocuSign |
| Email | Resend | Simple API, verified sending domain, delivery logs | Postmark |
| PDF receipts | React-PDF in the app | No extra service or cost | PDF service API |
| Scheduled jobs | Vercel Cron | Year-end acknowledgments, exports, reminders | Supabase scheduled functions |
| Error alerts | Sentry | Free tier covers the MVP | Vercel logs only |
| Accounting | QuickBooks Online (existing) | Official general ledger: the app posts journal entries through the API; bank feeds, reconciliation, statements and CPA access | — |

**Why Supabase over Firebase:** scholarship data is relational (a disbursement belongs to an award, a student, a school and a fund), and the dashboard is mostly sums across those links. Postgres handles that in plain SQL; Firebase's document model makes those reports harder and costlier to build.

**Modular by design:** each module is its own folder and set of tables, so later features (school portal, automated payouts, QuickBooks API sync, IRS transcript checks) are added beside the MVP rather than rebuilt into it.

### Third-party services for compliance

Each regulated or audit-sensitive job goes to a vendor that already carries that duty, so TENNISSA's own code holds no bank numbers, no signatures and no official ledger.

| Duty | Service | What the vendor takes on | Cost |
| --- | --- | --- | --- |
| Donor bank payments | Stripe ACH Direct Debit (existing) | Bank verification, ACH mandate, bank data storage, returns | 0.8%, capped at $5 per gift |
| School identity, bank account and payouts | Stripe Connect (Express accounts) | Business verification and EIN check, sanctions screening, bank verification, payouts, bank-change notices | $2 per school per month with a payout; 0.25% + 25¢ per payout; 0.25% of payout volume ([Stripe Connect pricing](https://stripe.com/connect/pricing)) |
| MOU signature | Dropbox Sign API | Legally binding e-signature with a signer audit trail and executed PDF | Essentials $75 a month or $900 a year, from 50 requests a month ([Dropbox Sign API pricing](https://sign.dropbox.com/products/dropbox-sign-api/pricing)) |
| General ledger, bank reconciliation, financial statements, auditor access | QuickBooks Online (existing) + its API | Official books, bank feeds, reconciliation, closing-date lock, Accountant user access | Existing plan; API writes are free and reads are free up to 500,000 calls a month ([Apideck on Intuit API pricing](https://www.apideck.com/blog/quickbooks-api-pricing-and-the-intuit-app-partner-program)) |
| W-9, TIN match and 1099s for tutors or other vendors (Later) | Avalara 1099 & W-9 | W-9 collection, IRS TIN matching, 1099 e-filing | TIN match $0.45; e-file $3.10 a form for the first 15 ([Avalara pricing](https://www.avalara.com/us/en/products/1099/pricing.html)) |
| Login, MFA, encrypted storage, backups | Supabase (existing) | Infrastructure security and encryption at rest | In the existing plan |

**Trade-off on school payouts.** Paying schools through Stripe Connect costs about 0.5% of what is paid out (about $75 on a $15,000 payment), against a few dollars for an ACH sent from TENNISSA's bank. What the fee buys is verified school bank accounts, automatic remittance references and no bank numbers in TENNISSA's hands. If the cost is too high, the fallback is to verify school bank accounts with Plaid and keep paying from the §25F bank as module 4 does today.

**Check with counsel.** With Stripe Connect, §25F money waits in TENNISSA's Stripe balance before it reaches schools. Confirm that a Stripe account used only for §25F gifts meets the separate-account rule; if not, use one Stripe account for §25F gifts and another for general gifts, or the bank fallback above.

## Database design and relationships

Twenty-six tables in Supabase Postgres (13 drawn below, plus helpers and `profiles` and `audit_log`); the award is the hinge that ties a student's application to a school, a fund and the payments made.

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
| schools | name, EIN, address, contact, status (registered, under review, approved, suspended), TDOE ID, Stripe Connect account ID, payouts enabled, bank last four, bank change pending | Has many awards and disbursements |
| disbursements | amount, term, method, reference, planned/paid date, status, approved by | Belongs to an award, a school and a fund |
| donors | donor number, name, address, email, spouse link, account owner (the login that manages this donor) | Belongs to a donor profile; has many donations |
| donations | amount, §25F designation, Stripe PaymentIntent ID, status, settled date, Stripe fee | Belongs to a donor and a fund; has acknowledgments |
| acknowledgments | type (gift or year-end), folder (donor + tax year), PDF path, issued date, emailed date, void flag, replaced-by link for corrected acknowledgments | Belongs to a donation (or a donor and year for year-end) |
| funds | name (§25F scholarship, operating), bank account label | Has many donations, disbursements and expenses |
| expenses | category, amount, date, vendor, reference, fund | Belongs to the operating fund |
| income\_limits | county, household size, 300% limit, year | Read by the eligibility check |
| audit\_log | actor, action, table, record, before, after, time | Written by database triggers; read-only |
| school\_agreements | MOU version, school year, Dropbox Sign request ID, signer, signed date, executed PDF path | Belongs to a school |
| tuition\_schedules | school year, grade, tuition, mandatory fees | Belongs to a school; caps awards |
| enrollment\_verifications | result (confirmed, declined), reason, tuition confirmed, school user, date | Belongs to an application and a school |
| school\_refunds | student withdrawal date, amount owed, due date, received date, status | Belongs to an award and a school |
| ledger\_accounts | code, name, type (asset, liability, income, expense, net assets), fund, QuickBooks account ID | Belongs to a fund; has many journal lines |
| journal\_entries | date, kind (gift settled, fee, return, payout, transfer, refund, expense, reversal), source record, reverses, QuickBooks ID, sync status | Has two or more journal lines; append-only |
| journal\_lines | account, debit, credit, fund | Belongs to a journal entry; debits equal credits per entry |
| reconciliations | account, month, statement balance, book balance, unmatched items, status, signed off by | Belongs to a bank or Stripe clearing account |
| period\_closes | month, closed by, closed date, reopened by, reason | Blocks back-dated journal entries |

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

Stripe tells the app when an ACH gift settles; the app issues the acknowledgment only then. Stripe Connect sends each school payment; QuickBooks receives every money event as a journal entry from day one.

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

### Stripe Connect school payment flow

1. The school finishes Stripe's hosted onboarding; Stripe verifies the school, its EIN and its bank account and sends `account.updated` with payouts enabled.
2. An administrator approves a batch of disbursements; amounts over the set limit need a second administrator.
3. The app creates one Stripe transfer per school with the disbursement IDs in its metadata, records it as **Sent** and posts the journal entry.
4. Stripe pays the school's bank; the app marks the payment **Paid** and emails remittance advice listing each student.
5. A returned payout or a bank-account change from Stripe puts the school on hold until an administrator confirms it.

### QuickBooks approach

| Option | What it does | When |
| --- | --- | --- |
| QuickBooks Online API sync (MVP) | App posts a balanced journal entry for every gift, fee, return, payout, school transfer, refund and expense, tagged by fund (class); nightly balance check | Launch |
| QuickBooks bank feeds and reconciliation (MVP) | QuickBooks pulls the §25F and operating bank accounts; the bookkeeper reconciles monthly and sets the closing date | Launch |
| CSV export | The same entries as a QuickBooks import file, as a backup if the API is unavailable | Launch |

In QuickBooks, track the §25F scholarship fund and operating funds separately (classes or separate bank accounts) so the books match the app's 90/10 numbers. QuickBooks stays the system of record for the general ledger; the app is the system of record for donors, awards and disbursements.

## Implementation phases and timeline

Four phases over about nine months, each usable on its own: donations first because January 1, 2027 is the only fixed deadline, then the scholarship cycle in the order TENNISSA will run it.

&#91;embedded content: MVP roadmap · 4 phases, 4 gates\]

Phase 1 has seven weeks, so it holds only what donors need on day one; the final two weeks of December are for testing with real ACH gifts of a few dollars.

| Phase | Weeks | Effort (developer-weeks) | Done when |
| --- | --- | --- | --- |
| 1 · Donations live | 7 | 7–8 | A test donor gives by ACH, sees Pending, then gets the PDF acknowledgment by email after settlement |
| 2 · Apply, review | 8 | 7–8 | A family submits a full application with documents; an assigned reviewer recommends an amount |
| 3 · Schools, award and pay | 11 | 9–11 | A school registers, signs the MOU in Dropbox Sign and connects its bank through Stripe; it verifies a student; the admin approves the award and a Stripe transfer appears under the school's Payments received |
| 4 · Books, report, harden | 10 | 8–10 | Journal entries since January 1 are posted to QuickBooks and balances agree; one month is reconciled and closed; the 90% tracker is right; an examiner login downloads the audit package; backup restore and access review completed |

**Team:** one experienced full-stack developer (Next.js and Supabase) full time, plus about 10% of a designer for screens and receipts and a TENNISSA staff member for testing each phase. Total about **31–37 developer-weeks**. A contractor at $75–$125 an hour would put the build at roughly $93,000–$185,000; a volunteer or in-house developer changes that, not the stack.

**After the MVP (phase 2 platform):** vendor payments for tutoring with W-9 and 1099s through Avalara, Spanish, IRS transcript checks, and the year-end acknowledgment and IRS export run for the first time in January–February 2028.

## Estimated monthly costs

Fixed platform costs run about **$120–$155 a month**; Stripe adds a few dollars per donation. Prices are list prices checked October 2026; confirm before signing up.

| Service | Plan | Monthly cost | Notes |
| --- | --- | --- | --- |
| Vercel (hosting) | Pro, 1 developer seat | $20 | Hobby is personal, non-commercial only ([Vercel pricing](https://dev.to/nayankyada/vercel-pricing-2026-what-you-actually-pay-for-a-real-nextjs-project-33md)) |
| Supabase (database, auth, storage, backups) | Pro | $25 + small usage | Includes daily backups; Team plan at $599/month adds SOC 2 reports and SSO, not needed for the MVP ([Supabase pricing](https://www.cloudzero.com/blog/supabase-pricing/)) |
| Resend (email) | Free, then Pro | $0–$20 | Free covers 3,000 emails/month at 100/day; Pro is $20 for 50,000/month, needed for January year-end acknowledgments ([Resend pricing](https://superdupr.com/blog/resend-pricing)) |
| Sentry (error alerts) | Developer (free) | $0 | Upgrade only if alerts outgrow it |
| PDF generation | Open-source library in the app | $0 | No separate service |
| QuickBooks Online | TENNISSA's existing plan | $0 added | Class tracking may need a higher QuickBooks tier |
| Domain and DNS | Existing or new | \~$2 | About $20–$25 a year |
| Dropbox Sign (MOU signatures) | API Essentials | $75 | $900 a year paid annually; from 50 signature requests a month; free test mode for development |
| **Total fixed** |  | **\~$120–$155** | Plus usage overages if volumes grow well beyond the MVP |

**Stripe per-donation fees (no monthly fee)**

| Fee | Amount |
| --- | --- |
| ACH Direct Debit, standard settlement | 0.8%, capped at $5 per gift |
| Instant bank verification (Financial Connections) | $1.50 per verification |
| Failed ACH payment | $4 |
| Disputed ACH payment | $15 |
| Connect: school paid in a month | $2 per school |
| Connect: each payout to a school | 0.25% + 25¢, plus 0.25% of payout volume |

A $1,700 gift costs $5 plus a one-time $1.50 bank verification for a new donor; a $15,000 payment to a school costs about $75 plus $2 for that month ([Stripe ACH fees](https://superdupr.com/blog/ach-fees)).

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
| §25F money held in a Stripe balance may not count as a separate account | Commingling finding in the annual audit | Counsel confirms before Phase 3; fallback is paying schools by ACH from the §25F bank with Plaid-verified school accounts |
| App journal and QuickBooks drift apart | Reports disagree with the books | Nightly balance check by account and class; bookkeeper edits in QuickBooks only through reversing entries; closing date locked after reconciliation |

**Open questions**

- [ ] Is TENNISSA approved, or applying, for the Tennessee SGO list, and is its 501(c)(3) status final or pending?
- [ ] Which bank holds the dedicated §25F scholarship account, and who can send ACH payments to schools from it?
- [ ] What award criteria and scoring should reviewers use beyond the income test?
- [ ] Will scholarships cover only tuition at schools, or also tutoring and other qualified expenses in the first year?
- [ ] Who builds it: an in-house developer, a contractor, or an agency? (Changes the timeline's labor cost, not the stack.)
- [ ] Which QuickBooks Online plan does TENNISSA use, and does it have class tracking?
- [ ] Should the donor portal also accept non-§25F general gifts, or §25F gifts only? (General gifts would need a separate bank account and receipt wording.)
- [ ] Can a gift from a joint bank account be designated as one spouse's gift, or split between spouses? The proposed rules treat spouses as separate taxpayers but don't address joint accounts; until counsel confirms, "split between us" stays off and each spouse gives from their own bank account or chooses with a short disclosure.
- [ ] Does a Stripe account used only for §25F gifts meet the separate-account rule, so schools can be paid by Stripe Connect transfer?
- [ ] Is about 0.5% of each school payment through Stripe Connect acceptable, or should schools be paid by ACH from the §25F bank?
- [ ] What must the partner school MOU say, and who at a school is authorized to sign it?

**Sources**

- [Federal Register: proposed §25F regulations (Oct 2, 2026)](https://www.federalregister.gov/documents/2026/10/02/2026-20277/federal-scholarship-tax-credit)
- [IRS: Federal Scholarship Tax Credit](https://www.irs.gov/government-entities/federal-state-local-governments/federal-scholarship-tax-credit-fstc)
- [TDOE: Scholarship Granting Organizations](https://www.tn.gov/education/about-the-tdoe/scholarship-granting-organizations.html)
- [Stripe: US bank payments](https://stripe.com/payments/us-bank-payments)
- [Superdupr: Stripe ACH fees 2026](https://superdupr.com/blog/ach-fees)
- [CloudZero: Supabase pricing](https://www.cloudzero.com/blog/supabase-pricing/)
- [DEV: Vercel pricing 2026](https://dev.to/nayankyada/vercel-pricing-2026-what-you-actually-pay-for-a-real-nextjs-project-33md)
- [Superdupr: Resend pricing 2026](https://superdupr.com/blog/resend-pricing)
- [Stripe: Connect pricing](https://stripe.com/connect/pricing)
- [Dropbox Sign: API pricing](https://sign.dropbox.com/products/dropbox-sign-api/pricing)
- [Apideck: QuickBooks API pricing and the Intuit App Partner Program](https://www.apideck.com/blog/quickbooks-api-pricing-and-the-intuit-app-partner-program)
- [Avalara: 1099 & W-9 pricing](https://www.avalara.com/us/en/products/1099/pricing.html)

Not tax or legal advice; confirm §25F requirements with TENNISSA's tax counsel.
