# Altriq: Supplier Verification for Small and Mid-Sized Finance Teams

*Working plan, October 2026. "Altriq" is a working name taken from the repo.*

## 1. The pitch

> Catch fake "we changed our bank account" emails before you pay them.
> Altriq watches every supplier in your accounting software. It flags any bank-detail change, screens suppliers against sanctions lists and keeps an audit trail. Setup takes 5 minutes.

**Who it's for:** finance teams and bookkeepers at companies with 20–500 employees that run Xero (and later QuickBooks, NetSuite and Odoo). Large companies already buy Trustpair or Eftsure.

> ⚠️ **Update:** smaller companies are *not* unserved. OutflowGuard and VendorAlert already sell bank-change alerts on the Xero App Store, and Ramp and BILL include vendor verification in their bill-pay products. A plain "alert when bank details change" Xero app is a copy. See §7 and §9 before building.

**Why now:**
- Scams that change a supplier's bank details are the most common payment fraud that hits businesses.
- EU banks have had to run Verification of Payee since 9 Oct 2025, and the UK has Confirmation of Payee. These checks only happen at payment time, one transfer at a time. Companies can also switch them off for bulk payment runs, which is how most accounts-payable teams pay suppliers. Nothing watches the supplier records in the ledger, which is where the fraud starts.
- The UK Sanctions List became the only official UK sanctions source on 28 Jan 2026. Many small businesses have no screening at all.

## 2. What the first version does (and doesn't)

### In (first 30 days)
| # | Feature | How |
|---|---|---|
| 1 | **Connect Xero** | OAuth. Read Contacts (suppliers, including bank account details), Bills and Batch Payments |
| 2 | **Baseline snapshot** | Store a hash and an encrypted copy of every supplier's bank details on day 1 |
| 3 | **Change detection** ⭐ | Poll Xero (webhooks when available). Any change to bank details creates an alert and a "hold" item |
| 4 | **Call-back checklist** | The hold item records: who called the supplier, on which phone number (taken from the *baseline*, not the new email), when, and the outcome |
| 5 | **Pre-payment-run check** | Before a batch payment, list every payee whose details changed recently or haven't been verified |
| 6 | **Bank account format check** | IBAN checksum, UK sort code and account number format, BIC lookup |
| 7 | **Sanctions screening** | Screen supplier names against the official OFAC, EU, UN and UK lists. Rescreen weekly and on any change |
| 8 | **Company and VAT check** | EU VIES (free), HMRC VAT API (free, needs registration), Companies House API (free) |
| 9 | **Audit log** | Append-only. Export to PDF for auditors and insurers |
| 10 | **Alerts** | Email first, then Slack or Teams |

### Out (later phases)
- Bank name-to-account matching through a VoP/CoP partner (SurePay, finAPI, or UK CoP access through a payments provider). This needs a partnership.
- Receiving and checking e-invoices (EN 16931 / Peppol)
- Contract signing at supplier onboarding
- A supplier self-service portal where suppliers enter their bank details and prove them
- QuickBooks, NetSuite and Odoo integrations

## 3. Data sources and costs

