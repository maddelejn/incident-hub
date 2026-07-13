---
type: product-knowledge
title: "CSLA Retail Integration Overview - Technical Architecture"
status: active
last_updated: 2026-07-13
---

# CSLA Retail Integration Overview

## Integration Architecture

### Lifecycle Overview

```
          NORDNET                           CSLA (Sharegain)                    CITI (Custodian)
            │                                    │                                   │
  ┌─────────┴─────────┐              ┌──────────┴──────────┐              ┌─────────┴─────────┐
  │ Start of Day       │              │                     │              │                   │
  │ Send Position File │─────────────►│ Calculate lendable  │              │                   │
  │ (daily, pre-mkt)   │              │ pool per client     │              │                   │
  └─────────┬─────────┘              │ per stock           │              │                   │
            │                        └──────────┬──────────┘              │                   │
  ┌─────────┴─────────┐                         │                        │                   │
  │ Intraday           │              ┌──────────┴──────────┐              │                   │
  │ Send Transaction   │─────────────►│ Net buys/sells      │              │                   │
  │ Files (hourly)     │              │ against positions   │              │                   │
  └─────────┬─────────┘              └──────────┬──────────┘              │                   │
            │                                   │                        │                   │
            │                        ┌──────────┴──────────┐              │                   │
            │                        │ Match supply with    │              │                   │
            │                        │ borrower demand      │──────────────►│ Settlement via   │
            │                        │ Execute loans        │              │ MT54X messages   │
            │                        └──────────┬──────────┘              │                   │
            │                                   │                        └─────────┬─────────┘
  ┌─────────┴─────────┐              ┌──────────┴──────────┐                       │
  │ End of Day         │◄─────────────│ Generate reports:    │                       │
  │ Receive reports:   │              │ - Lent Positions     │                       │
  │ - Loans            │              │ - Collateral         │                       │
  │ - Collateral       │              │ - Custody Loans      │                       │
  │ - Revenue          │              │ - Revenue / Billing  │                       │
  └────────────────────┘              └─────────────────────┘                       │
```

### Fee Structure

Revenue flows through a three-tier split:

```
Borrower pays lending rate (e.g., 7.5% annualized)
    │
    ├── CSLA agent fee (e.g., 20%) ──► CSLA/Sharegain
    │
    └── Net to Nordnet (e.g., 80%)
         │
         ├── Nordnet share (e.g., 50%) ──► Nordnet revenue
         │
         └── Client share (e.g., 50%) ──► Customer earnings
```

**Example from sample data:**
- Gross lending rate: 7.5%
- CSLA fee split: 20% → CSLA gets 1.5%
- Nordnet fee split (of remaining): 50% → Nordnet gets 3.0%
- Client fee split: 50% → Client gets 3.0%
- Lending rates in reports: Gross Bank = 0.075, Net Bank/Gross Client = 0.06, Net Client = 0.03

### Collateral Structure

