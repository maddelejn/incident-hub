---
type: product-knowledge
title: "ExCo Presentation - Securities Finance: Stock Lending in NBAB"
status: approved
date: 2026-06-24
audience: "Executive Committee"
last_updated: 2026-07-13
---

# ExCo Presentation: Stock Lending in NBAB (June 24, 2026)

## Executive Summary

Presentation to ExCo requesting support to move forward with Securities Lending in NBAB (Nordnet Bank AB) retail accounts. This extends the existing pension lending program (established 2018, 450m+ SEK paid to customers) to retail/bank accounts using CSLA (Citi + Sharegain) platform.

**Revenue opportunity: 58+ mSEK p.a.** (Nordnet share, after conservative haircuts)

**Ask to ExCo:**
1. Bring Sec Lending into Strategic Capture and Cycle 3 planning
2. Prepare for split of Team Shark (2 devs for Sec Financing, 4 devs for Corp Actions)
3. Prepare for workshops and background work with Citi, Sharegain, and Nordent Legal during the fall, ahead of implementation planning

## Why Now

- Pension lending established since 2018, over **450m SEK paid to customers** with matching revenue to Nordnet
- Lending in Bank was previously postponed due to **lack of scalable systems for individual accounts**
- **CSLA (Citi & Sharegain) have now developed a platform** that manages opt-in and lending operations for individual accounts
- Securities Lending is common in Europe; Nordnet has **first-mover opportunity in the Nordics**
- Development possible in **fall 2026** for **early 2027 launch**

## Retail vs Pension: Key Differences

| Aspect | Pension (Existing) | Retail (New - NBAB) |
|--------|-------------------|---------------------|
| **Participation** | Opt-out (default enrollment, maximizes volume) | **Opt-in** (manual, required via web interface for regulatory/ownership reasons) |
| **Position handling** | Aggregate | **Per-client EOD snapshots + intraday transaction updates** |
| **Compensation** | Calculated daily | **Monthly reporting and calculation cycle** (via Sharegain) |
| **Transparency** | Limited | **High** - monthly statements with specific loaned positions and daily collateral values |
| **Operational booking** | Aggregate model | **Individual account level** - track and report loan positions per client, dynamically re-allocate collateral |

## CSLA Platform: Responsibility Split

| Capability | Owner | Details |
|------------|-------|---------|
| **Opt-in Handling & Active Enrollment** | Nordnet | Retail customers must actively choose to participate; no default enrollment |
| **Availability Calculation** | Sharegain + Nordnet | Availability reported per account (anonymised). Regular delta changes sent to Sharegain on position change |
| **Compensation & Payouts** | Sharegain + Nordnet | Sharegain manages daily interest & monthly statements. Internal systems update for automated payouts. Send client billing files showing loans & revenue |
| **Loan Position & Collateral** | Sharegain | Daily storage on loan and collateral reports. Supports customer billing and mandatory reporting requirements |
| **Loan Bookings** | Sharegain + Nordnet | Bookings between Citi & Nordnet Bank in Abasec. End client booking on Abasec or ledger |
| **Client Info** | Sharegain + Nordnet | Show lent position and collateral to customer |
| **Other Needs** | Nordnet | Reconciliation, Whitelist, SFTP Integration |

## Development & Implementation

### Development Timeline: 4-6 months
- Two developers needed full-time to build lending capabilities
- Front-end development also needed
- Developers split from Team Shark

### Implementation Timeline with Citi: 4 months (parallel)
- Risk & Compliance review
- Legal signing
- Operational review
- Connection set-up
- Pilot trades

### Rollout Plan (18-24 months)

| Phase | Scope |
|-------|-------|
| **1** | Launch with **Nordic and US/CA shares** in **Swedish AF** (Aktie- och fondkonto) |
| **2** | Integrate pension solution into NNX (avoid fragmented on-prem + cloud environments) |
| **3** | Expand to **all European markets** (Bank, Pension & Liv) |
| **4** | Denmark (all markets after omnibus change) and Norway & Finland (excluding book-entry, home market shares) |

