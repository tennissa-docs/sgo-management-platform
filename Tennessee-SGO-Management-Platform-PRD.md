# Tennessee SGO Management Platform — PRD

Oct 29, 2026

## Executive summary

Build a Tennessee-only, multi-tenant SaaS that lets a Scholarship Granting Organization (SGO) run the whole federal §25F program from one ledger: take credit-eligible gifts, award income-verified scholarships, pay schools and vendors directly, and prove the 90% spending test on its Tennessee §25F account. The first credit-eligible gifts arrive on **January 1, 2027**, so the donation-and-receipt module must ship in about nine weeks; everything else can follow on the program's own calendar.

**Product vision.** The system of record an SGO's auditor, state regulator and IRS filing all read from. Every dollar is tagged at entry to the Tennessee §25F segregated account and traced to a qualified expense, so compliance is a query, not a year-end reconstruction.

**Who buys it.** Launch customer: newly formed, single-state SGOs in Tennessee, sold as software only (no managed services), priced at 2% of each processed donation, capped at $25 per gift, with ACH as the default. Scope is Tennessee only: SGOs located in and listed by Tennessee, funding Tennessee-resident students; other states and multistate SGOs are out of scope. Treasury projects 600 to 700 SGOs by 2030 and about $26 billion a year in contributions.

**Five modules (your scope):**

1. Applications and eligibility: household income at or below 300% of area median gross income (AMGI), secure document collection, priority rules, disqualified-person screening.
2. Contributions and donor credits: cash-only qualified gifts, unique donor numbers, January 31 acknowledgments, February 28 IRS portal report.
3. Payments routing: tuition paid directly to schools, verified vendors, receipt-backed reimbursements, approval controls, overpayment recovery.
4. School and vendor registration: onboarding, verification, enrollment confirmation, payout accounts.
5. §25F compliance: per-account 90% tracking, annual certification and audit package, state reports.

**Build stance.** SaaS-first: buy payments, banking, identity, documents, auth, email and analytics; build only the §25F rules engine, the ledger and the workflows. That keeps the team small and inherits SOC 2 controls from vendors.

**Regulatory caveat.** The rules are proposed regulations (October 2, 2026) that SGOs may rely on; final rules, Form 8525 and separate §530 qualified-expense guidance are still pending. Every rule must be configuration, not code.

## Required capabilities checklist

Every item below is in scope for v1 and traces to a requirement ID. SGO Pro already advertises most of them, so treat this list as table stakes that buyers expect, not as differentiators. Positioning line: **Built for the Education Freedom Tax Credit (§25F)**, engineered for IRC §25F and EFTC scholarship granting organizations.

| Group | Capability | Where it is specified |
| --- | --- | --- |
| Federal §25F compliance | Anti-earmarking controls that block student-level designations at checkout | DON-8 |
| Federal §25F compliance | §25F donor receipts with the $1,700 cap and double-benefit language | DON-6, DON-10 |
| Federal §25F compliance | 90/10 scholarship-ratio monitoring with alerts and reports | CMP-3, CMP-4, CMP-7 |
| Federal §25F compliance | AMI income-ceiling checks on family applications | APP-4 |
| Federal §25F compliance | Restricted-funds ledger for contribution accounting | CMP-1 |
| Federal §25F compliance | Conflict-of-interest guardrails on fund releases | PAY-5, APP-9 |
| One dashboard for your SGO | Contributions, applications, disbursements and reports without switching tools | EXP-1, EXP-4 |
| Your own branded portal | Families, contributors and partner schools each get a clear path | EXP-2, EXP-3 |
| Platform security | Encrypted transport (HTTPS) for all platform traffic | Non-functional: Encryption |
| Platform security | Role-based access for SGO staff and portal users | Non-functional: Access |
| Platform security | Payments processed by Stripe on the SGO's own Stripe account | Tech stack: Donations in (Stripe Connect Standard) |
| Platform security | Card numbers never touch platform servers | Non-functional: Certifications (SAQ-A) |
| Platform security | Encrypted storage for tax identifiers and payment keys | Non-functional: Encryption |
| Award management | Cycles, budgets, letters, balances, amendments, waitlists | AWD-1 to AWD-8 |

## Regulatory background: what §25F requires

The platform's requirements come almost entirely from IRC §25F, the [proposed regulations (REG-117199-25, 91 FR 62818, October 2, 2026)](https://www.federalregister.gov/documents/2026/10/02/2026-20277/federal-scholarship-tax-credit) and the [temporary regulations (T.D. 10057)](https://www.federalregister.gov/documents/2026/10/02/2026-20264/federal-scholarship-tax-credit), summarized in plain English by [eftccredit.com](https://eftccredit.com/documents/treasury-proposed-regulations). Comments close December 1, 2026; a public hearing is set for December 15, 2026.

| Rule | What it says | Platform implication |
| --- | --- | --- |
| Credit amount | Nonrefundable credit up to $1,700 per individual per year; each spouse on a joint return has a separate cap ($3,400 total); 5-year carryforward | Track designated totals per donor per year; show cap usage, never block gifts above it |
| State credit offset | Federal credit = qualified contributions minus any state credit on them, capped at $1,700 | Record state-credit program per gift where an SGO also runs a state program |
| Qualified contribution | Cash only (card, ACH, check, payroll), designated at time of gift, irrevocable; no digital assets; partnership/S-corp shares don't count | Designation captured at checkout; reject stock, crypto, DAF and entity gifts into §25F accounts |
| Segregated account | Qualified contributions kept in a §25F account, not commingled (one Tennessee account; multistate rules not applicable) | One Tennessee §25F sub-ledger per SGO, backed by its own bank account |
| 90% spending test | Spend ≥ 90% of income on scholarships by the end of the following year; income = gross receipts, or account receipts + earnings under the 85% safe harbor; FIFO by year; refunds are new income; wallet transfers count when made | Year-cohort ledger with FIFO allocation and a forecast of each cohort's deadline |
| Distribution | Scholarships to ≥ 10 students not all at one school | Live counter on the compliance dashboard |
| Student eligibility | Household income ≤ 300% of AMGI (Section 8 method, family-size adjusted); resides in the state | Geocode address to the HUD area; versioned income-limit tables |
| Income verification | Direct (pay stubs, returns, transcripts, W-2), categorical (SNAP, TANF, WIC, Section 8, SSI letter ≤ 12 months old), census-tract and foster-child safe harbors | Four verification paths with document checklists and reviewer sign-off |
| Priority | Prior-year recipients first, then their siblings | Award queue that sorts by priority tier before any rubric |
| No earmarking | Donors may not direct gifts to a specific student | No student selector on giving pages; school-preference only if counsel approves |
| Disqualified persons | No awards to substantial contributors (> $5,000 and > 2%), officers, directors, selection-committee members, or their families | Screen applicants against donor and governance registers |
| Paying out | Tuition and school charges paid to the school; vendors verified and unrelated; family reimbursements only with verified receipts; fraud and duplicate controls; qualified digital wallets allowed | Direct-pay rails, vendor KYB, receipt review, duplicate-expense checks |
| Reporting | Register in the IRS SGO portal; acknowledgment with EIN, total and donor number by **Jan 31**; IRS donor report by **Feb 28**; annual certification attached to Form 990 and sent to each listing state | Generators for each artifact, with delivery tracking |
| Audit | Annual financial and programmatic audit; independent external auditor above $500,000 receipts | Auditor workspace with read-only access and evidence export |
| States | State files Form 15714 by Jan 1, 2027 and its SGO list by Feb 15, 2027; may not be more restrictive than federal rules | Tennessee configuration: TDOE listing status and state reporting |

