---
type: product-knowledge
title: "Securities Lending & Borrowing - Product Overview"
status: active
product_owner: "Madde"
current_team: "Team Shark (shared with Corporate Actions)"
planned_team_split: "Q3/Q4 2026 - Split into separate CA team and SecLending/Borrowing team"
last_updated: 2026-07-13
---

# Securities Lending & Borrowing

## Product Owner

Madde (PO for Instrument Intake, Corporate Actions, Securities Lending/Borrowing & SFTR)

## Current Team Structure

**Team Shark** currently owns both Corporate Actions and Securities Lending/Borrowing. Planned split in fall 2026:

```
Current (2026 H1):
┌─────────────────────────┐
│      Team Shark         │
│  - Corporate Actions    │
│  - Securities Lending   │
│  - Securities Borrowing │
└─────────────────────────┘

Planned (2026 H2):
┌─────────────────────┐  ┌──────────────────────────┐
│  Team [TBD]         │  │  Team [TBD]              │
│  Corporate Actions  │  │  Securities Lending      │
│                     │  │  Securities Borrowing    │
└─────────────────────┘  └──────────────────────────┘
```

## Two Tracks

### 1. Pension Securities Lending (Existing)
- **Status:** Live, in production
- **Platform:** On-prem (legacy)
- **Planned migration:** Move to NNX (Nordnet's cloud platform)
- **Scope:** Available for pension accounts only

### 2. Retail Securities Lending (New)
- **Status:** Planned / In development
- **Platform:** NNX (cloud-native, new service)
- **Scope:** New offering for retail customer accounts
- **Timeline:** Fall 2026 onwards

### Key Architectural Difference

```
Pension (existing):          Retail (new):
┌──────────────┐            ┌──────────────┐
│   On-prem    │  ──────►   │     NNX      │
│   Legacy     │  (migrate) │    Cloud     │
└──────────────┘            └──────────────┘
                            ┌──────────────┐
                            │     NNX      │
                            │  New service │
                            └──────────────┘
```

Both will eventually run on NNX, but the retail solution is being built cloud-native from scratch while the pension solution needs migration.

## Documentation

Documents will be added as they're collected:

- `pension/` - Existing pension securities lending documentation
- `retail/` - New retail securities lending documentation

## What is Securities Lending?

Securities lending is when Nordnet lends a customer's shares to a borrower (typically a bank or broker) for a fee. The borrower needs the shares for various reasons (short selling, settlement, hedging). The customer earns income on shares that would otherwise sit idle in their account.

### Basic Flow

```
Customer's shares (sitting in account)
         │
         ▼
Nordnet (as lending agent) ──── lends shares to ────► Borrower
         │                                              │
         │◄──── receives collateral ◄──────────────────┘
         │◄──── receives lending fee ◄─────────────────┘
         │
         ▼
Customer receives share of lending fee (income)
```

### Key Concepts

- **Lender:** The customer whose shares are lent out
- **Borrower:** The counterparty receiving the shares (banks, brokers, hedge funds)
- **Collateral:** Security provided by borrower to protect against default (typically >100% of share value)
- **Lending fee:** Income generated, split between customer and Nordnet
- **Recall:** The ability to get shares back (e.g., if customer wants to sell)
- **SFTR:** Securities Financing Transactions Regulation - EU reporting requirement for all securities lending transactions
