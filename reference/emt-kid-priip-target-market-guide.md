---
type: reference
title: "EMT, KID, PRIIP & Target Market - Complete Knowledge Guide"
category: regulatory-data
last_updated: 2026-07-10
---

# EMT, KID, PRIIP & Target Market Guide

A comprehensive guide for Product Owners working with Instrument Intake and regulatory data pipelines.

---

## Day 1: The Core Mission (The "Why" and the Big Picture)

### The Law: PRIIPs

**PRIIPs** = Packaged Retail and Insurance-based Investment Products.

**The problem:** Historically, disclosure documents for funds, structured notes, and insurance products were 80 pages of dense legal jargon. No one read them.

**The solution:** The EU mandated that manufacturers must create a standardized, easy-to-read, maximum 3-page document for retail investors.

### The Document: KID

**KID** = Key Information Document. It is a legal requirement. If a retail investor in the EU can buy an instrument, a KID must exist before they can trade it.

### The Data: EMT

The KID is a PDF for humans. Computers cannot easily read PDFs.

**EMT** = European MiFID Template. It's a standardized Excel/CSV file containing all the raw numbers and flags that match the KID. This is what gets ingested by systems.

### The Golden Rule

> **The KID is the PDF for the end customer. The EMT is the data file your system ingests to power your platform's rules, filters, and disclosures.**

---

## Day 2: The PRIIP KID (Deconstructing the PDF)

Every KID has the exact same sections. Your intake system needs to understand these concepts:

| Section | Content |
|---------|---------|
| **Product Identity** | ISIN, name, manufacturer |
| **SRI (Summary Risk Indicator)** | Score from 1 (lowest risk) to 7 (highest risk). Displayed in UI. |
| **Performance Scenarios** | Estimates for Favourable, Moderate, Unfavourable, and Stress conditions |
| **Credit Risk** | What happens if the manufacturer defaults |
| **Costs Over Time** | How much the investment costs if held for 1, 3, or 5 years |

### KIID vs KID (The "Double I")

- **KIID** (Key Investor Information Document): Legacy format specific to **UCITS funds** (traditional mutual funds and many European ETFs)
- **KID** (Key Information Document): The **PRIIPs format** (for structured products, derivatives, and newer funds)

The market has been transitioning UCITS ETFs to the PRIIPs KID format, but many legacy systems still refer to the document layer as "KIID."

---

## Day 3: Target Market Data (Who is allowed to buy this?)

Under MiFID II, manufacturers must state exactly who their product is meant for. This is the **Target Market Assessment**. Your intake system must capture this so the trading platform can block or warn users.

### 5 Core Lenses of Target Market Data

| Lens | Question |
|------|----------|
| **Investor Type** | Retail, Professional, or Eligible Counterparty? |
| **Knowledge & Experience** | Does the buyer need to be an expert, or can a novice buy it? |
| **Ability to Bear Losses** | Can they lose all their money, or do they need capital preservation? |
| **Risk Tolerance** | Does it match their risk profile? |
| **Client Objectives & Needs** | Is it for ESG investing, retirement, or short-term speculation? |

---

## Day 4: Costs & Charges (The Hidden Numbers)

Regulators want total transparency on fees. The EMT breaks costs into granular buckets so distributors can show the "Total Cost of Investing" **before** they trade (Ex-ante) and **after** (Ex-post).

### Four Main Cost Buckets

| Bucket | Description | Example |
|--------|-------------|---------|
| **One-off Costs** | Entry/exit fees | 2% front-end load fee to buy a fund |
| **Ongoing Costs** | Management fees taken out yearly | Annual management fee |
| **Transaction Costs** | Costs incurred by the fund buying/selling underlying assets | Broker commissions |
| **Incidental Costs** | Performance fees | "We take 20% of profits above a benchmark" |

---

## Day 5: The EMT Data File (The Technical Reality)

The EMT is managed by **FinDatEx** (a joint structure of European financial associations).

Key characteristics:

- Massive flat file (CSV/Excel) with **over 100 standardized data fields**
- Fields are strictly coded: `Y` (Yes), `N` (No), `Neutral` (not Y/N)
- **Version control matters:** The industry uses EMT V4.1 (or later). Your ingestion parser must match the exact version the market is sending.

---

## Day 6: System Integration (The Intake Pipeline)

```
[Data Vendors: Morningstar, SIX, WM]
               │
               ▼ (Daily EMT Delta Files)
   ┌───────────────────────┐
   │ Your Intake Ingestor  │ ──► [Validation Engine]
   └───────────────────────┘     (Does it have an SRI? Is Cost numeric?)
               │
               ▼ (Parsed Data)
┌─────────────────────────────┐
│    Instrument Master DB     │
└─────────────────────────────┘
   │                         │
   ▼                         ▼
[Compliance/Blocking Engine] [Frontend / UI Displays]
(Block Retail if              (Show 3-page KID PDF
 Target Market = Prof Only)    and SRI = 5)
```