### Team Split: Shark → Two Teams

```
Current Team Shark (6 developers):
  ├── 2 developers → Securities Financing platform (new team)
  └── 4 developers → Corporate Actions platform (stay in Shark/new CA team)
```

## Revenue Model

### Look-back Analysis (1 year to March 2026)

| Step | Amount (mSEK) |
|------|--------------|
| Gross Revenue Potential | 662 |
| After dilution from current pension lending | 494 |
| Excluding direct CSD holdings (DK, FI, NO) | 272 |
| After 50% opt-in rate* | 136 |
| After Citi fee (15%) | 116 |
| **Customer Revenue (50% share)** | **58** |
| **Nordnet Revenue (50% share)** | **58** |

*Note: 50% opt-in is by VALUE, not by number of clients. Concentrated ownership means a much lower ratio of clients is needed to reach this level.*

### Revenue Estimates (2027-2031)

| Year | Gross Revenue | Citi Fee | Customer Share | **Nordnet Revenue** |
|------|--------------|----------|----------------|---------------------|
| 2027 | 136 mSEK | -20 | -58 | **58 mSEK** |
| 2028 | 146 mSEK | -22 | -62 | **62 mSEK** |
| 2029 | 156 mSEK | -23 | -66 | **66 mSEK** |
| 2030 | 167 mSEK | -25 | -71 | **71 mSEK** |
| 2031 | 178 mSEK | -27 | -76 | **76 mSEK** |

**5-year totals (2027-2031):**
- Gross revenue: 783 mSEK
- Nordnet revenue: **333 mSEK**
- Customer revenue: 333 mSEK
- Citi fees: -117 mSEK

## Competitive Landscape

### European Brokers with Retail Lending

| Broker | Market | Lending Markets | Customer Share | Payout |
|--------|--------|----------------|---------------|--------|
| flatexDEGIRO | Europe | Europe, North America | 50% | Monthly |
| Saxo Bank | Global | Europe, North America, Asia | 50% | Monthly |
| Interactive Brokers | Global | Europe, North America, Asia | 50% | Daily |
| Trade Republic | Europe | Europe, North America | Indirect | Fee Subsidies |
| Robinhood | US/UK | Europe, North America | 15% | Monthly |
| eTORO | Europe/UAE | Global | 50% | Monthly |

### Nordic Competitive Position

| Broker | Market | Status |
|--------|--------|--------|
| **Avanza** | Sweden | Sec lending on endowment accounts only; 60/40 split (client favor). **Not on retail brokerage accounts** |
| **Saxo** | Europe/UAE | Opt-in program for all retail clients; 50/50; available in DK, NL, CH, SG |
| **Montrose** | Sweden | **Not offered yet, but product in development** |
| **Nordea** | Europe | Institutional only |
| **Danske Bank** | Europe | Institutional only |
| **SEB** | Europe | Institutional only |

**Key insight:** No pan-Nordic platform has launched a retail lending offering yet. Montrose is actively working on one. Nordnet has a **first-mover advantage window** in the Nordics.

## Market Restrictions

| Market | Lendable? | Notes |
|--------|-----------|-------|
| Swedish AF | **Yes - Phase 1** | Launch market |
| US/CA shares | **Yes - Phase 1** | Already lending on pension |
| Nordic shares | **Yes - Phase 1** | |
| European markets | **Phase 3** | Later expansion |
| Denmark | **Phase 4** | Requires omnibus change |
| Norway & Finland | **Phase 4** | Excluding book-entry, home market shares |
| Direct CSD holdings (DK, FI, NO) | **Not initially** | Excluded from revenue model (272→136 reduction) |

## Related Documents

- **Technical integration:** products/securities-lending/retail/integration-overview.md
- **Workshop notes:** products/securities-lending/retail/csla-workshop-april-2026.md
- **Overview:** products/securities-lending/overview.md
