---
type: product-knowledge
title: "Corporate Actions 1.0 - Project Documentation (IPO, Manual Answers, Presubscription, Reserve TP, German Tax)"
status: active
owning_team: "Team Shark"
product_owner: "Madelene Söderström"
last_updated: 2026-07-13
---

# Corporate Actions 1.0 - Project Documentation

This document consolidates the project documentation for the Corporate Actions 1.0 initiative, covering IPO presubscription, manual answers, reservation of trading power, and German tax reporting integration.

---

## 1. Markup Answers (Teckningsatagare / Presubscription)

- **Source:** [Confluence - Markup Answers](https://nordnetbank.atlassian.net/wiki/spaces/DE/pages/495091740)
- **Epic:** EQF-1151, EQF-1308
- **Status:** ACTIVE

### Problem

The existing Corporate Actions pre-subscriber process relies on manual Excel data entry. This approach is error-prone, lacks traceability, creates an operational bottleneck, and introduces employee dependencies.

### Solution

A new NNX admin feature replaces the Excel sheet for pre-subscriber markups. Customer service can create and delete pre-subscriber entries directly in the admin interface. Data is automatically transferred to the final event file.

### Presubscription Types

| Type | Description |
|------|-------------|
| `PRESUBSCRIBEE` | Teckningsatagare |
| `CORNERSTONE_INVESTOR` | Garant |
| `FRIENDS_AND_FAMILY` | Friends and family allocation |
| `EMPLOYEE` | Employee allocation |

### Architecture Decision

**One-way PubSub** from `ca-presubscription` to `corporate-action` on-prem was chosen over REST or two-way PubSub. This approach provides resilience with less complexity.

### Key Stakeholders

| Name | Role |
|------|------|
| Marcus Gangnor | Business |
| Edward Neptune | Business |
| Mathias Persson Almqvist | BA driver |
| Anders Furby | SME |
| Eva von Bell | FIOPS |
| Andreas Persson | PB |
| Tim Kampeberg | KB |

### Scope

**In scope:**
- Create and delete pre-subscriber entries in NNX admin
- Auto-transfer data to compile file

**Not in scope:**
- Edit after creation
- SSN search
- Bulk file upload
- Customer-facing web interface

### Competitive Note

Avanza has started offering employee handling for IPOs. As noted by Marcus: the team needs to "up our game."

---

## 2. Manual Answers

- **Source:** [Confluence - Manual Answers](https://nordnetbank.atlassian.net/wiki/spaces/DE/pages/478740482)
- **Epic:** EQF-1331
- **Status:** PROPOSED

### Problem

Manual customer answers received via phone are tracked in an unvalidated Excel sheet, then copy-pasted into the compile file. This process is error-prone and provides no traceability.

### Solution

An NNX admin feature for customer service to log manual answers. The system uses an append-only database with revision tracking. It integrates with the event-response-handler (ERH) via REST. Answers are automatically added to the compile file.

### Key Insight from Stakeholders

Manual answers are mostly used when an event has closed on the web but is still open per the prospect. Customers call in for late answers. This occurs rarely (less than once per month). The most common use case is for rights issues. Finland does not accept late answers at all.

### Authorization

| Role | Permission |
|------|------------|
| Engineers | View only |
| Customer Service | Modify |

### Decision Maker

The default decision maker is the real account owner. For Power of Attorney (POA) exceptions, CA operations updates Excel manually.

### Initial Pivot

The first attempt focused on automation that did not support late answers, which turned out to be the stakeholders' main use case. The team pivoted and restarted with late answer support in August 2025.

---

## 3. Reservation of Trading Power

- **Source:** [Confluence - Reservation of Trading Power](https://nordnetbank.atlassian.net/wiki/spaces/DE/pages/1089077249)
- **One-pager:** [Confluence - One-pager](https://nordnetbank.atlassian.net/wiki/spaces/DE/pages/1605435477)
- **Epic:** ASB-496
- **Status:** ACTIVE
- **Go-live:** Week 12, 2026 (Swedish flow)
- **GitHub:** [ca-voluntary-event](https://github.com/nordnet-private/ca-voluntary-event)

### Problem

Corporate Actions uses fictitious instruments to reserve cash and positions. This approach is error-prone, affects many departments, and delivers a poor customer experience. The Germany market launch requires fixing this because fictitious instruments make accurate German tax reporting almost impossible.

### Solution

Replace fictitious "Allocation" instruments with direct "Reservation of Trading Power" via the Pre-Trades API. Cash is reserved at the trading power level. When delivery is finalized, the reservation is released and the final trade is created. Nordnet uses its own capital to cover the settlement timing difference.

### What Changes for Customers

**Before:**
1. Sign up
2. Trade in Allocation instrument
3. Exchange to actual shares

**Now:**
1. Sign up
2. Cash reserved (Available to Trade decreases, total equity unchanged)
3. On settlement, reservation is converted to shares

Customers no longer see confusing temporary tickers in their portfolio.

### Benefits

- Better UX with no fake instruments visible
- Cleaner portfolio view
- Real-time accuracy
- Reduced confusion and support calls
- Improved operational risk and compliance
- Fixes incorrect reporting in TRS, Nominee, SRDII, Securities Reconciliation, SFTR, and Pension systems
- Critical enabler for Germany market entry

### Phased Decommission of Fictitious Instruments

| Phase | Scope | Status |
|-------|-------|--------|
| Phase 1 | Allocation instruments (rights issues, IPOs) | Done |
| Phase 2 | Accept instruments (tender offers), TRF/URF instruments | Next (summer goal) |
| Phase 3 | Application instruments (visualization - remains until good alternative) | Future |

### Country-Specific Considerations

**Norway:**
- Cannot go live until reserve position is completed (needs support for rights/extra coverage)
- Two files needed for Norwegian flow

**Finland:**
- Demands multiple files per event
- Demands real-time reservation when customer subscribes
- Demands ability to take out a fee (interest rate)
- Not fully supported in MVP1

### Key Decisions

| Decision | Detail |
|----------|--------|
| File format | Single file per event in MVP1 (not multiple) |
| Account deletion | No individual account deletion; abort all and re-upload corrected file |
| Four-eye principle | Not enforced in MVP1 |
| File format | Semicolon CSV |
| Booking accounts | 1516 for SEK, 1517 for FX bookings |

### Norwegian Shareholders Disclosure

An endpoint is provided for reporting reserved positions (account, ISIN, shares) to satisfy the 5% disclosure obligation.

---

## 4. German Tax Reporting (Sectras Integration)

- **Source:** [Confluence - German Tax Reporting](https://nordnetbank.atlassian.net/wiki/spaces/DE/pages/1240793089)
- **Status:** ACTIVE
- **Admin:** [German Tax Reporting Admin](https://corporate-action-admin.tools.prod.nntech.io/german-tax-reporting)

### Problem

Germany market entry requires correct German tax calculation. The third-party vendor Sectras needs CA event data enriched with transaction data. Team Hodor reports to Sectras but needs Team Shark's CA data.

### Architecture

Three services form a pipeline:

```
ca-poller --> ca-bookkeeper --> ca-sectras
```

The pipeline polls Abasec, transforms the data, and serves it to Sectras.

### Deliverables

| # | Deliverable | Target | Status |
|---|-------------|--------|--------|
| 1 | CEG interface support | Q1 | Ready for Testing |
| 2 | TRDG interface support | Q2 | Ongoing |
| 3 | Event filtering for German customers | Q2 | Not started (decided to use transaction-master data) |
| 4 | Automatic Sectras event creation via MQ | TBD | Not started |
| 5 | Replace on-prem Abasec poller with PubSub | TBD | Not started |

### Key Technical Decisions

**Data model:**
"God object lite" approach. Publish an internal PubSub message that no other service uses. Strangle out event by event as data becomes available in NNX.

**Sectras Event Reference format:**
`ISIN_EVENT_TYPE_DATE` with collision handling via suffix.

**BID reference:**
Sourced from WM DATA. Needed when multiple dividends occur on the same ISIN and date.

**Multi-events:**
- Rights issue maps 4 Abasec events to 2 Sectras events
- LRN and SUB events are connected
- `_BZR` suffix used for loosening rights

**FLEX events:**
Can have multiple incoming instruments, each potentially needing separate Sectras IDs.

**ROC events:**
Creates both Dividend (SE/FI) and ROC (NO/DE) events. ROC is reported via TRDG.

**Late tax reporting:**
Foreign withholding tax may be charged retroactively. This is not a dealbreaker for launch (limited to employees initially). No concrete technical solution exists yet.

**MVP3 filtering:**
Listen to `transaction-master` PubSub to identify German customers. This was chosen over Abasec polling or Hodor positions.

**Currency exchange rates:**
Accepted as-is from Abasec. Only updated if manually corrected.

**1:1 same-ISIN same-ratio exchanges:**
Classified as `NON_REPORTABLE` (decision by Felix Mandoki).
