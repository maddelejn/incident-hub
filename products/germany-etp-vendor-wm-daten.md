---
type: product-knowledge
title: "Germany ETP Expansion - WM Daten Vendor for FTT, EMT & KID Data"
status: in-progress
stakeholders: "Joachim Wegebrand (Head of ETP), Edward Neptune, Madde (PO Instrument Intake)"
vendor: "WM Daten (WM Gruppe)"
vendor_contact:
  sales: "Funda Üstün (F.Uestuen@wmdaten.com, +49 171 6431894)"
  technical: "Andreas Knabe (A.Knabe@wmdaten.com)"
last_updated: 2026-07-10
---

# Germany ETP Expansion: WM Daten as Vendor for FTT, EMT & KID Data

## Context

Nordnet is entering the German market and needs a vendor for:
1. **FTT data** (Financial Transaction Tax) for ETPs - needed by internal team Match for cost and charges
2. **EMT and KID data** for ETPs (ETNs/ETCs) listed on Xetra - regulatory requirement for order placement

The chosen vendor is **WM Daten** (Wertpapier-Mitteilungen), Germany's official national numbering agency.

## Who is WM Daten?

WM Daten is the **official national numbering agency for Germany**. Every financial instrument traded, settled, or taxed in Germany must be registered with them. They've been running the German financial master-data universe for decades.

### Why they're hard to work with

Their system is a massive legacy monolith. They don't organize data into clean modern templates like an "EMT file." Instead, they have **thousands of proprietary raw data fields** (often called "WM Field Codes" like GD210, GV340, etc.).

When Joachim asked "don't you identify Target Market and Costs by the EMT?", WM's response was essentially: "We just dump all the raw data fields on you; it's your job to map our fields to your EMT system."

**Key insight:** WM Daten speaks legacy German database architecture. Nordnet speaks modern EU regulation (EMT, KID). A translation/mapping layer is needed between the two.

## The Critical Legal Distinction: Funds vs. Structured Products

"ETP" (Exchange Traded Product) is a marketing umbrella term, but **legally they are completely different animals**:

| Product | Legal Classification | WM Daten Cost Category | Regulatory Framework |
|---------|---------------------|----------------------|---------------------|
| **ETFs** (Exchange Traded Funds) | Funds (UCITS collective investment schemes) | "Costs — Funds" | UCITS/PRIIPs |
| **ETCs** (Exchange Traded Commodities) | Structured Products (debt securities/certificates) | "Costs — Structured Products" | PRIIPs |
| **ETNs** (Exchange Traded Notes) | Structured Products (debt securities issued by a bank) | "Costs — Structured Products" | PRIIPs |

**This is why WM Daten splits costs into two buckets.** Germany treats a Swedbank ETF differently than a Xetra Gold ETC.

**Critical risk:** If the intake system only listens to "Fund" fields, it will completely miss data for ETNs and ETCs. The ingestion must handle both buckets.

## What is FTT Data and Why Does Team Match Need It?

**FTT = Financial Transaction Tax**

While Germany doesn't currently have its own FTT like France or Italy, German brokers must strictly enforce transaction taxes when clients buy foreign instruments that qualify (e.g., a French or Italian stock, or an ETP wrapping an asset bound by FTT rules).

Germany also has complex domestic tax laws (**Abgeltungsteuer** / Investment Tax Act). Team Match needs WM Daten's tax flags to determine:

- Is this ETP tax-exempt, or does it trigger German withholding tax?
- Is it classified as a "Certificate" or a "Fund Share" for tax calculations?
- How much tax to deduct from a client's account when they sell their ETP?

**If the platform doesn't ingest these tax flags**, the clearing/settlement system won't correctly calculate tax obligations for German-listed products.

## Current Status

### What's been done
- Initial discussions with WM Daten (Andreas Knabe, then Funda Üstün)
- Joachim asked for a filtered list of fields related only to EMT and KID for ETPs (ETNs/ETCs) on Xetra
- **Funda delivered an Excel sheet** with only the relevant fields, confirmed 100% coverage
- Everyone except Madde is on vacation; sync planned for August

### What's outstanding
- Review Funda's Excel sheet as a team
- Map WM Field Codes to Nordnet's internal EMT/Target Market/Cost database fields
- Build translation/mapping layer in ingestion service
- Hand over FTT/tax fields to Team Match
- Commercial negotiation (price differs if buying only a subset of fields vs. full data product)

## Game Plan for August

```
[Funda's Excel Sheet]
         │
         ▼
[Step 1: Product Type Filter]
Filter to fields for ETNs/ETCs and ETFs on Xetra only.
Discard any mutual-fund-only fields.
         │
         ▼
[Step 2: Legal Type Mapping]
Group fields by:
- "Costs — Structured Products" (ETCs/ETNs)
- "Costs — Funds" (ETFs)
- Target Market fields
- KID document fields
- FTT/Tax fields
         │
         ▼
[Step 3: Internal Mapping]
Map WM Field Codes to Nordnet internal DB fields.
Example: "WM Field X with Value Y" → Target_Market_Retail = Allowed
         │
         ▼
[Step 4: Team Match Handover]
Give FTT/tax-specific fields to Match team
for cost and charges calculation rules.
         │
         ▼
[Step 5: Build Ingestion]
Implement the translation layer in the intake service.
Must handle BOTH fund and structured product cost buckets.
```

## Vendor Contact

| Person | Role | Email | Phone |
|--------|------|-------|-------|
| Funda Üstün | Senior Sales Specialist | F.Uestuen@wmdaten.com | +49 171 6431894 |
| Andreas Knabe | Sales/Technical | A.Knabe@wmdaten.com | - |

**Company:** WM Gruppe, Sandweg 94, 60316 Frankfurt a. M.
**Website:** www.wmdatenservice.com
**LEI:** 5299000J2N45DDNE4Y28

## How This Connects to Existing Knowledge

### Relationship to EMT/KID guide (reference/emt-kid-priip-target-market-guide.md)
WM Daten provides the **same data concepts** covered in the EMT/KID guide (costs & charges, target market, KID documents), but in their own proprietary format rather than the standard FinDatEx EMT file format. The mapping layer essentially translates WM's field codes into the standard EMT fields the platform already understands.

### Relationship to existing Morningstar pipeline
Morningstar currently provides EMT/KID data for ETFs and mutual funds. WM Daten fills the gap for **German-specific ETPs (ETCs/ETNs) on Xetra** and provides the FTT/tax data that Morningstar doesn't cover. These are complementary, not competing, data sources.

### Relationship to INC-002 and INC-003
Those incidents involved missing regulatory data (KID/EMT/target_market) blocking ETF trading. The same risk exists for the German ETP expansion - if the WM Daten intake isn't correctly mapping both fund and structured product cost buckets, ETCs/ETNs on Xetra will be blocked for trading.
