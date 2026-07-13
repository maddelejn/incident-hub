---
type: product-knowledge
title: "CSLA Retail Lending Workshop - April 2026"
status: active
date: 2026-04-21
participants:
  nordnet: "Quincy Curry, Kristofer Burén, Madelene Söderström, Hampus Holmertz, Aleksander Sekulic, Mattias Olausson, William Sörensen"
  citi: "Ismail Ibrahim, Peter Lennon"
  sharegain: "Patrick Sykes, Keren Halperin, Benjamin Smith, Ronny Maate, Oliver Blease"
last_updated: 2026-07-13
---

# CSLA Retail Lending Workshop - April 2026

## Tri-Party Structure

The retail securities lending solution involves three parties:

| Party | Role |
|-------|------|
| **Nordnet** | Lender / Platform (customers' shares are lent out) |
| **Citi** | Custodian (holds the assets, handles market-level trade instructions) |
| **Sharegain (CSLA)** | Lending agent / Platform (manages the lending program, finds borrowers, handles allocation and revenue) |

**CSLA** = Citi Securities Lending Agency (the lending program operated via Sharegain's technology)

## Architecture: Data Flow Overview

```
┌──────────┐                    ┌──────────┐                    ┌──────────┐
│          │  Position file     │          │  Loan instructions │          │
│          │  (daily, pre-mkt)  │          │  (MT54X format)    │          │
│          │ ──────────────────►│          │ ──────────────────►│          │
│  Nordnet │  Transaction file  │  CSLA    │  Loans report      │   Citi   │
│          │  (intraday/hourly) │(Sharegain)│  Revenue report    │(Custodian)│
│          │ ──────────────────►│          │  Collateral report │          │
│          │                    │          │ ──────────────────►│          │
│          │◄───────────────────│          │                    │          │
│          │  Reports (loans,   │          │                    │          │
│          │  revenue, collat.) │          │                    │          │
└──────────┘                    └──────────┘                    └──────────┘
         All file delivery via SFTP flat files
```

## Inbound Data: Nordnet → CSLA

### Position File

- **Frequency:** Once per day, Mon-Fri, outside market hours
- **Granularity:** Per client (anonymised), per stock
- **Content:**
  - Total units (factoring in on-loan units)
  - Contractually settled units
- **Securities identified by:** ISIN, ticker, exchange code, PSET identifiers
- **Purpose:** Creates a "start of day" snapshot of lendable inventory per stock

### Transaction File

- **Frequency:** Intraday, Mon-Fri (e.g., every hour)
- **Granularity:** Per client (anonymised), per stock
- **Content:**
  - Buys and sells at the point of transaction booking
  - Standard contractual settlement date per market
  - Each transaction reported individually (preferred) or updated via unique transaction ID
- **Securities identified by:** ISIN, ticker, exchange code, PSET identifiers

### Daily Logic: "Flush & Fill"

```
Start of day:
  Nordnet sends Position File → CSLA creates lendable inventory snapshot
                                     │
During the day:                      ▼
  Nordnet sends Transaction Files → CSLA nets buys/sells against position
                                     │
                                     ▼
                              Position file = ceiling on lendable supply
                              Outstanding sells with future settlement
                              dates remain encumbered until settlement
                                     │
Next day:                            ▼
  New Position File → "Flush & Fill" → fresh inventory snapshot
```

## Opt-In: How Customers Enroll

Two approaches supported by CSLA:

### Option 1: Pre-filtered Reporting
- Nordnet only reports holdings and transactions of customers who have opted in
- Simpler data flow, less data sent

### Option 2: Full Reporting + Opt-In Flag File
- Nordnet reports ALL customer holdings and transactions (anonymised)
- Separate "client reference" file flags which customers are opted in for that day
- File refreshed daily (customers can opt out at any time)

### Revenue Concentration Insight

From experience with other lenders:
- **~0.5% of clients can represent 50%+ of revenue potential**
- Efficient targeting and marketing to high-value lenders is critical
- Sharegain offers "MAX" opt-in product to support ongoing targeting
- CSLA can provide non-enrolled stock breakdown to identify key targets

## Outbound Data: CSLA → Nordnet

### Market Trade Instructions
- **Format:** MT54X (SWIFT message format)
- **Same as existing pension lending** - Nordnet books these at custody level
- Handled by Citi as custodian

### Reports from CSLA

| Report | Frequency | Content |
|--------|-----------|---------|
| **Loans report** | Daily (Mon-Fri) | Per-client loans booked that day |
| **Collateral report** | Daily (Mon-Fri) | Per-client collateral pledged |
| **Revenue report** | Daily (Mon-Fri) | Unreconciled revenue accrued (optional) |
| **Billing report** | Monthly | Reconciled final billing |

### Data Handling

- **CSLA handles all allocation and calculations** - Nordnet does not need to calculate allocations
- Reports are on a **per-client basis**
- Nordnet stores client data for:
  - End of month reports
  - Customer-facing display in app/website (depending on program visibility decision)
- On-loan positions tracked via **"Flush & Fill"** - use latest CSLA loans report to update daily
- Nordnet can reconcile CSLA loans report against MT54X market instructions received during the day (same approach as pension lending)

## Comparison: Retail vs Pension Lending

| Aspect | Pension (Existing) | Retail (New) |
|--------|-------------------|--------------|
| **Platform** | On-prem | NNX (cloud) |
| **CSLA program** | Same CSLA | Same CSLA |
| **MT54X instructions** | Yes | Yes (same setup) |
| **Opt-in** | N/A (program-level) | Per-customer opt-in/opt-out |
| **Reporting** | Existing reports | Same reports + customer-facing display |
| **Custody** | Citi | Citi |
| **Data reconciliation** | MT54X matching | Same approach |

## Key Contacts

### Citi
| Person | Role | Email |
|--------|------|-------|
| Peter Lennon | CSLA Program | peter.lennon@citi.com |
| Ismail Ibrahim | CSLA | - |

### Sharegain (CSLA Technology)
| Person | Role |
|--------|------|
| Patrick Sykes | - |
| Keren Halperin | - |
| Benjamin Smith | - |
| Ronny Maate | - |
| Oliver Blease | - |

### Nordnet
| Person | Role |
|--------|------|
| Quincy Curry | - |
| Kristofer Burén | - |
| Madelene Söderström | PO |
| Hampus Holmertz | - |
| Aleksander Sekulic | - |
| Mattias Olausson | - |
| William Sörensen | - |

## Next Steps (from April 2026)

| Action | Owner | Target | Status |
|--------|-------|--------|--------|
| Provide example files for all in-scope files | CSLA | April 24 | TBD |
| Confirm depos for all Citi-custodied markets for retail SSIs | Citi | April 24 | TBD |
| File walkthrough with Nordnet team on file logic & fields | CSLA + Nordnet | May 6 | TBD |
| Corporate event handling session for on-loan retail positions | Nordnet + CSLA | May 6 | TBD |
| Confirm fields Nordnet can populate in inbound reports | Nordnet | May 6 | TBD |
| Rollout plan & opt-in strategy planning (GTM) | Nordnet + CSLA | TBC | TBD |
| Development breakdown, timeline & resource estimation | Nordnet | TBC | TBD |