- **Type:** Non-Cash (typically government bonds)
- **Margin:** 100-105% of loan value
- **Triparty Agent:** JPM Bank Luxembourg S.A. (manages collateral pool)
- **Daily mark-to-market:** Collateral values updated daily with haircuts applied
- Collateral includes: instrument details, ISIN, ratings (Moody's, S&P), maturity dates

### Settlement & Custody

- Market instructions use **MT54X SWIFT message format** (same as pension)
- Settlement via **Citi as custodian**
- Depositary/PSET identifiers used for routing (e.g., `DTCYUS33` for DTC US)
- Custody loans tracked at omnibus account level (Nordnet as lender entity "NRDNT")

## File Specifications

### Inbound Files (Nordnet → CSLA)

#### Position File
- **Frequency:** Daily, Mon-Fri, outside market hours
- **Format:** CSV via SFTP

| Field | Example | Format | Required |
|-------|---------|--------|----------|
| ISIN | US0378331005 | String (12) | M |
| Ticker | AAPL US | String (64) | M |
| Exchange | US | String | M |
| External Instrument ID | | String | O |
| Quantity | 100 | Integer | M |
| Currency | | | O (not relevant) |
| Client ID | 12342342 | String (anonymised) | M |
| Account ID | 10001 | String (omnibus custody account) | M |
| Tax rate | | | O (not relevant) |
| Position Status | | | O (not relevant) |
| Depositary | DTCYUS33 | String (PSET) | M |

#### Transaction File
- **Frequency:** Hourly, Mon-Fri intraday

| Field | Example | Format | Required |
|-------|---------|--------|----------|
| Transaction ID | 1234567 | String | O |
| Client ID | 12342342 | String (anonymised) | M |
| Account ID | 10001 | String | M |
| ISIN | US0231351067 | String (12) | M |
| Ticker | AMZN US | String (64) | M |
| Exchange | US | String | M |
| External Instrument ID | | String | O |
| Buy / Sell | Sell | String | M |
| Units | 35 | Integer | M |
| Trade Date | 2026-02-10 | dd/MM/yyyy | M |
| Settlement Date | 2026-02-11 | dd/MM/yyyy | M |
| Depositary | DTCYUS33 | String (PSET) | M |

#### Client Reference File
- **Frequency:** Daily, Mon-Fri (opt-in/opt-out management)

| Field | Example | Format | Required |
|-------|---------|--------|----------|
| Date | 2026-04-27 | dd/MM/yyyy | M |
| Client ID | Client 1 | String | M |
| Opt In | In / Out | String | M |
| Fee Split | 50 | Integer (%) | O |
| Custom | TIER 1 Client | String (Nordnet custom field) | O |

### Outbound Files (CSLA → Nordnet)

#### Lent Positions Report (Client Level)
- **Frequency:** Daily, Mon-Fri
- **47 fields** per row including:
  - Loan identifiers (Loan ID, Actual Loan ID)
  - Loan details (status, type, collateral type, units)
  - Lending rates at three levels (Gross Bank, Net Bank/Gross Client, Net Client)
  - Fee splits (Bank %, Client %)
  - Dates (trade, collateral, settlement, end)
  - Client identifiers (anonymised lender code, account number)
  - Borrower details (code, name - e.g., "BOFA SECURITIES, INC.")
  - Instrument details (name, ISIN, ticker, BBGID, type, country)
  - Valuation (market price, FX rate, loan value in loan currency)
  - Tax and margin (withholding tax rate, margin %, haircut %)
  - Accruals at four levels (Gross, Agent, Net Bank, Net Client)

#### Collateral Report (Client Level)
- **Frequency:** Daily, Mon-Fri
- **29 fields** including: triparty agent, borrower details, collateral instrument details (ISIN, ticker, type, issuer, maturity, ratings), quantities, valuations, FX rates, haircuts

#### Client Revenue Report
- **Frequency:** Monthly (reconciled)

| Field | Example | Description |
|-------|---------|-------------|
| Billing Month | 01/2026 | Month of accrual |
| Client Code | 12342342 | Anonymised client |
| Billing Currency | SEK | Revenue currency |
| Net Client Revenue | 237.62 | What to pay the customer |
| Net Aggregator Revenue | 237.62 | Nordnet's share |

#### Custody Loans Report
- **Frequency:** Daily, Mon-Fri
- **42 fields** - maps to Citi's custody-level loan view
- Used to track on-loan units via a "pseudo-custodian" and reconcile against MT54X instructions
- Shows Nordnet as lender entity ("NRDNT" / "Nordnet AB")

#### Billing Summary Report
- **Frequency:** Monthly

| Field | Example | Description |
|-------|---------|-------------|
| Billing Month | 01/2026 | |
| Borrower Code | BAML US | |
| Billing Currency | SEK | |
| Gross Accrual | 594.05 | Total lending revenue |
| Agent Accrual | 118.81 | CSLA's cut |
| Net Client Revenue | 237.62 | Customer's share |
| Net Accrual | 475.24 | Nordnet + Customer share |

## Key Borrowers (from sample data)

| Code | Name |
|------|------|
| BAML US | BOFA SECURITIES, INC. (Bank of America Merrill Lynch) |
| BCSL | Barclays Capital Securities Limited |

## What Nordnet Needs to Build (NNX Service)

Based on the file specs, the new retail NNX service needs to:

1. **Generate and send** Position File (daily) and Transaction File (hourly) via SFTP
2. **Generate and send** Client Reference File (daily) for opt-in/opt-out management
3. **Receive and process** Lent Positions, Collateral, Custody Loans reports (daily)
4. **Receive and process** Client Revenue and Billing Summary reports (monthly)
5. **Store** per-client lending data for app/web display and end-of-month reporting
6. **Reconcile** Custody Loans report against MT54X instructions from Citi
7. **Handle** opt-in/opt-out flow in customer-facing app/web
8. **Track** on-loan positions via "Flush & Fill" using daily Lent Positions report

## Related Documents

- **Workshop notes:** products/securities-lending/retail/csla-workshop-april-2026.md
- **Steering committee:** products/securities-lending/retail/csla-steering-committee-dec-2025.md
- **Overview:** products/securities-lending/overview.md
