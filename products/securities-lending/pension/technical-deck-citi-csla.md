---
type: product-knowledge
title: "Nordnet Technical Deck - CSLA Pension Integration Architecture"
status: active
source: "Nordnet Technical Deck - Final.pdf"
last_updated: 2026-07-13
---

# Nordnet Technical Deck - CSLA Pension Integration

## Current Architecture: Pension Lending via CSLA

### System Landscape

```
                    NORDNET
┌─────────────────────────────────────────────┐
│                                             │
│  ON-PREM                    NNX (CLOUD)     │
│  ┌──────────────────┐      ┌──────────────┐│
│  │ service-          │      │ Lending      ││
│  │ securities-       │─────►│ Compensation ││
│  │ lending           │ Data │              ││
│  │                   │Bucket│ (Payout to   ││
│  │ - Availability    │      │  customers)  ││
│  │ - Loan booking    │      └──────────────┘│
│  │ - Compensation    │                      │
│  │ - Reconciliation  │                      │
│  │ - Whitelist       │                      │
│  └────────┬─────────┘                      │
│           │ SFTP                            │
└───────────┼─────────────────────────────────┘
            │
     ┌──────┴──────┐
     │  SHAREGAIN  │ (CSLA Platform)
     │  - Loan     │
     │    matching │
     │  - Borrower │
     │    mgmt     │
     └──────┬──────┘
            │ MT54X (SWIFT)
     ┌──────┴──────┐
     │    CITI     │ (Custodian)
     │  - Custody  │
     │  - Settle-  │
     │    ment     │
     └─────────────┘
```

### Daily Task Flow

```
NIGHT (23:30-00:15)
  ├── CalculateAndUploadEndOfDayAvailabilityTask → Sharegain + Citi SFTP
  └── OnLoanUploadTask → Citi SFTP

MORNING (07:00-09:00)
  ├── LoanDownloadTask → Download from Sharegain SFTP (every 15 min)
  ├── OpenLoanDownloadTask → Download Citi's view (every 30 min)
  ├── CollateralDownloadTask → Download from Citi (07:30-08:30)
  ├── ReconciliationTask → Compare Nordnet vs Citi view (08:45)
  ├── DailyCompensationTask → Calculate customer compensation (08:45)
  ├── CollateralReconciliationTask → Verify collateral (08:45)
  └── BackOfficeReportTask → Report for Corporate Actions (09:00)

DAYTIME (08:10-21:30)
  ├── BookLoanTask → Book into Abasec (every 20 min)
  └── CalculateAndUploadIntradayAvailabilityTask → Sharegain only (every 60 min)
```

### Markets in Scope (Pension)

Currently lending on pension accounts covers:
- Nordic markets (SE, NO, DK, FI)
- US and Canadian markets

### Settlement Flow

```
Sharegain matches borrower demand with Nordnet availability
    │
    ▼
Loan instruction sent to Nordnet via SFTP
    │
    ▼
Nordnet books loan in Abasec (via service-abatron)
    │
    ▼
MT54X settlement instruction to Citi
    │
    ▼
Citi settles at custody level (T+2)
```

### Key Stakeholders

| Person | Role |
|--------|------|
| Kristofer Burén | Business stakeholder - monitors compensation, catches Citi data errors |
| Jonathan | Payments - books customer payouts |
| Madelene Söderström | Product Owner |
| Hampus Holmertz | Lead Developer |

### Compensation Flow

```
Daily:
  Open loan file from Citi → Calculate 85% of loan values →
  50% allocated to customers → Daily compensation stored

Monthly:
  Citi pays Nordnet for previous month

Quarterly:
  Nordnet pays customers (first month of quarter,
  sum of previous quarter's daily compensation)
  Via: Lending Compensation (NNX) → Deposit-Withdrawal Domain
```
