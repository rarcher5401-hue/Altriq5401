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
- **OutflowGuard**: watches supplier bank-detail changes, pauses suspicious payments, requires two-person approval, sends alerts to Slack and Teams, keeps an audit trail. Aimed at accountants and bookkeepers. Free and paid plans (prices not public). Barely indexed online; looks like an early indie product.
- Neither company publishes its revenue (MRR). At A$9/month, $10k MRR would take about 1,700 paying Xero organisations, so expect pricing pressure if you compete on alerts alone.
- **VendorAlert** (Xero App Store): emails the people you choose when a contact's bank field changes, aimed at businesses and accounting firms. **Price: A$9/month per Xero organisation** (14-day free trial). Read-only, checks about every 6 hours. Listed in Xero's "New and noteworthy" collection, so probably launched recently.
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

## 10. Product roadmap

The basic bank-change alert is already sold for A$9/month (VendorAlert), so it can't be the whole product. The first version (§5) gets you into Xero and gets customers connected. Each phase after that adds something the A$9 apps don't have.

### Phase 0: First version (days 1–30), as in §5
Xero connection, bank-change detection with hold and call-back checklist, bank account format checks, sanctions/VAT/company checks, audit log, email alerts.
**Design now for later:** store a fingerprint of every bank account (hashed) and every verification result, so the fraud network (Phase 3) has data from day one.

