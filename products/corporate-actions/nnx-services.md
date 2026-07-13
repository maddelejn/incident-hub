---
type: product-knowledge
title: "Corporate Actions - NNX Cloud Services"
status: active
owning_team: "Team Shark"
product_owner: "Madelene Söderström"
platform: "NNX (cloud)"
last_updated: 2026-07-13
---

# Corporate Actions - NNX Cloud Services

These services have been migrated from on-prem to NNX as part of the incremental cloud migration. They extend the on-prem `service-corporate-action` with new cloud-native capabilities.

---

## 1. Presubscription (ca-presubscription)

**Source:** [Confluence](https://nordnetbank.atlassian.net/wiki/spaces/DE/pages/579371064)
**GitHub:** https://github.com/nordnet-private/ca-presubscription

### Purpose

Manages presubscriptions to corporate actions. When issuers require customers to have specific presubscription types (e.g., employees getting priority allocation in an IPO), this service handles the assignment.

### Presubscription Types

| Type | Swedish | Use Case |
|------|---------|----------|
| PRESUBSCRIBEE | Teckningsåtagare | Committed subscribers |
| CORNERSTONE_INVESTOR | Garant | Guaranteed investors |
| FRIENDS_AND_FAMILY | - | Issuer's network |
| EMPLOYEE | - | Company employees |

### How It Works

1. Service fetches active corporate actions from **Event-Response-Handler (ERH)**
2. Operator adds a presubscription type for a specific account via admin
3. An event is generated and sent to the on-prem `corporate-action` service
4. Presubscription type is added to the row in the generated IPO file

### Integration with Hidden Events

When on-prem `corporate-action` creates a hidden event with automatic presubscription:
- On-prem sends a PubSub command to `ca-presubscription`
- `ca-presubscription` creates the presubscription
- `ca-presubscription` sends a PubSub event back confirming creation
- On-prem `corporate-action` receives and stores the confirmation

### Caveat

No feedback from the on-prem `corporate-action` service - if the presubscription type failed to be added to the CA file or the customer never exercised it, this service won't know.

### Dataland

| Data | Classification | Lifecycle |
|------|---------------|-----------|
| Presubscription type and account | Confidential | Experimental/production |

---

## 2. Manual Answer (ca-manual-answer)

**Source:** [Confluence](https://nordnetbank.atlassian.net/wiki/spaces/DE/pages/954499147)
**GitHub:** https://github.com/nordnet-private/ca-manual-answer

### Purpose

Enables authorized users to create, view, and delete manual answers for corporate actions that require manual intervention (e.g., customer calls in to respond).

### Key Design Decisions

- **One answer per account-id + corporate-action-id** (unique constraint)
- **Append-only database** - deletions are marked as `DELETED`, not removed (audit trail)
- **Revision tracking** - create → delete → create results in 3 entries with revision numbers 1, 2, 3

### Integration

When an answer is created or deleted via admin, a REST call is made to **event-response-handler (ERH)** and incorporated into the normal answer flow.

### Authorization

| Group | Access |
|-------|--------|
| `AAD_Tools_Corporate_Action` | Top-level access |
| `AAD_Tools_Corporate_Action_Manual_Answers_View` | View permission |
| `AAD_Tools_Corporate_Action_Manual_Answers_Modify` | Modify permission |
| Role: Engineer | View access |
| Role: Customer Service | Modify access |

### Decision Maker

- Default: **real owner of the account**
- Exception: POA (Power of Attorney) calls → Customer Service contacts CA operations → manual Excel update

### Validation

All validation is done by **ERH** (event-response-handler). Customer Service has additional checks to mitigate any missing rules in ERH.

---

## 3. Reservation of Trading Power (ca-reserve-trading-power)

**Source:** [Confluence](https://nordnetbank.atlassian.net/wiki/spaces/DE/pages/1672216620)
**Epic:** [ASB-496: MVP1](https://nordnetbank.atlassian.net/browse/ASB-496)

### Purpose

Reserves and releases trading power on customer accounts for corporate actions. Without this, customers might spend cash needed for CA events, causing processing failures.

### Available Operations (Admin Interface)

| Operation | Description |
|-----------|-------------|
| Upload CSV file | Create booking group from allocation file |
| View booking groups | Browse all groups for a specific CA event |
| Reserve trading power | Trigger reservation, preventing customers from using allocated funds |
| Release trading power | Release reservations, making funds available again |
| Delete specific bookings | Remove individual bookings by account number |
| Delete booking group | Remove entire group with all bookings |
| Retry failed bookings | Manually retry bookings that failed due to temporary errors |

### CSV File Format

Semicolon-separated, required headers: `accno`, `price`, `shares`

```csv
accno;price;shares
123456;10.50;100
234567;11.25;150.5
345678;12.00;200
```

- Each account number must be unique
- All three columns required
- Supports both dot and comma as decimal separator

### Architecture

```
Operations User
    │
    ▼ (upload CSV)
Admin Frontend
    │
    ▼
ca-reserve-trading-power
    ├──► Cloud Storage (audit trail of uploaded files)
    ├──► Pre-Trades API (reserve/release trading power)
    ├──► Event Receiver Hub (corporate action event info)
    └──► PubSub (reliable async processing with auto retries)
```

### Norwegian Shareholders Disclosure

Endpoint returns all currently reserved positions for disclosure obligations:
- Fields: AccountNumber, ISIN, Number of shares
- Only includes positions with status `RESERVED`
- Required because Norway has obligations to disclose when X number of shares are being issued to Nordnet

---

## 4. German Tax Reporting (ca-sectras)

**Source:** [Confluence](https://nordnetbank.atlassian.net/wiki/spaces/DE/pages/2011070551)
**Admin:** https://corporate-action-admin.tools.prod.nntech.io/german-tax-reporting

### Purpose

Supports correct calculation of German customers' taxes. Third-party vendor **Sectras** needs corporate action data to calculate taxes. **Team Hodor** reports transaction data to Sectras, but needs Shark's CA data for enrichment.

### Architecture (3 services + admin)

```
Abasec (corporate action event data)
    │
    ▼
ca-poller (polls Abasec for CA events)
    │
    ▼
ca-bookkeeper (transforms to domain representation)
    │
    ▼
ca-sectras (persists and serves CA data)
    │
    ├──► Admin UI (operations manual handling)
    └──► Team Hodor / transaction-gateway (queries CA data for Sectras reporting)
```

### Integration with Hodor

#### Regular Transactions

Hodor queries ca-sectras with:
- verificationNumber + verificationNumberType → identifies the corporate action
- custodyInstrumentId + transactionTypeId → identifies the specific leg

Response includes: sectrasReportingType, sectrasEventReference, sectrasBidReference, sectrasReportableStatus, price, currency, exchange rates, tax verification numbers.

#### Tax Transactions

Hodor queries with just the verification number. Ca-sectras responds with the corporate action's verification numbers and the derived transaction type.

### Reporting Types

| Type | Interface | Description |
|------|-----------|-------------|
| **CEG** | Sectras CEG | Coupons, dividends |
| **TRDG** | Sectras TRDG | Splits, exchanges, removals, spin-offs, etc. |
| **OTHER** | Not reported | Not tax-relevant |

### Event Type Mapping

| Event Type | Reporting Type | Sub-class |
|-----------|---------------|-----------|
| COUPON, DIVIDEND | CEG | - |
| AMORTISATION, MATURITY_BOND, REMOVAL_* | TRDG | Buy/Sell |
| CHANGE, EXCHANGE, SPLIT, SPLIT_NEW_INSTRUMENT, EXCHANGE_OF_CURRENCY | TRDG | Reorganization |
| BONUS_ISSUE_DIRECT_TO_SHARE, DIVIDEND_OF_INSTRUMENT, DIVIDEND_REINVESTMENT, FLEXIBLE_EXCHANGE, SPIN_OFF, SUBSCRIPTION, etc. | TRDG | - |
| BONUS_ISSUE_LOOSENING_RIGHTS, EXCHANGE_OF_FORWARDS/OPTIONS, MISCELLANEOUS, REPURCHASE, RIGHTS_ISSUE, SPIN_OFF_FORWARDS/OPTIONS | OTHER | - |

### Reportable Status

| Status | Meaning |
|--------|---------|
| **CONFIRMED** | Ready to be reported to Sectras |
| **PENDING** | Requires manual intervention by operations in admin |
| **NON_REPORTABLE** | Should not be tax reported |

Default status rules:
- **CEG** → CONFIRMED (but DIVIDEND → PENDING when conflicting same ISIN + ex-date)
- **TRDG Buy/Sell** → CONFIRMED
- **TRDG Reorganization** → CONFIRMED or NON_REPORTABLE (depends on whether legs have same ISIN+quantity)
- **TRDG other** → PENDING
- **OTHER** → NON_REPORTABLE

### Manual Processes

PENDING events require operations to:
1. Link Nordnet's CA to corresponding Sectras CA (via BID reference from WM DATA, or event reference from admin)
2. Set status to CONFIRMED in admin

### Key Concepts

- **Legs:** Position legs (instrument movements) and Cash legs (money movements) derived from Abasec eventbookings
- **Verification Numbers:** From Abasec eventexecutions, type CASH or POSITION
- **BID Reference:** From WM DATA, used to distinguish same-day dividends on same ISIN
- **Currency Exchange Rates:** Base currency from eventbookings, quote currencies from eventcurrencyprices

### Tax Verification Number Mapping

| Event Type | Transaction Type |
|-----------|-----------------|
| REMOVAL_WITH_PAYMENT | GRU (Reimbursement out) |
| DIVIDEND_OF_INSTRUMENT | GDI (Dividend instrument in) |
| SPIN_OFF | GSPI (Detach spinoff in) |

Other event types under investigation.

---

## Migration Status Overview

```
ON-PREM (service-corporate-action)     NNX (Cloud)
┌────────────────────────────────┐     ┌────────────────────────────────┐
│ CA Admin (event creation)      │     │ ca-presubscription            │ ✅
│ Customer responses (Next)      │     │ ca-manual-answer              │ ✅
│ Abasec integration (AIS/WCF)  │     │ ca-reserve-trading-power      │ ✅
│ Compile file generation       │     │ ca-poller                     │ ✅
│ TRS reporting                 │     │ ca-bookkeeper                 │ ✅
│ Emission settings             │     │ ca-sectras (German tax)       │ ✅
│ Hidden events                 │     │ lending-compensation          │ ✅
│ Danish 20% rule               │     │                               │
└────────────────────────────────┘     └────────────────────────────────┘
         PubSub sync ◄──────────────────────►
```

## Related Documents

- **On-prem service:** products/corporate-actions/service-corporate-action.md
- **Securities lending (shares on-prem):** products/securities-lending/overview.md
