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

## Coverage Check Results (July 2026)

Both vendors have only delivered **coverage checks** so far - confirming they have the data, not delivering actual field values or field definitions. The real evaluation requires sample data files from both.

### Side-by-side coverage comparison

| Data Point | SIX | WM Daten |
|------------|-----|----------|
| **Instruments checked** | 384 | 383 |
| **Target Market** | 384/384 **(100%)** | 379/383 (99%) - 4 missing |
| **Costs & Charges** | 384/384 **(100%)** | 382/383 (99.7%) - 1 missing |
| **KIDs** | 379/384 (5 missing) | 379/383 (4 missing) |
| **FTT/Tax data** | **Not confirmed** | **Not in coverage file** |

### Overlapping gaps (same ISINs missing from both vendors)

These ISINs are missing KIDs at both vendors - likely very new or obscure structured products:
- `XS3269544030`
- `XS3373414203`
- `XS3373438483`

### WM Daten additional gaps (not missing at SIX)

| ISIN | Target Market | Costs | KID |
|------|--------------|-------|-----|
| `XS2595366340` | Missing | Missing | Missing |
| `XS3269544030` | Missing | Available | Missing |
| `XS3373414203` | Missing | Available | Missing |
| `XS3373438483` | Missing | Available | Missing |

### WM Daten field codes in coverage file

| WM Field | Values | Meaning |
|----------|--------|---------|
| `gd100a` | All `1` | Instrument matched/active in WM system |
| `gd496b` | `1` = available, `2` = not available | KID availability |
| `gd496c` | `J` = Ja, empty = no | Target market data available |
| `gd496d` | `J` = Ja, empty = no | Costs & charges data available |
| `gd198b` | All `4000` | Instrument classification code |

### ISIN country distribution (383 instruments)

| Country | Count | Examples |
|---------|-------|---------|
| DE (Germany) | 123 | German-issued ETPs |
| XS (International) | 85 | Cross-border structured products |
| GB (United Kingdom) | 62 | UK-issued ETPs |
| CH (Switzerland) | 47 | Swiss-issued ETPs |
| JE (Jersey) | 43 | Jersey-issued ETPs |
| SE (Sweden) | 12 | Swedish-issued ETPs |
| IE (Ireland) | 10 | Irish-issued ETPs |
| FR (France) | 1 | French-issued ETP |

## What We Still Need for the August Decision

Neither vendor has provided enough to make a decision yet. Here's what to request:

### From both vendors
1. **Sample data file** - actual field values for 10-20 instruments, not just "yes/no" coverage
2. **Field definitions list** - what specific EMT/target market/cost fields are included?
3. **FTT/Tax data** - do they provide German tax classification flags (Abgeltungsteuer)? This is critical for Team Match and is the make-or-break question
4. **Data format** - standard EMT CSV? Proprietary? API?
5. **Delivery mechanism** - SFTP, API, portal?
6. **Update frequency** - daily, real-time, on-change?
7. **Pricing** - full product vs subset, per-instrument or flat fee?

### From SIX specifically
- Confirm FTT/tax data availability (existing Nordnet relationship makes this easiest to ask)
- What format do they deliver in? If standard EMT, that's a major integration advantage

### From WM Daten specifically
- Get the actual field-level list Funda referenced (the coverage file only had 5 yes/no fields)
- Andreas offered to show how to work with WM documentation - schedule that for August
- Clarify pricing: full data product vs the 10-15% of fields Joachim asked about

## Vendor Comparison Matrix

| Criteria | SIX | WM Daten | Winner |
|----------|-----|----------|--------|
| **Coverage (TM + Costs)** | 100% | 99% | SIX |
| **Coverage (KIDs)** | 98.7% | 98.9% | Tie |
| **FTT/Tax data** | Unknown - must ask | **Confirmed** (18 fields, FR/IT/ES) | WM Daten (unless SIX matches) |
| **Sanctions data (ISM)** | TBD | Available (92002/92003/92004 files) | WM Daten |
| **Data format** | TBD | Proprietary (mapping needed) | TBD |
| **Existing Nordnet vendor** | Yes | No | SIX |
| **Communication language** | Swedish | English/German | SIX |
| **Integration effort** | TBD | High (custom mapping) | TBD |
| **Multi-market coverage** | Yes (confirmed) | TBD | SIX |
| **Price** | TBD | TBD | TBD |

## Detailed Vendor Docs

- **WM Daten details:** products/germany-etp-vendor-wm-daten.md
- **SIX details:** products/germany-etp-vendor-six.md

## Timeline

| Date | Event |
|------|-------|
| 2026-07-02 | Madde contacted SIX (Carl Sundman) |
| 2026-07-02 | Sent instrument universe to SIX for coverage check |
| 2026-07-03 | SIX returned coverage results (384 ISINs, 5 missing KIDs) |
| 2026-02-20 | WM Daten meeting - Vadim Peters shared FTT field list (18 fields) + ISM sanctions data + delivery options |
| 2026-07-03 | WM Daten ETP discussions (Joachim → Andreas Knabe) |
| 2026-07-09 | Funda (WM Daten) delivered coverage check Excel (383 ISINs, 5 WM fields) |
| 2026-07-10 | Analysis: both files are coverage checks only, not actual data |
| 2026-07-10 | Both vendor discussions paused for summer vacation |
| August 2026 | Request sample data files and field definitions from both vendors |
| August 2026 | Team sync (Madde, Pierre, Joachim, Edward) |
| August 2026 | SIX meeting (booked with Carl) |
| August 2026 | Vendor decision and contract negotiation |