| Need | Source | Cost |
|---|---|---|
| Accounting data | Xero API | API access depends on Xero's developer pricing tiers (check before building). **15% referral fee** on App Store sign-ups |
| Sanctions (first version) | Official lists hosted yourself: OFAC SDN, EU Financial Sanctions File, UN Consolidated, UK Sanctions List | Free |
| Sanctions (once there's revenue) | OpenSanctions | €0.10 per API call, or an internal-use or reseller license. Commercial use needs a license |
| EU VAT | VIES | Free |
| UK VAT | HMRC Check a UK VAT number API | Free, needs a developer account |
| UK companies | Companies House API | Free |
| IBAN / BIC | Open-source validation libraries (e.g. `ibantools`, `python-stdnum`) plus a BIC directory | Free to cheap |
| Name-to-account match (later) | SurePay / finAPI VoP / a UK CoP provider | Quote-based. Phase 2 |

**Sanctions cost note:** rescreening every supplier daily through a per-call API gets expensive (500 suppliers × 30 days × €0.10 = €1,500/month per customer). Host the official lists and match against them locally, and call the paid API only for fuzzy-match follow-ups.

**Infrastructure:** Next.js and Postgres (e.g. Supabase or Neon), a background worker for polling, rescreening and alerts, and Resend or Postmark for email. **About $50–150/month** until there are real customers.

## 4. Pricing (to test)

| Plan | Suppliers watched | Price |
|---|---|---|
| Starter | up to 100 | £49 / $59 per month |
| Growth | up to 500 | £149 / $179 per month |
| Pro | up to 2,000 + audit PDF + Slack | £399 / $479 per month |
| Bookkeeper / firm | many client companies | £15–25 per client company per month |

One prevented fraud saves an average business thousands to hundreds of thousands, so prices can be framed against that loss. The bookkeeper plan is the growth engine: one accounting firm can bring 20–200 client companies.

## 5. The 30-day build

**Run validation alongside the build. Don't wait until the product is done.**

### Week 1: Foundations and talking to customers
- [ ] Register a Xero developer app. Read the current commercial terms and API pricing tier
- [ ] Next.js + Postgres skeleton, sign-in, Xero OAuth connect
- [ ] Pull Contacts, store the baseline snapshot with bank details encrypted at rest
- [ ] **Validation:** message 30 finance managers or bookkeepers (LinkedIn, r/Accounting, Xero community, local accounting firms). Goal: 10 calls booked

### Week 2: The core feature
- [ ] Change-detection worker (poll every 15 minutes) and hold queue
- [ ] Call-back verification checklist UI
- [ ] Email alerts
- [ ] IBAN / sort code / BIC validation
- [ ] **Validation:** hold 10 calls. Ask: "Has anyone ever sent you fake bank details? What do you do today when a supplier says they changed banks?"

### Week 3: Screening and checks
- [ ] Load the OFAC, EU, UN and UK lists, schedule a daily refresh, add fuzzy name matching
- [ ] VIES, HMRC VAT and Companies House lookups
- [ ] Supplier risk page: one screen per supplier showing every check
- [ ] Append-only audit log and PDF export

### Week 4: Pre-payment check and launch
- [ ] Pre-payment-run report (from Xero Batch Payments / Bills awaiting payment)
- [ ] Billing (Stripe now; Xero App Store subscriptions for listing)
- [ ] Landing page with the pitch, a 2-minute demo video and a pre-sale button
- [ ] Onboard 3 design-partner companies for free or cheap. **Xero certification requires 3 active connections within 30 days**
- [ ] Submit the Xero App Store listing

### Days 31–60
- [ ] Turn design partners into paying customers. Goal: **10 paying customers or 2 accounting firms**
- [ ] Get a VoP/CoP partner quote for name-to-account matching
- [ ] Choose the next integration from customer demand (QuickBooks or NetSuite)

## 6. Risks and how to handle them

| Risk | Response |
|---|---|
| Xero or banks build this themselves | Stay multi-platform (Xero, QuickBooks, NetSuite) and own the workflow and audit trail, not just one check |
| Customers don't trust a new vendor with bank data | Read-only access, encryption at rest, data stored in the UK/EU, a clear security page. SOC 2 / ISO 27001 later |
| Missing a fraud gives a false sense of security | Position as a control plus a checklist, not a guarantee. Clear terms. Professional indemnity and cyber insurance |
| Sanctions false positives annoy customers | Show the match reason and score, let customers whitelist with a recorded reason |
| Some platforms' APIs don't expose bank details | Check each platform's API before promising an integration (Xero Contacts exposes bank account details; check QuickBooks per region) |
| GDPR (bank details are personal data for sole traders) | Data processing agreement, data minimisation, retention policy |

## 7. Competitors

Small-company competitors that already exist:
- **OutflowGuard** (Xero App Store): watches supplier bank-detail changes, pauses suspicious payments, keeps an audit trail. Almost the same pitch as this plan.
- **VendorAlert** (Xero App Store): emails the people you choose when a contact's bank field changes, aimed at businesses and accounting firms.
- **Ramp Bill Pay, BILL**: vendor bank verification and fraud alerts built into their bill-pay products (mostly US).

| | Trustpair | Eftsure | Trustmi / nsKnox | **Altriq** |
|---|---|---|---|---|
| Target customer | Enterprise | Mid-market to enterprise (AU first) | Enterprise | **Small and mid-sized companies, bookkeepers** |
| Integrations | SAP, Oracle, Coupa | ERP / AP | ERP / AP | **Xero first, then QuickBooks, NetSuite, Odoo** |
| Setup | Sales-led project | Sales-led | Sales-led | **Self-serve, 5 minutes** |
| Price | Enterprise contract | Contract | Contract | **From £49/month** |

## 8. Questions to answer in customer calls
1. How do you find out today when a supplier's bank details change?
2. Has fraud like this ever happened to you or a client? How much was lost?
3. Who approves payment runs, and what do they check?
4. Do you screen suppliers against sanctions lists today? Does anyone ask you to (auditors, banks, insurers)?
5. Would you pay £49–149/month for this? Who signs off that spend?

## 9. Honest check: is there room?

- **Demand is real.** The FBI's 2025 IC3 report lists $3.05B of US business email compromise losses (averaging about $123k per incident), and AFP's 2026 survey says 74% of organisations were hit by it in 2025.
- **But the small-business Xero gap is already being filled** (OutflowGuard, VendorAlert). Small companies also tend not to buy prevention until they've been burned.
- **Ways to stand out (test these in customer calls):**
  1. **Accounting-firm dashboard:** one screen covering all of a firm's client companies, sold per client, so one sale brings many companies.
  2. **Insurer angle:** cyber and crime insurers ask for call-back controls before covering social-engineering fraud. Proof that a company passed the control could earn a premium discount, and insurers could become a sales channel.
  3. **Platforms the existing apps don't cover yet:** QuickBooks, Sage, Odoo, NetSuite mid-market.
  4. **More than alerts:** bank name-to-account matching, sanctions checks, VAT checks and a verified supplier record, not just a notification.
- **Before writing code:** check OutflowGuard's and VendorAlert's review counts and pricing on the Xero App Store. If they have few reviews after a long time on the store, that's a warning about demand for the standalone app.

## Sources
- [Trustpair vs Eftsure (Capterra)](https://www.capterra.ae/compare/187396/216234/trustpair/vs/eftsure)
- [Eftsure competitors (CB Insights)](https://www.cbinsights.com/company/eftsure/alternatives-competitors)
- [EU Verification of Payee mandatory from Oct 2025 (CBI)](https://www.cbi-org.eu/Verification-of-Payee-en)
- [finAPI VoP service](https://www.finapi.io/en/products/vop-service/)
- [SurePay Verification of Payee](https://www.temenos.com/solution-provider/surepay/)
- [OpenSanctions licensing](https://www.opensanctions.org/licensing/)
- [Xero Developer Platform commercial terms](https://developer.xero.com/xero-developer-platform-commercial-terms)
- [Xero App Store certification checkpoints (Codat)](https://docs.codat.io/integrations/accounting/xero/partner-certification/checkpoints-app-store)
- [OutflowGuard](https://dolphinvoice.ai/zh/ai-apps/outflowguard-1096483)
- [VendorAlert on the Xero App Store](https://apps.xero.com/app/vendoralert)
- [Ramp vendor verification](https://support.ramp.com/vendor-verification)
- [FBI IC3 2025 report, BEC figures (McDonald Hopkins)](https://www.mcdonaldhopkins.com/insights/news/the-sobering-truth-of-the-fbis-2025-internet-crime-complaint-center-report)
- [AFP 2026 Payments Fraud and Control Survey](https://www.financialprofessionals.org/about/learn-more/press-releases/Details/over-75-percent-of-us-firms-experienced-payments-fraud-in-2025-while-ai-adoption-for-fraud-mitigation-lags)
- [UK move to a single sanctions list, 28 Jan 2026 (GOV.UK)](https://www.gov.uk/guidance/moving-to-a-single-list-for-uk-sanctions-designations-28-january-2026)