### Phase 1: Stand out (days 31–90)
| Feature | What it does | Why it matters |
|---|---|---|
| **Invoice inbox scanning** ⭐ | Connect the accounts-payable mailbox (Gmail / Microsoft 365). Read invoice PDFs, compare bank details with the supplier's verified details, flag lookalike domains (`acme-co.com` vs `acmeco.com`), first-time senders and "we changed banks / urgent" wording | Catches fraud *before* anyone enters it into Xero. Competitors only see it afterwards |
| **Accounting-firm dashboard** | One screen covering every client company, alerts grouped by client, branded monthly "your payments are protected" report | The growth channel: one firm brings 20–200 client companies |
| **Call-back workflow done properly** | Phone number taken from a trusted source (Companies House, the supplier's website, the baseline record), never the email. Records who called, when, outcome, notes | Turns a checklist into evidence an auditor or insurer will accept |
| Slack / Teams alerts | Alerts with approve/hold buttons | Matches OutflowGuard |

### Phase 2: Protect the payment (months 3–6)
| Feature | What it does |
|---|---|
| **Payment file check** ⭐ | Upload or intercept the bank payment file (BACS, SEPA XML, ABA, NACHA) before it goes to the bank and compare every payee with verified details. Catches tampering outside Xero |
| **Supplier self-service portal** | Supplier enters bank details via a secure link and uploads a bank letter or statement. AI checks the document (name, account and logo consistency, signs of editing). No more bank details by email |
| **Payment anomaly alerts** | New supplier with a large first payment, duplicate invoices, an unusual amount for this supplier, a bank in a different country from the supplier, a payment just after a bank change |
| **Xero access monitoring** | Alert on a new Xero user, a role change, or an edit from someone outside the finance team. Many frauds start with a compromised login |
| **Insurance evidence pack** | One-click PDF of the controls in place and their history, for cyber or crime insurance applications and claims |

### Phase 3: Make it hard to copy (months 6–12)
| Feature | What it does |
|---|---|
| **Shared fraud network** ⭐ | A bank account flagged as fraudulent at one customer is flagged for all. A supplier account already verified by many customers shows higher confidence. Uses hashed account fingerprints only. Competitors can't copy this without the same customers |
| **Bank account ownership check** | Name-to-account match through a UK Confirmation of Payee / EU Verification of Payee partner (SurePay, finAPI), Plaid or micro-deposits in the US, and Australia's Confirmation of Payee as it rolls out |
| **Company status alerts** | Supplier dissolved, in liquidation, directors changed, newly sanctioned (Companies House streaming API plus the sanctions refresh) |
| **More platforms** | QuickBooks Online next, then NetSuite, Sage and Odoo, chosen by customer demand |
| Insurer partnerships | Premium discounts for customers who pass the controls, with insurers sending you customers |

### Pricing by plan
| Plan | Includes | Price |
|---|---|---|
| **Free / Basic** | Bank-change alerts, format checks | Free–£9/month per company (matches VendorAlert, brings customers in) |
| **Protect** | + call-back workflow, sanctions/VAT/company checks, audit log, Slack/Teams | £49/month |
| **Shield** | + invoice inbox scanning, payment file check, anomaly alerts, supplier portal, insurance pack | £149/month |
| **Firm** | Firm dashboard + Protect or Shield for every client company | £15–40 per client company per month |
| **Network add-ons** (Phase 3) | Ownership checks | Per check, passed through at a markup |

### What to measure at each phase
- **Phase 0:** 3 connected Xero companies (needed for Xero certification), 10 customer calls
- **Phase 1:** 10 paying customers or 2 accounting firms. Share of invoices scanned that produce a useful alert
- **Phase 2:** £5k MRR. Number of payment files checked. At least one documented fraud caught (your best marketing)
- **Phase 3:** £20k+ MRR. Number of companies in the fraud network. Monthly cancellation rate under 2%

## 11. Insurance partners (US)

**Why insurers care:** crime and cyber policies cover "social engineering fraud" (paying a fake supplier), usually only up to $25k–$250k. Many of these policies only pay out if the customer **did a call-back check** before changing bank details or sending money. Altriq's recorded call-back workflow and audit log are proof of that check.

| Type | Examples | Role |
|---|---|---|
| **Cyber insurers that also do security scanning (start here)** | Coalition, At-Bay, Cowbell, Corvus (Travelers), Resilience, Embroker, Vouch | Already partner with security tools and give customers discounts or recommendations. Fastest to work with |
| **Crime and cyber insurance companies** | Chubb, Travelers, AIG, CNA, Beazley, The Hartford, Zurich, Great American, Intact, Markel | Write social-engineering cover. Slow to partner but large |
| **Large brokers** | Marsh, Aon, WTW, Gallagher, Lockton | Advise mid-sized clients on controls and could recommend Altriq |
| **Independent agents** | ~37,000 agencies | Sell to small businesses. Reach them through Big I and agency networks |
| **Wholesale brokers** | Amwins, CRC, RT Specialty | Place specialist and hard-to-place cover |

**Market size:** about 3,900 property, casualty and direct insurance companies in the US (IBISWorld, 2025). In cyber, the top 30 insurance groups write over 90% of admitted premium, and US cyber premiums were about $9.1B in 2024 (NAIC). Realistically there are **15–30 companies worth approaching**.

**Order:** get customers and evidence first (insurers want data on losses prevented). Then approach the cyber insurers that already partner with security tools, then brokers, then large insurers.

## 12. Who needs this most (target customers)

**The ideal customer:** pays 50+ suppliers by bank transfer (ACH, wire, BACS, SEPA), has invoices over $5k, a finance team of 1–5 people, runs Xero or QuickBooks, receives invoices by email, and has cyber or crime insurance. Best of all if they've already had a fraud attempt.

| Industry | Why they're at risk | Priority |
|---|---|---|
| **Construction and contractors** | Many subcontractors and suppliers, large progress payments, new subcontractors every job, bank changes are routine, finance teams are small | ⭐ Start here |
| **Property management and real estate** | Pay many contractors and vendors, handle owners' and tenants' money, frequent targets of email fraud | ⭐ Start here |
| **Logistics and freight** | Pay many carriers, fake carriers and payment redirection are common | High |
| **Manufacturing, wholesale and importers** | Many suppliers, often overseas, large invoices, cross-border payments | High |
| **Law firms (especially conveyancing / settlements)** | Move large client sums. A loss also damages their reputation and brings regulator trouble | High (higher price) |
| **Nonprofits, schools, churches, local councils** | Weak controls, frequent fraud victims, public money | Medium (slow buyers, small budgets) |
| **Healthcare groups and clinics** | Many vendors, busy and small admin teams | Medium |
| **Car dealerships** | Large payments to manufacturers, finance companies and vendors | Medium |
| **Accounting firms and bookkeepers** | Pay suppliers on behalf of clients and are blamed if fraud gets through | ⭐ The channel to reach all of the above |

**Not a good fit:** very small businesses with few suppliers or who pay by card, and large companies already using Trustpair or Eftsure.

**How to reach them:** accounting firms that specialise in construction or property management. One firm brings dozens of companies in the same industry, so the marketing can speak that industry's language ("protect your progress payments").

## 13. Sales channels: accounting firms and independent insurance agents

Both channels reach many customers through one relationship.

### Accounting firms and bookkeepers (first channel)
- Work in their clients' Xero / QuickBooks every day and often run their payments
- Are blamed when fraud gets through, so they have their own reason to buy
- **Offer:** the Firm plan (£15–40 per client company per month), a white-label client report, and a partner listing on the Xero and QuickBooks app stores

### Independent insurance agents (second channel, after the first case studies)
**What they are:** local insurance agencies that are their own businesses and sell policies from many insurers (Travelers, Chubb, The Hartford, etc.), unlike captive agents who work for one company (State Farm, Allstate). There are about **37,000** in the US, averaging about 10 staff, and they sell business insurance (liability, property, workers' comp, cyber, crime) to exactly our target customers.

**Why they'd recommend Altriq:**
1. They already insure many contractors, property managers and other target businesses
2. They review each client's risks at every yearly renewal, a natural moment to recommend controls
3. A client's fraud loss means a hard claim, an unhappy client and a higher premium, which hurts the agent too
4. Many social-engineering fraud policies only pay out if a call-back was done. Altriq records it, so claims are more likely to be paid

**How to work with them:**
| Model | Details |
|---|---|
| Referral commission | 10–20% of the subscription, recurring for as long as the client stays |
| Co-branded offer | "Clients of Smith Insurance get 3 months free" |
| Renewal pack | Agent gives clients an Altriq "payment fraud controls" report to attach to their insurance application |
| Agent dashboard (later) | Shows which of the agent's clients are protected. Helps them place cover and argue for better premiums |

**How to find them:** state associations of the Big I (Independent Insurance Agents & Brokers of America), agency networks and clusters that group hundreds of agencies (e.g. SIAA, Iroquois, Renaissance Alliance), local agency events and LinkedIn. Start with agencies that specialise in construction or real estate.

## 14. More features: AI, non-AI and financial services

Added to the backlog. The suggested phase refers to the roadmap in §10.

### AI features
| Feature | What it does | Phase |
|---|---|---|
| **AI call-back assistant** ⭐ | Places the verification call to the supplier's *trusted* number, asks them to confirm the new bank details, records and transcribes the call, and attaches it to the audit log. Turns a chore staff skip into one click | 2 |
| **Email thread-hijack detection** | Spots a fraudster replying inside a real email thread: reply-to mismatch, SPF/DKIM/DMARC failures, a new sender in an old thread, changes in writing style | 1–2 |
| **Forged document detection** | Checks bank letters and invoice PDFs for editing (metadata, font and layout mismatches, logo copies) | 2 |
| **Plain-English alert explanations** | Every alert says why in one sentence ("Bank changed 2 days after a new email domain first appeared; supplier's last 14 payments went to another bank") | 1 |
| **"Is it safe to pay?" assistant** | Ask about any supplier or payment and get an answer with its sources across all the checks | 2–3 |
| **Fraud-awareness training simulations** | Send finance staff realistic fake "we changed our bank" emails and short training. Insurers often require this training | 2 |
| **Supplier risk score** | Combines company status, sanctions, negative news, payment history and bank-change history into one score | 3 |
| **Invoice capture into Xero / QuickBooks** | Read invoices and create draft bills automatically. Adds everyday time savings, not just protection (competes with Dext / Hubdoc, so add it only if customers ask) | 3 |

### Non-AI features
| Feature | What it does | Phase |
|---|---|---|
| **Free "supplier fraud health check"** ⭐ | One-time scan: duplicate suppliers, suppliers sharing a bank account, missing details, recent bank changes, dormant suppliers. The best way to win customers | 0–1 |
| **Lookalike domain monitoring** ⭐ | Watches for newly registered domains that look like the customer's or their top suppliers' (`acme-c0.com`). Warns *before* the attack starts | 1 |
| **Email security check of suppliers** | Shows which suppliers have weak email security (no DMARC), so are easier to impersonate | 1 |
| **Two-person approval for bank changes** | Requires a second person to approve. Enforces separation of duties | 1 |
| **"We've been scammed" emergency button** | Step-by-step guide: call the bank to recall the payment (first 24–72 hours matter), report to FBI IC3 (whose Recovery Asset Team can sometimes freeze wires), notify the insurer. Pre-filled from the audit log | 1 |
| **US W-9 collection and IRS TIN matching** | Supplier portal collects W-9s and checks name and tax ID against IRS records. Also helps with 1099s | 2 |
| **Mobile approvals** | Approve or hold changes and payment runs from the phone | 2 |
| **Policy templates** | Payment-fraud control policy, call-back procedure and staff checklist for audits and insurance applications | 1 |
| **Owner / board report** | Monthly summary: checks done, frauds blocked, controls in place | 1 |
| **Bank Positive Pay files** | Generate Positive Pay files (the bank only pays approved payees and amounts) for US banks that offer it | 3 |

### Financial-services add-ons (extra revenue)
| Service | How it makes money | Phase |
|---|---|---|
| **Payment guarantee** | Partner with an insurer: if Altriq marks a payment as verified and it turns out to be fraud, it's covered up to a limit. Charge a premium; the insurer takes the risk | 3+ |
| **Verified supplier payments** | Pay suppliers through Altriq using only verified accounts, with a payments partner (e.g. Stripe Treasury, Modern Treasury, Wise Platform) handling the money. Earn a fee per payment | 3+ |
| **International payments** | Cheaper currency exchange for overseas suppliers through a payments partner. Earn on the exchange margin | 3+ |
| **Early payment / supplier finance** | Suppliers can be paid early for a small discount, funded by a lending partner | Later |
| **Cyber / crime insurance referrals** | Customers can get quotes from partner insurers through Altriq. Earn a referral or commission fee (needs an insurance license, or work through a licensed partner) | 3 |

*Note:* moving money or selling insurance brings licensing and regulatory requirements. Always go through licensed partners rather than doing it yourself.

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
- [NAIC 2025 Cybersecurity Insurance Report](https://content.naic.org/sites/default/files/inline-files/2025_Cybersecurity_Insurance%20Report.pdf)
- [How cyber and crime policies respond to social engineering (Amwins)](https://www.amwins.com/resources-and-insights/market-insights/article/how-cyber-and-crime-insurance-policies-respond-to-social-engineering)
- [Social engineering endorsement with call-back provision (Intact)](https://portal.intactinsurance.com/system/files/mydocuments/M350%20%2803-20%29%20v1%20Social%20Engineering%20Endorsement%20-%20Call%20Back%20Provision.pdf)
- [Big I 2026 Agency Universe Study (Insurance Business)](https://www.insurancebusinessmag.com/us/news/technology/independent-agency-revenue-rose-at-three-in-four-firms-as-ai-adoption-tripled-big-i-study-finds-590958.aspx)
- [UK move to a single sanctions list, 28 Jan 2026 (GOV.UK)](https://www.gov.uk/guidance/moving-to-a-single-list-for-uk-sanctions-designations-28-january-2026)
