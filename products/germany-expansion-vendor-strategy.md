---
type: product-knowledge
title: "Germany Expansion - Consolidated Vendor Strategy for FTT, EMT, KID & ETP Data"
status: in-progress
leads: "Madde (PO Instrument Intake), Pierre"
stakeholders: "Joachim Wegebrand (Head of ETP), Edward Neptune, Team Match (Costs & Charges)"
last_updated: 2026-07-10
---

# Germany Expansion: Consolidated Vendor Strategy

## Ownership

**Madde and Pierre** are the single point of contact for all vendor discussions related to the Germany expansion data needs. All requirements flow through them to avoid duplicate negotiations and overlapping vendor contracts.

## The Three Data Needs (One Vendor Decision)

The Germany expansion requires data for three distinct but overlapping purposes:

```
                    ┌─────────────────────┐
                    │  VENDOR SELECTION    │
                    │  (Madde + Pierre)    │
                    └──────┬──────────────┘
                           │
            ┌──────────────┼──────────────┐
            ▼              ▼              ▼
    ┌──────────────┐ ┌───────────┐ ┌──────────────┐
    │ 1. FTT/Tax   │ │ 2. ETP    │ │ 3. Costs &   │
    │    Data      │ │ Regulatory│ │    Charges    │
    │              │ │ Data      │ │              │
    │ Need: Enter  │ │ Need: KID │ │ Need: Ex-ante│
    │ German mkt   │ │ EMT, TM   │ │ disclosure   │
    │              │ │ for ETPs  │ │              │
    │ Consumer:    │ │ Consumer: │ │ Consumer:    │
    │ Tax/Clearing │ │ Instrument│ │ Team Match   │
    │              │ │ Intake    │ │              │
    └──────────────┘ └───────────┘ └──────────────┘
```

### 1. FTT / Tax Data
- **Why:** Required to enter the German market. German tax laws (Abgeltungsteuer / Investment Tax Act) require correct tax classification of every instrument.
- **Who needs it:** Tax/Clearing systems, also Team Match for cost calculations
- **Key data:** Tax classification flags, withholding tax rates, fund vs certificate classification, FTT applicability for foreign instruments

### 2. ETP Regulatory Data (EMT, KID, Target Market)
- **Why:** Cannot offer ETPs (ETNs/ETCs) on Xetra to retail customers without KID documents, EMT target market data, and cost disclosures
- **Who needs it:** Instrument Intake pipeline, compliance/blocking engine, customer-facing UI
- **Key data:** KID PDFs, target market flags, SRI scores, eligible investor types

### 3. Costs & Charges
- **Why:** MiFID II requires ex-ante cost disclosure before every trade. Customers must see total cost of investing.
- **Who needs it:** Team Match (owns the cost calculation and display)
- **Key data:** Entry/exit costs, ongoing costs, transaction costs, performance fees

### The overlap

All three needs pull from the same underlying vendor data. A single vendor contract covering EMT + KID + tax flags serves all three consumers. This is why centralizing through Madde and Pierre prevents:
- Instrument Intake buying EMT from vendor A
- Team Match buying costs from vendor B
- Tax team buying FTT from vendor C
- ...when one vendor could serve all three

## Vendors Under Evaluation

| Vendor | Coverage | FTT/Tax | EMT/KID | Costs | Format | Status |
|--------|----------|---------|---------|-------|--------|--------|
| **WM Daten** | 100% (confirmed) | Yes (German NNA) | Yes | Yes | Proprietary field codes (mapping needed) | Excel received from Funda |
| **SIX** | 100% TM+Costs, 379/384 KIDs | TBD | Yes | Yes | TBD | Coverage check done |

### Decision criteria for August
1. **FTT/Tax coverage** - Does SIX cover German tax flags as well as WM Daten (Germany's NNA)?
2. **Data format** - Does SIX deliver standard EMT format? (Would save significant mapping effort vs WM Daten)
3. **Price** - Full product vs subset pricing
4. **Integration effort** - Existing SIX relationship vs new WM Daten onboarding
5. **Multi-market coverage** - SIX covers beyond Xetra; does WM Daten?
6. **Field-level comparison** - Do both files contain the same actual data points?

## Detailed Vendor Docs

- **WM Daten details:** products/germany-etp-vendor-wm-daten.md
- **SIX details:** products/germany-etp-vendor-six.md

## Timeline

| Date | Event |
|------|-------|
| 2026-07-02 | Madde contacted SIX (Carl Sundman) |
| 2026-07-02 | Sent instrument universe to SIX for coverage check |
| 2026-07-03 | SIX returned coverage results (384 ISINs, 5 missing KIDs) |
| 2026-07-03 | WM Daten initial discussions (Joachim → Andreas Knabe) |
| 2026-07-09 | Funda (WM Daten) delivered filtered Excel with ETP-only fields |
| 2026-07-09 | Both vendor discussions paused for summer vacation |
| August 2026 | Team sync and vendor comparison. Meeting with SIX booked. |
| August 2026 | Vendor decision and contract negotiation |