**Still pending:** final regulations, Form 8525 and its instructions, the donor-number format (issued through the IRS SGO portal), the IRS-published AMGI tables, and §530 guidance on qualified expenses and schools. The [IRS FSTC page](https://www.irs.gov/government-entities/federal-state-local-governments/federal-scholarship-tax-credit-fstc) is the canonical place these will appear.

## Launch state: Tennessee

Tennessee is the first configured state: it has opted in, the Department of Education (TDOE) certifies the SGO list, and TDOE opened its [preliminary SGO interest form on October 7, 2026](https://www.tn.gov/education/news/2026/10/7/tennessee-advances-participation-in-the-federal-education-tax-credit-program-.html). Questions go to SGO.Questions@tn.gov.

The state's [Federal Tax Credit Scholarship Act (HB2187/SB2206)](https://billtexts.s3.amazonaws.com/tn/SB-2206-TNB00059738Amended-with-HA0888-03-30-2026-2026-03-30.pdf) directs TDOE to certify, submit and publish the list; lets TDOE set rules for the documentation SGOs must provide; and allows scholarships for any qualified K-12 expense, including home school expenses, to the extent federal law allows. The House amendment in that text also requires the list to include, and show first, at least one SGO devoted solely to tutoring, afterschool and special-needs services for public school students (confirm against the enacted text).

| Setting | Tennessee configuration |
| --- | --- |
| State accounts | One §25F segregated account (TN) per SGO; no multistate features |
| Listing authority | TDOE; federal deadline for the 2027 list is February 15, 2027 |
| SGO onboarding checklist | 501(c)(3) status (pending allowed), IRS SGO portal registration, TDOE interest form, Tennessee charitable-organization compliance with the Secretary of State |
| Student residency | Resides in Tennessee; military and Indian-lands exceptions per federal rule |
| Income limits | 300% AMGI by Tennessee county and HUD area (for example Nashville-Davidson–Murfreesboro–Franklin), from IRS tables when published |
| Expense types | Tuition and fees (paid to school), tutoring, supplies, special-needs services, home school expenses once §530 guidance confirms them |
| Award programs | Templates for private-school tuition awards and public-school tutoring/afterschool/special-needs awards |
| School and vendor registry | Seed public schools from the [TN School Directory](https://tnschooldirectory.tnedu.gov/); non-public schools and tutoring vendors self-register |
| Other aid | Ask applicants about other scholarships for the same expense (for example state programs) and block duplicate payment |
| State reporting | Annual certification and audit results to TDOE; add any TDOE report format once rules are issued |
| Language | English and Spanish |

## Market research: competitive landscape

The market splits into four groups, and none yet owns the §25F SGO end to end: new §25F-native startups have the right model but no track record; payment-wallet incumbents have scale but sell to states; school-finance incumbents have SGO customers but stop at the award. The opening is a §25F-native platform with incumbent-grade payments and security, priced inside the SGO's 10% operating allowance.

| Product | Group | What it does for SGOs | Pricing model | Gap vs. this PRD |
| --- | --- | --- | --- | --- |
| [scholarships.software / SGO Software](https://sgosoftware.com/) | §25F-native | Applications, donor §25F credits, school disbursement; 300% AMGI math; segregated balances on regulated rails it does not custody | No setup or license fees; earns only on processed donations | Young product; your feature list closely mirrors its homepage wording, so differentiation must come from execution, not the feature names |
| [ClearPath by SGO Guide](https://sgoguide.com/sgo-software) | §25F-native | Donors, Scholarship, Shield (per-account 90/10, blocked over-cap releases), Insights; formation and managed-SGO services; school fundraising suite | Consultation-based; paid from the 10% allowance | Bundles services; not a neutral platform for SGOs that want their own staff |
| SGO HQ (via [eftccredit.com](https://eftccredit.com/documents/treasury-proposed-regulations)) | §25F-native | Donors, scholarships and compliance; free SGO builder and content funnel | Not published | Early-stage |
| [ClassWallet](https://classwallet.com/fstc) | Payment wallet | Turnkey FSTC offer: fundraising, compliance, wallet distribution; multi-program wallets (FSTC + state ESA, one login); $7.5B+ distributed, 38 states, SOC 1 and SOC 2 Type II | Vendors pay a 2.5% fee in state ESA programs ([NC ESA+](https://k12.ncseaa.edu/media/roqmwfwh/provider-introduction-to-esaplus-july-2025_final.pdf)) | Wallet-first; weaker on application intake and donor CRM. Best treated as a **partner** (a qualified digital wallet) |
| [Odyssey](https://yespress.io/odyssey) | Payment wallet | State ESA administration: applications, marketplace, payments, audit; Texas, Iowa, Idaho, Georgia, Louisiana, Utah, Wyoming | Per student plus transaction fees, sold to states | Government buyer, not SGOs |
| Student First Technologies, Merit | Payment wallet | ESA and microgrant fund administration | Not published | Same as Odyssey |
| [FACTS Compass](https://factsmgt.com/blog/how-sgos-can-streamline-scholarship-management-at-scale/) (Nelnet) | School finance | White-label applications, document collection, IRS income validation, award prioritization, waitlists, reports; 120+ SGOs already | Contract | Strong at intake and award; no §25F donor ledger or per-account 90% engine evident |
| [Step Up For Students EMA](https://www.stepupforstudents.org/wp-content/uploads/SFO-Renewal-Press-Release.pdf) | In-house SGO | Florida's largest SFO; EMA handles applications, processing, state reporting, providers and payments | Not sold | Proof that a mature SGO needs one platform; also a model of the school/provider portal |
| [Submittable, Good Grants, SmarterSelect, AwardSpring](https://www.capterra.com/scholarship-management-software/) | Generic awards | Forms, reviewer scoring, applicant portals | Annual license | No donor credit, no AMGI, no payments, no 90% test |

Also in market: [SGO Pro](https://sgopro.app/compliance) advertises per-gift §25F receipts with the $1,700 cap, §170 double-benefit notice and state-credit reminder; a restricted-funds ledger with audit trail; anti-earmarking blocks at checkout that log the attempt; a configurable AMI ceiling check (default 300%); 90/10 monitoring with alerts, board reports and an audit packet export; and conflict-of-interest blocks on fund releases. That set is now the baseline buyers will compare against.

**What this means for the product:**

- **Differentiate on proof, not features.** Every competitor lists the same five capabilities. Win on a tamper-evident ledger, a one-click audit package, SOC 2 Type II from day one, and fast school payments.
- **Partner for wallets, build for direct pay.** Tuition goes school-direct on your own rails; non-tuition spending can route to ClassWallet as a qualified digital wallet instead of rebuilding a marketplace.
- **Price inside the 10%.** Competitors anchor on fees that come from the operating allowance; the chosen model is 2% per processed donation, capped at $25, with ACH as the default, so a $1,700 ACH gift costs the SGO about 1.8% in total.
- **Win new Tennessee SGOs.** Tennessee has few established SGOs, so the wedge is getting newly formed Tennessee SGOs listed by TDOE and taking gifts on January 1, 2027; large national and multistate SGOs are not the target.

## Personas and user roles

Nine roles share one tenant; every permission is scoped to an SGO.

| Role | Who | Core jobs | Access |
| --- | --- | --- | --- |
| SGO administrator | Executive director, ops lead | Configure programs, award cycles, fees, staff; release payments | Full tenant, except auditor-only views |
| Finance officer | Controller, bookkeeper | Reconcile bank accounts, approve disbursements, close the year, file reports | Ledger, payments, reports; dual approval above thresholds |
| Eligibility reviewer | Staff or contractor | Review documents, verify income, resolve flags | Applications in assigned queue; masked PII outside it |
| Selection committee member | Volunteer or board | Score and decide awards | Award docket only; flagged as disqualified-person themselves |
| Family (applicant) | Parent or guardian | Apply, upload documents, accept award, submit receipts | Own household only; English and Spanish |
| Donor | Individual taxpayer | Give, designate as §25F, see cap usage, download acknowledgment | Own giving history |
| School / vendor | Bursar, tutor, therapist | Register, confirm enrollment, invoice, receive payouts, return overpayments | Own students' award balances and payouts |
| Auditor / state reviewer | Independent CPA, state agency | Sample transactions, review procedures, receive annual package | Read-only, time-boxed, fully logged |
| Platform operator | Your company | Onboard SGOs, support, monitor | Break-glass access only, logged and alerted |

## Goals, non-goals and success metrics

**Goals**

1. An SGO can accept its first credit-eligible gift on January 1, 2027, with a correct donor number and acknowledgment.
2. Every award is reproducible from stored evidence: income documents, AMGI table version, priority tier, reviewer and committee decision.
3. Every scholarship dollar traces from a donor's designated gift to a paid, qualified expense.
4. The 90% test, 10-student test and disqualified-person rule are visible daily, not discovered at year end.
5. Annual artifacts (acknowledgments, IRS donor report, certification, state reports, audit package) generate in under one hour of staff time.

**Non-goals (v1)**

- SGO formation, legal or tax advice (partner referrals only).
- Filing on the donor's behalf or preparing Form 8525.
- An open e-commerce marketplace for curriculum and supplies (use a qualified-wallet partner).
- Custodying funds as a money transmitter; balances sit in the SGO's own FDIC-insured accounts.
- State ESA program administration for governments, and managed SGO operations: the product is software only. Also out of scope: SGOs in other states and multistate SGOs.

**Success metrics**

| Metric | Target (first 12 months) |
| --- | --- |
| Donor acknowledgment errors | 0 by January 31, 2028 |
| IRS donor report submitted on time | 100% of tenants by February 28, 2028 |
| Application-to-decision median | ≤ 10 business days |
| Share of applications auto-verified (categorical or payroll data) | ≥ 50% |
| School payout time after enrollment confirmation | ≤ 3 business days |
| Payments to unverified payees | 0 |
| Audit findings attributable to platform records | 0 |
| Live SGO tenants | 25 Tennessee SGOs by end of 2027 |
| Gross gift volume processed | $25M in 2027 (about $370,000 to $500,000 platform revenue) |

The last two are business targets to set with your go-to-market plan; the rest are product quality bars.

## Business model and pricing

Recommended model: **2% of each processed donation, capped at $25 per gift, with ACH bank transfer as the default at checkout**. The product is software only, with no setup, license or monthly fees. This replaces the earlier flat 3%, which on a card gift, added to Stripe's fees, cost an SGO about 6% and used most of its 10% operating allowance.

| Item | Decision |
| --- | --- |
| Fee | 2% of each processed donation, capped at $25 per gift |
| Collection | Stripe Connect application fee at the moment of the gift; works with each SGO's own Stripe account |
| Checkout default | ACH bank transfer preselected; cards and digital wallets still available |
| Processing fees | Charged by Stripe to the SGO's account: roughly 0.8% capped at $5 for ACH, roughly 2.9% + 30¢ for cards (nonprofit discount available); confirm current rates with Stripe |
| Donor-covered fees | Optional "cover the fees" box at checkout, pending counsel review |
| Accounting | Platform and processing fees post to the operating side of the restricted-funds ledger and count against the 10%, never against scholarship funds |
| What's included | All modules, unlimited staff users, Tennessee configuration, IRS transcript verification |

**Cost of a $1,700 gift under the recommended model**

| Payment method | Platform fee | Stripe fee (approx.) | SGO's total cost |
| --- | --- | --- | --- |
| ACH | $25 | $5 | $30 (1.8%) |
| Card | $25 | $50 | $75 (4.4%) |

**Alternatives considered** (per $1,700 gift)

| Model | Platform fee | SGO's total cost (card / ACH) | Trade-off |
| --- | --- | --- | --- |
| 2% capped at $25, ACH default (recommended) | $25 | ~$75 (4.4%) / ~$30 (1.8%) | Low cost to SGOs, easy to explain; less revenue per large gift |
| Flat 3% | $51 | ~$101 (5.9%) / ~$56 (3.3%) | Simple, but expensive on cards |
| 3% capped at $25 | $25 | ~$75 (4.4%) / ~$30 (1.8%) | Same cost at the cap; higher take on small gifts |
| Flat $15–25 per gift | $15–25 | ~$65–75 / ~$20–30 | Predictable; fits because gifts cluster near the $1,700 cap |
| 1% + subscription ($99–299 a month by tier) | $17 + subscription | ~$67 / ~$22, plus subscription | Steady revenue; harder sell to new SGOs with no donors yet |
| All-in rate (platform pays Stripe; e.g. 3.5% card / 1% ACH) | ~$10 net card, ~$12 net ACH | ~$60 (3.5%) / ~$17 (1%) | One number for buyers; platform carries processing risk; needs Stripe Connect Express or Custom |
| Annual SaaS tiers by gift volume | $0 per gift | Stripe only (~$50 / ~$5) | Cheapest for large SGOs; risky revenue in year one |

At the $25M gift-volume target, the model yields roughly $370,000 to $500,000 a year, depending on how many gifts are large enough to hit the cap (it binds from $1,250 upward). It undercuts competitors' flat 3% pricing. The all-in rate is the upgrade path: moving SGOs to Stripe Connect Express or Custom would let the platform negotiate Stripe volume pricing and offer one rate.

## Functional requirements

P0 = needed for that module's first live use; P1 = within the first program year; P2 = later. The contributions module is P0 for January 1, 2027; the rest are P0 for their own go-live dates in the roadmap.

### 1. Student applications and eligibility verification

| ID | Requirement | Priority |
| --- | --- | --- |
| APP-1 | Configurable application forms per award cycle (household, students, school, expense type), mobile-first, English and Spanish, WCAG 2.2 AA | P0 |
| APP-2 | Household model: members, ages, relationships, residence address, custody (longest-residence rule for two-household students) | P0 |
| APP-3 | Geocode residence to state, county and HUD income-limit area; reject or flag out-of-state residents; military and Indian-lands dual-state exceptions | P0 |
| APP-4 | Income engine (AMI/AMGI ceiling check at application time): compute household cash income vs. 300% AMGI for family size from a versioned table; allow a stricter SGO-set limit | P0 |
| APP-5 | Four verification paths: direct (pay stubs, W-2, 1099, returns, IRS transcripts), categorical (SNAP/TANF/WIC/Section 8/SSI letter dated ≤ 12 months), census-tract safe harbor, foster-child safe harbor | P0 |
| APP-6 | Secure document upload with malware scan, OCR and field extraction, expiry checks, and reviewer side-by-side view | P0 |
| APP-7 | IRS transcript verification at launch: family e-signs Form 4506-C consent, request routed through an IVES participant, transcript income parsed and matched to the application; payroll/bank income connection as a second instant path | P0 |
| APP-8 | Identity and fraud checks on the applicant (name, DOB, address match; duplicate household and duplicate student detection across cycles) | P0 |
| APP-9 | Disqualified-person screen against substantial-contributor, officer, director, committee and family registers, re-run at award date | P0 |
| APP-10 | Priority engine: tier 1 renewals, tier 2 siblings of recipients, then SGO rubric or lottery; need-based priority allowed for tutoring/special-needs awards | P0 |
| APP-11 | Reviewer queues, SLA timers, request-more-info loop, notes, and four-eyes approval on overrides | P0 |
| APP-12 | Committee docket with blind-to-donor view, conflict attestations, recorded votes, and decision letters | P0 |
| APP-13 | Award acceptance, waitlists, award amendments and cancellations, multi-year awards | P1 |
| APP-14 | Renewal applications prefilled from prior year | P1 |

### 2. Contributions and §25F donor credits

| ID | Requirement | Priority |
| --- | --- | --- |
| DON-1 | Hosted, SGO-branded giving pages and embeddable widget; card, ACH and digital wallets; recurring gifts; check and payroll-deduction entry by staff | P0 |
| DON-2 | Explicit, irrevocable §25F designation at checkout with disclosure text; non-designated gifts route to the general (§170) account | P0 |
| DON-3 | Every designated gift posts to the SGO's Tennessee §25F segregated ledger and bank account | P0 |
| DON-4 | Reject non-cash and entity gifts (stock, crypto, DAF, partnership, S-corp) from the §25F path | P0 |
| DON-5 | Unique donor number generated in the IRS-prescribed uniform format, stable per donor per SGO; spouses as separate donors | P0 |
| DON-6 | Donor portal: running designated total, $1,700 cap indicator (per spouse), state-credit note, receipts; an instant §25F receipt on every gift showing the qualified amount, year-to-date total against the $1,700 cap, the §170 double-benefit notice and the state-credit offset reminder | P0 |
| DON-7 | Quid-pro-quo tracking: record good-faith value of goods or services and reduce the qualified amount | P0 |
| DON-8 | Anti-earmarking at checkout: block any student-level designation (no student fields; names or IDs in gift notes detected and the gift held for review); log every blocked attempt; school-level preference only behind a counsel-approved feature flag | P0 |
| DON-9 | Substantial-contributor tracking (> $5,000 and > 2% of SGO or account contributions; spouse aggregation; status for that year and the next) feeding APP-9 | P0 |
| DON-10 | Annual acknowledgment by January 31: EIN, total designated contributions, donor number, goods/services statement, §170 double-benefit language (a credited amount cannot also be deducted) and the $1,700 cap; e-delivery with consent, print-and-mail fallback | P0 |
| DON-11 | IRS SGO portal report by February 28: donor number, name, address, annual total, in the portal's file format | P0 |
| DON-12 | Refunds, chargebacks and corrections with a full reversal trail and corrected acknowledgments | P0 |
| DON-13 | Donor CRM basics: campaigns, school-ambassador referral links, employer payroll programs | P1 |
| DON-14 | State-credit offset field and reporting, used only if Tennessee adds a state tax-credit scholarship | P1 |

### 3. Payments routing to schools and vendors

| ID | Requirement | Priority |
| --- | --- | --- |
| PAY-1 | Payment request types: school invoice (tuition, fees, room and board), vendor invoice, family reimbursement with receipt, transfer to a qualified digital wallet | P0 |
| PAY-2 | Tuition and school charges payable only to a verified school, never to a family | P0 |
| PAY-3 | Gate the first payment on school enrollment confirmation for that student and term | P0 |
| PAY-4 | Expense-category rules engine mapped to §530(b)(3)(A) categories; versioned so the coming §530 guidance can be loaded as data | P0 |
| PAY-5 | Approval workflow: auto-approve within rules and limits, single approval, dual approval above an SGO-set threshold; segregation of duties enforced; conflict-of-interest guardrails flag and block any release where the requester or approver has a recorded relationship to the school, vendor or student household, or is a disqualified person, and require a COI attestation on each approval | P0 |
| PAY-6 | Pre-payment checks: award balance, funds available in the Tennessee §25F account and year cohort, duplicate-expense detection, payee not related to recipient, sanctions screen | P0 |
| PAY-7 | Payouts by ACH (same-day where available) and check; scheduled batch runs; remittance detail per student for schools | P0 |
| PAY-8 | Overpayment and error recovery: demand letters, school returns, offsets against future payouts; returned funds booked as new income | P0 |
| PAY-9 | Qualified-digital-wallet integration (for example ClassWallet) for non-tuition spending; record transfer date as spend date | P1 |
| PAY-10 | Family reimbursement review: receipt OCR, category match, duplicate-receipt hash, cap per category | P1 |

### 4. School and vendor registration

| ID | Requirement | Priority |
| --- | --- | --- |
| SCH-1 | Self-service registration: legal name, EIN, address, grades served, accreditation or state approval, contacts, W-9 | P0 |
| SCH-2 | Business verification (KYB): EIN match, Secretary of State status, sanctions, beneficial owners for vendors | P0 |
| SCH-3 | Bank account verification by instant account link or micro-deposits before any payout | P0 |
| SCH-4 | School eligibility under Tennessee rules (K-12 per §530(b)(3)(B) and Tennessee law); document expiry tracking and re-verification | P0 |
| SCH-5 | School portal: student roster of awardees, enrollment confirmation, invoices, payout history, 1099 data where applicable | P0 |
| SCH-6 | Multi-campus organizations and dioceses with one parent account and many schools | P1 |
| SCH-7 | Vendor types for tutors, therapists and special-needs providers, with credential uploads | P1 |
| SCH-8 | Bulk import of schools from state lists and existing state-program rosters | P1 |

### 5. §25F compliance

| ID | Requirement | Priority |
| --- | --- | --- |
| CMP-1 | Restricted-funds ledger: double-entry, holding qualified contributions as restricted funds apart from operating cash, with separate books per SGO for its Tennessee §25F segregated account; tracks every allocation and release; daily bank reconciliation | P0 |
| CMP-2 | Income definition switch: organization-wide gross receipts, or segregated-account receipts plus earnings under the 85% safe harbor | P0 |
| CMP-3 | 90% tracker by year cohort: income received, scholarship paid, FIFO allocation, remaining amount, deadline (last day of following taxable year), and spend-rate forecast; configurable targets with dashboard and email alerts (for example at 92% and 90%); monthly and board-ready 90/10 reports | P0 |
| CMP-4 | Hard block on operating releases that would push the §25F account below the 90% path; release requires finance approval and logs the projection | P0 |
| CMP-5 | 10-student / more-than-one-school counter and alerts | P0 |
| CMP-6 | Activity measurement for the 85% safe harbor (staff time or expense allocation inputs), since Treasury has not fixed the metric | P1 |
| CMP-7 | Annual certification report: applications, awards, award sizes, schools, expense categories, income, percent spent; export for Form 990 attachment and TDOE | P0 |
| CMP-8 | Audit package: procedures manual, sample selection, per-award evidence bundle, ledger exports, access logs; auditor workspace | P0 |
| CMP-9 | Immutable, hash-chained audit log of every create, change, approval, payout and export | P0 |
| CMP-10 | Rules registry: every statutory parameter (cap, 300%, 90%, deadlines, thresholds) stored as dated, versioned configuration with source citation | P0 |
| CMP-11 | TDOE report templates and Tennessee listing-status tracking | P1 |
| CMP-12 | Records retention schedule (at least 7 years) with legal hold | P0 |

### 6. Award management

| ID | Requirement | Priority |
| --- | --- | --- |
| AWD-1 | Award cycles with budgets per school year; available-to-award computed from restricted funds | P0 |
| AWD-2 | Award record: student, amount, terms, expense types, priority tier, committee vote, rule version | P0 |
| AWD-3 | Award letters with e-acceptance by families; school notified on acceptance | P0 |
| AWD-4 | Live balances per student (awarded, committed, paid, remaining) visible to SGO, family and school | P0 |
| AWD-5 | Amendments, cancellations and school transfers with reason codes and re-approval | P0 |
| AWD-6 | Waitlist auto-promotion in statutory priority order when funds free up | P1 |
| AWD-7 | Renewal pipeline flagging prior-year recipients and siblings for the next cycle | P1 |
| AWD-8 | Award reports by school, grade, income band, expense type and county | P1 |

### 7. SGO dashboard and branded portals

| ID | Requirement | Priority |
| --- | --- | --- |
| EXP-1 | One SGO dashboard: contributions, applications, awards, disbursements, 90/10 status and report deadlines on one home screen with drill-down | P0 |
| EXP-2 | Branded portals per SGO: logo, colors, custom domain (for example give.yoursgo.org), branded email sender | P0 |
| EXP-3 | Guided paths: families (apply, verify, accept, track), contributors (give, designate, receipt), partner schools (register, confirm enrollment, get paid) | P0 |
| EXP-4 | Staff task inbox: items awaiting review, approval or follow-up | P0 |
| EXP-5 | Compliance calendar: Jan 31 acknowledgments, Feb 28 IRS report, certification and state report dates | P1 |

## Non-functional requirements

The platform holds income documents, children's records and money movement, so it needs bank-grade controls from launch, not after the first customer asks.

| Area | Requirement |
| --- | --- |
| Certifications | SOC 2 Type I before first live gift; Type II within 12 months; PCI DSS SAQ-A by keeping card data in the payment provider's hosted fields |
| Privacy | GLBA Safeguards Rule posture for financial data; FERPA-aligned handling of school records; COPPA-safe (no accounts for children); state student-privacy laws; Data Processing Agreement per tenant |
| SSNs | Do not collect donor SSNs (the donor number replaces them); collect applicant SSN/ITIN for IRS transcript verification (a launch feature) only with signed Form 4506-C consent, tokenized in a vault and never shown in full to staff |
| Encryption | HTTPS (TLS 1.2+, HSTS) for all platform traffic; AES-256 at rest; field-level encryption for PII, tax identifiers (EIN, SSN/ITIN) and bank details; Stripe and other payment keys kept in a secrets manager, never in plaintext; customer-managed keys for enterprise SGOs (P2) |
| Tenant isolation | Row-level security on every table keyed by tenant; per-tenant encryption context; isolation tests in CI |
| Access | SSO (SAML/OIDC) for staff, MFA required for staff, schools and auditors; passkeys for families and donors; least-privilege RBAC; session timeout 15 minutes for staff |
| Money controls | Dual approval for payouts above threshold and for any bank-detail change; payee bank changes trigger a cool-off and callback; no single person can create a payee and pay it |
| Auditability | Append-only, hash-chained event log; WORM storage for documents and logs; 7-year retention |
| Availability | 99.9% monthly; 99.95% for giving pages from December 15 to January 31 and December 31 peak; RPO ≤ 5 minutes, RTO ≤ 4 hours |
| Performance | Giving page p95 < 1.5 s; reviewer queue loads < 1 s for 10,000 applications; acknowledgment batch for 100,000 donors < 1 hour |
| Scale (year 1) | 100 tenants, 500,000 donors, 200,000 applications, 1 million documents |
| Accessibility | WCAG 2.2 AA across family, donor and school portals; plain-language copy at grade 6–8 reading level |
| Localization | English and Spanish at launch; string catalogs ready for more |
| Observability | Centralized logs, traces and metrics; alerts on payout failures, reconciliation breaks and 90% path deviations |
| Data residency | All data stored in U.S. regions |

## System architecture and data model

A modular monolith on managed Postgres, with every module writing to one double-entry ledger and one append-only audit log; vendors are reached only through adapters so each can be swapped.

**Architecture: every dollar posts to one ledger before it moves**

```
 PORTALS                    CORE PLATFORM (built)                    SAAS VENDORS (bought)
 ┌──────────────────┐      ┌───────────────────────────────────┐    ┌──────────────────────┐
 │ Families         │ ───▶ │ Applications   │ Contributions    │ ◀─▶│ Stripe               │
 │ apply, upload    │      │ AMGI, docs,    │ donor no.,       │    │ gifts in, refunds    │
 ├──────────────────┤      │ priority       │ receipts         │    ├──────────────────────┤
 │ Donors           │ ───▶ ├────────────────┼──────────────────┤ ◀─▶│ Modern Treasury      │
 │ give, receipts   │      │ Payments       │ School registry  │    │ TN §25F account, ACH │
 ├──────────────────┤      │ approvals,     │ KYB, enrollment  │    ├──────────────────────┤
 │ Schools, vendors │ ───▶ │ payouts        │                  │ ◀─▶│ Verification         │
 │ register, invoice│      ├────────────────┴──────────────────┤    │ Truv, Plaid, Ocrolus │
 ├──────────────────┤      │ Compliance: 90% cohorts,          │    ├──────────────────────┤
 │ SGO staff        │ ───▶ │ certification, audit package      │ ◀─▶│ Identity, KYB        │
 │ review, approve  │      ├───────────────────────────────────┤    │ Middesk, Persona     │
 ├──────────────────┤      │ Rules registry · versioned §25F   │    ├──────────────────────┤
 │ Auditors, states │ ───▶ │ RESTRICTED-FUNDS LEDGER · TN §25F │ ◀─▶│ IRS, states          │
 │ read-only        │      │ Audit log · hash-chained, WORM    │    │ SGO portal, reports  │
 └──────────────────┘      └───────────────────────────────────┘    └──────────────────────┘
```

Portals call the five modules; modules call vendors through adapters; nothing moves money without a ledger entry and an audit event first.

**Key design decisions**

- **Modular monolith, not microservices.** One deployable with strict module boundaries is faster to ship by January and easier to audit. Split out payments later if volume demands.
- **Ledger as source of truth.** Bank and Stripe events reconcile into the ledger daily; the 90% test, certification and state reports are queries over it.
- **Rules as data.** Every statutory number and date lives in a versioned rules registry with effective dates and a citation, so final regulations become a config release, not a rewrite.
- **Evidence by reference.** Each award and payment stores pointers to its documents, rule version and approvers, so an auditor can replay the decision.
- **Tenant per SGO, one Tennessee §25F account.** Row-level security on tenant ID on every table; Tennessee rules live in configuration rather than code.

**Core entities**

| Entity | Key fields | Notes |
| --- | --- | --- |
| Tenant (SGO) | EIN, taxable year, safe-harbor election, IRS portal status | One per 501(c)(3) |
| StateAccount | state, bank account ref, listing status by year | One Tennessee §25F account per SGO |
| Donor | name, address, donor number, spouse link, consent flags | Donor number unique per SGO |
| Contribution | amount, method, designated (Y/N), §25F account, goods/services value, state-credit program | Immutable once designated |
| Household | members, address, AMGI area, family size, verified income, path | Re-verified each cycle |
| Student | household, grade, school, renewal and sibling flags | |
| Application | cycle, status, reviewer, documents, rule version | |
| Award | student, amount, term(s), expense types, priority tier, committee decision | Multi-year supported |
| School / Vendor | EIN, KYB result, bank ref, eligibility documents, relationship flags | |
| PaymentRequest | award, payee, category, amount, approvals, evidence | |
| Payout | request(s), rail, bank trace ID, status, return reason | |
| LedgerEntry | account, debit/credit, cohort year, source event | Double-entry, append-only |
| GovernancePerson | role (officer, director, committee), family links | Feeds disqualified-person screen |
| RuleVersion | parameter, value, effective dates, citation | Referenced by every decision |
| AuditEvent | actor, action, object, before/after hash, timestamp | Hash-chained |

## Recommended technology stack (SaaS-first)

Buy every commodity capability from a SOC 2-certified vendor and build only three things: the §25F rules engine, the scholarship workflows, and the compliance reporting layer. Money never sits with you: funds stay in the SGO's own FDIC-insured accounts and you orchestrate movement through API rails, which avoids money-transmitter licensing.

| Layer | Recommended | Alternative | Why |
| --- | --- | --- | --- |
| Web app and portals | Next.js + TypeScript on **Vercel** | AWS Amplify | One codebase for staff, family, donor, school portals; fast preview deploys |
| UI kit | Tailwind + shadcn/ui; i18n via next-intl | MUI | Accessible primitives, Spanish from day one |
| Database | **Supabase** Postgres (row-level security, PITR) or **Neon** | AWS RDS / Aurora | Managed Postgres with RLS for tenant isolation |
| Durable workflows | **Temporal Cloud** | Inngest | Approvals, payout batches, Jan 31 / Feb 28 runs need retries and exactly-once steps |
| Auth | **Clerk** (orgs, MFA, passkeys) + **WorkOS** for SGO SAML SSO | Auth0 | Consumer and enterprise login in one place |
| Donations in | **Stripe** (cards, Apple/Google Pay, ACH via Financial Connections), Stripe Checkout hosted fields; Stripe Connect (Standard) so each SGO processes on its own Stripe account and card numbers never touch platform servers | Give Lively, Classy | PCI scope stays at SAQ-A; recurring gifts and disputes handled |
| Bank accounts and payouts | **Modern Treasury** (payment ops, reconciliation, **Ledgers**) over SGO-owned accounts at a partner bank such as **Column** or **Increase** | Increase alone (accounts + ACH + checks) | Dedicated Tennessee §25F account per SGO; ACH, same-day ACH, checks; reconciliation without custody |
| Bank account verification | **Plaid** Auth + Identity | Stripe Financial Connections | Instant verification of school and vendor payout accounts |
| School/vendor KYB and sanctions | **Middesk** | Persona KYB, ComplyAdvantage | EIN, Secretary of State, OFAC, beneficial owners |
| Family identity | **Persona** | Socure | Document + selfie checks when fraud signals appear |
| Income verification | **Truv** or **Argyle** (payroll), **Plaid Income**, IRS transcripts via an IVES provider | Pinwheel | Auto-verify income with consent; cut manual review |
| Document AI | **Ocrolus** for pay stubs and statements; **Claude API** or AWS Textract for classifying benefit letters | Google Document AI | Extraction with a human reviewer always signing off |
| Address and AMGI | **Smarty** geocoding + HUD Income Limits API, replaced by IRS tables when published | Census Geocoder (free) | Map each household to its income-limit area |
| File storage | **AWS S3** with Object Lock (WORM) + GuardDuty Malware Protection | Cloudflare R2 | Tamper-proof evidence for auditors |
| Sensitive data vault | **VGS** or **Skyflow** | AWS KMS field encryption | Tokenize SSNs, bank numbers, DOBs |
| E-signature | **Dropbox Sign** API | DocuSign | Consent, attestations, committee conflict forms |
| Email, SMS, mail | **Postmark**, **Twilio**, **Lob** (printed acknowledgments) | SendGrid, Customer.io | Deliverability and proof of delivery for Jan 31 notices |
| Reporting and BI | **Metabase Cloud** (embedded) on a read replica | Looker Studio, Hex | SGO dashboards and auditor exports without a warehouse at first |
| Accounting sync | **QuickBooks Online** and **Sage Intacct** connectors | Xero | SGOs keep their GL; platform pushes journal entries |
| Donor CRM sync (P1) | Salesforce Nonprofit Cloud, Bloomerang | HubSpot | For SGOs with an existing CRM |
| Qualified digital wallet (P1) | **ClassWallet** partnership | Odyssey | Non-tuition spending without building a marketplace |
| SOC 2 automation | **Vanta** | Drata | Evidence collection and policies; audit-ready in months |
| Observability | **Sentry** + **Datadog** | Better Stack | Errors, traces, payout and reconciliation alerts |
| Feature flags | **LaunchDarkly** | Statsig | Switch rules on as federal and TDOE guidance lands |
| Support | **Intercom** (bilingual) | Zendesk | Family and donor help in-app |
| Code and infra | GitHub, GitHub Actions, Terraform | GitLab | IaC and reviewed changes for SOC 2 |

**What you build (your IP):** the rules registry and eligibility engine, the 90% cohort tracker, application and award workflows, the payment-request approval engine, the annual artifact generators, and the auditor workspace.

**Rough run-rate (planning estimate, confirm with vendors):** about $5,000 to $15,000 per month in platform subscriptions at launch, before usage. Usage-based costs scale with volume: card processing (Stripe's standard U.S. rate is roughly 2.9% + 30¢, with nonprofit discounts available; ACH is far cheaper), per-check income and identity verifications ($1 to $10 each), and per-payout bank fees. Price the product so card fees are passed to donors optionally ("cover the fee") and platform fees fit inside the SGO's 10% operating share.

**All-AWS fallback:** if an SGO or state demands a single cloud, the same design runs on AWS (Amplify, Aurora Postgres, Step Functions, Cognito, Textract, S3, KMS) with Stripe and Modern Treasury unchanged.

## Roadmap, risks and open questions

Ship in four phases timed to the program's calendar: giving by January 1, 2027, awards by spring, school payments before fall tuition, and the first annual filings by January 31 and February 28, 2028. The first 90% test for 2027 income is not measured until the end of 2028, but the tracker must run from the first gift.

**Roadmap: giving must be live by Jan 1, 2027; everything else follows the school year**

| Phase | Dates | Scope | Gate |
| --- | --- | --- | --- |
| 0 · Launch giving | Nov–Dec 2026 | Giving pages, §25F designation, donor numbers, state accounts and ledger, audit log, SOC 2 start | First gift · Jan 1, 2027 |
| 1 · Award | Jan–Apr 2027 | Applications and docs, AMGI and IRS transcripts, priority and committee, school registration, disqualified-person screen | First awards · 2027–28 school year |
| 2 · Pay schools | May–Aug 2027 | Payment requests, approvals and payouts, enrollment checks, overpayment recovery, wallet partner | Fall tuition paid · Aug 2027 |
| 3 · Prove it | Sep 2027–Feb 2028 | 90% cohort tracker, certification report, audit package, acknowledgments, IRS donor report | Jan 31 and Feb 28 · 2028 filings |

Phase 0 is the only hard external deadline; slipping it means an SGO cannot take credit-eligible gifts at the year's busiest giving moment.

**Suggested team for phases 0–2:** 1 product lead with SGO domain knowledge, 4–5 full-stack engineers (one payments-focused), 1 designer, 1 QA/automation, a fractional security/compliance lead, and outside tax counsel on retainer.

**Risks**

| Risk | Likelihood | Impact | Mitigation |
| --- | --- | --- | --- |
| Final regulations change definitions (income, 85% safe harbor metric, donor number) | High | High | Rules registry with effective dates; feature flags per rule; track comment outcomes after Dec 15, 2026 hearing |
| IRS SGO portal file format or donor-number format arrives late | Medium | High | Generate donor IDs internally, map to IRS format at issuance; build the report exporter behind an adapter |
| Phase 0 misses January 1 | Medium | High | Scope Phase 0 to giving, designation, ledger and receipts only; use Stripe hosted checkout |
| Income-verification fraud or document forgery | Medium | High | Payroll/bank connections first; document AI with tamper checks; random secondary review sample |
| Payment fraud (bank-detail change, fake vendor) | Medium | High | KYB, account verification, change cool-off, dual approval, payee-relationship check |
| TDOE adds Tennessee rules or reporting | Medium | Medium | Configurable TDOE report templates; states may not be more restrictive than federal rules but can require tailored reporting |
| Being seen as a money transmitter | Low | High | Funds always in SGO-owned accounts; counsel opinion before launch |
| Strong incumbents (ClassWallet, FACTS) bundle §25F | High | Medium | Partner on wallets; win on §25F depth, speed and audit proof |

**Decisions and open questions**

- [x] Target customer first: new SGOs in Tennessee (Tennessee-only platform)
- [x] Pricing model: 2% per processed donation, capped at $25 per gift, ACH-first checkout
- [x] Software only; no managed-service tier
- [x] First state configured: Tennessee
- [x] IRS transcript verification at launch: yes
- [ ] Which partner bank holds SGO accounts, and does each SGO open its own accounts or use FBO sub-accounts?
- [ ] School-level donor preference: offer it at all, given the no-earmarking rule? Needs counsel sign-off.
- [ ] Does the 2% fee apply to offline gifts (checks, payroll) recorded in the platform, and is it refunded when a gift is refunded?
- [ ] May donors cover processing and platform fees inside a designated gift? Counsel to confirm.
- [ ] Which IVES participant handles transcripts, at what cost and turnaround per request?
- [ ] Will TDOE issue state rules or report formats beyond the federal requirements?

## Sources

Regulatory

- [Federal Register: Federal Scholarship Tax Credit, proposed regulations (2026-20277)](https://www.federalregister.gov/documents/2026/10/02/2026-20277/federal-scholarship-tax-credit)
- [Federal Register: Federal Scholarship Tax Credit, temporary regulations (2026-20264)](https://www.federalregister.gov/documents/2026/10/02/2026-20264/federal-scholarship-tax-credit)
- [IRS: Federal Scholarship Tax Credit (FSTC) program page](https://www.irs.gov/government-entities/federal-state-local-governments/federal-scholarship-tax-credit-fstc)
- [eftccredit.com: plain-English summary of REG-117199-25](https://eftccredit.com/documents/treasury-proposed-regulations)
- [Dykema: Treasury releases §25F regulations](https://www.dykema.com/news-insights/treasury-releases-section-25f-scholarship-tax-credit-regulations.html)
- [Tax990: §25F for 501(c)(3) organizations](https://blog.tax990.com/2026/10/05/section-25f-scholarship-tax-credit-what-501c3-organizations-need-to-know-before-2027/)

Tennessee

- [TDOE: Tennessee advances participation in the federal education tax credit (Oct 7, 2026)](https://www.tn.gov/education/news/2026/10/7/tennessee-advances-participation-in-the-federal-education-tax-credit-program-.html)
- [TDOE: Scholarship Granting Organizations page](https://www.tn.gov/education/about-the-tdoe/scholarship-granting-organizations.html)
- [HB2187/SB2206, Federal Tax Credit Scholarship Act, with House Amendment 1](https://billtexts.s3.amazonaws.com/tn/SB-2206-TNB00059738Amended-with-HA0888-03-30-2026-2026-03-30.pdf)

Market

- [SGO Software / scholarships.software](https://sgosoftware.com/)
- [SGO Software compliance page](https://sgosoftware.com/compliance)
- [SGO Guide: ClearPath SGO software](https://sgoguide.com/sgo-software)
- [SGO Pro: §25F compliance](https://sgopro.app/compliance)
- [ClassWallet: FSTC offering](https://classwallet.com/fstc)
- [NC ESA+ provider guide (ClassWallet 2.5% fee)](https://k12.ncseaa.edu/media/roqmwfwh/provider-introduction-to-esaplus-july-2025_final.pdf)
- [FACTS: SGO scholarship management](https://factsmgt.com/blog/how-sgos-can-streamline-scholarship-management-at-scale/)
- [Yes Press: Odyssey profile](https://yespress.io/odyssey)
- [Step Up For Students: SFO renewal and EMA](https://www.stepupforstudents.org/wp-content/uploads/SFO-Renewal-Press-Release.pdf)
- [Capterra: scholarship management software](https://www.capterra.com/scholarship-management-software/)

Not tax or legal advice; confirm program rules with tax counsel before launch.