---

## Day 7: Edge Cases & Expertise

### Three Classic Intake Traps

1. **The Missing KID Dilemma:** Vendor sends data for a new instrument, but the manufacturer hasn't generated the KID yet. Rule: If `Retail Type = Allowed` but `KID URL = Null` → Block onboarding.

2. **Execution-Only vs. Advisory:** If the platform is "Execution-Only" (users just click buy), Target Market validation rules are less strict than with robo-advisors.

3. **The UK vs EU Split:** Since Brexit, the UK has its own template versions (EMT v4.1 UK, consumer duty templates). The intake system must differentiate between instruments sold to EU vs UK retail investors.

---

## ETF_KIID_ERROR vs EMT_DATA_ERROR

These represent **two entirely different data supply chains**.

### The Short Answer

- **EMT_DATA_ERROR** = The structural, machine-readable data file (spreadsheet/feed) failed ingestion rules
- **ETF_KIID_ERROR** = The legal, human-readable PDF document is missing, corrupt, expired, or in the wrong language/framework

### Detailed Comparison

| Feature | EMT Data (EMT_DATA_ERROR) | KIID/KID Document (ETF_KIID_ERROR) |
|---------|--------------------------|-------------------------------------|
| **What is it?** | A row in a CSV/XML file with numbers and codes | A physical PDF document |
| **Who uses it?** | Your trading system (automated blocking and compliance) | Your end-customer (reads before buying) |
| **Example error cause** | Management Fee field contains text instead of number, or Target Market flag is blank | PDF URL is broken (404), or PDF is in German but user is in France |

### Two Separate Gates in the Intake Workflow

```
[Incoming ETF]
       │
       ▼
 [Gate 1: Ingest EMT Data] ──(Fail)──► EMT_DATA_ERROR (Block Intake)
       │ (Pass)
       ▼
 [Gate 2: Fetch & Verify PDF] ──(Fail)──► ETF_KIID_ERROR (Block Retail Trading)
       │ (Pass)
       ▼
[Instrument Live on Platform]
```

### EMT_DATA_ERROR Typical Triggers

- Vendor sent file with missing mandatory fields (e.g., missing SRI score)
- Vendor updated file format to new FinDatEx version but intake parser wasn't updated
- Data type mismatch (Y/N flag contains NULL or True)

### ETF_KIID_ERROR Typical Triggers

- EMT data gave a URL to the KID PDF but the link was dead (404)
- ETF is cross-listed and vendor provided UK version of KIID instead of EU EEA version
- Document has expired or is missing an updated translation for the target country

---

## Morningstar Identifier System

Understanding Morningstar's internal ID structure is critical for troubleshooting intake issues.

### ID Types

| ID Pattern | Type | Purpose |
|------------|------|---------|
| `F0000...` | Fund/Share Class ID | High-level identifier for the fund umbrella or share class |
| `0P000...` | SecID / Performance ID | Granular security-level identifier for a specific traded line |
| `0C000...` | Management Company ID | Identifies the fund provider/manufacturer |

### How They Map in the Nordnet System

In Instrument Admin, a correctly mapped instrument should have in its **Secondary identifiers** block:

- An `0P...` ID (SecID) - used by the enrichment service to pull EMT costs and charges
- An `F00...` ID mapped with source type `MORNINGSTAR_PRIMARY_SHARE_CLASS_ID`

### The New Instrument Race Condition

When an ETF is brand new:

1. The exchange/listing team onboards the raw ISIN/Ticker into the instrument master
2. Morningstar may not have finished classifying it yet
3. The daily mapping sync job can't fetch Morningstar's newly assigned `secId`
4. The `morningstar-fund-enrichment` service skips the instrument because it can't find the required identifiers
5. Result: Missing costs and charges, target market data, or KID mappings

**Diagnostic steps:**

1. Check **Secondary identifiers** in Instrument Admin - is the `0P...` ID present?
2. Check the **MORNINGSTAR intake row** - what raw data did Morningstar actually send?
3. If the `0P...` ID is missing from Secondary identifiers but present in the intake row → mapping job failed to promote it
4. If the `0P...` ID is missing from the intake row entirely → Morningstar hasn't provided it yet, contact vendor

---

## Related Incidents and Cases

| Case | Issue | Error Type |
|------|-------|------------|
| INC-002 | Missing ETF target_market data - ISIN absent from RegXchange | ETF_KIID_ERROR / EMT_DATA_ERROR |
| INC-003 | Dual-listed ETP - pipeline mapped data only to Xetra version | EMT_DATA_ERROR |
| SC-002 | Missing Morningstar OnDemand EMT data for mutual funds | EMT_DATA_ERROR |
| SC-003 | New ETF missing Morningstar secId - enrichment skipped | EMT_DATA_ERROR (costs & charges) |
