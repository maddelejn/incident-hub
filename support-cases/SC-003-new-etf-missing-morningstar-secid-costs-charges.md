---
id: SC-003
title: "New ETF Missing Morningstar secId - Costs and Charges Skipped"
date: 2026-07-10
status: resolved
requester: "Internal (enrichment log error)"
requester_team: "Team Wolf"
service: "morningstar-fund-enrichment, instrument-admin"
team_responsible: "Team Wolf, Team Navigator"
product_owner: "Team Wolf (Madde)"
category: integration
frequency: occasional
faq_candidate: true
tags: [morningstar, secid, fundshareclassid, costs-and-charges, emt, enrichment, new-instrument, race-condition, bnp-paribas, etf, secondary-identifiers]
---

# SC-003: New ETF Missing Morningstar secId - Costs and Charges Skipped

## Question / Problem

The `morningstar-fund-enrichment` service logged:

> "Skipping CostsAndCharges creation for isin LU3243907741 / instrumentId 86075d2d-8fe3-4552-9821-9e5e363aa86a due to missing secId or fundShareClassId"

The affected instrument is **BNP Paribas Easy MSCI ACWI UCITS ETF EUR Acc** (ISIN: `LU3243907741`), launched January 2026.

Customers would be unable to trade this ETF due to missing regulatory cost data.

## Answer / Solution

This is a **new instrument race condition**. The instrument was onboarded into the instrument master from the exchange listing, but Morningstar hadn't yet provided the granular security identifiers needed by the enrichment service.

### Root Cause (Confirmed via Investigation)

Under the Morningstar intake row in Instrument Admin, Morningstar had only provided:

- `F00001TCGP` as the high-level Intake Instrument ID
- `A41XQA` (WKN) in Secondary identifiers

**Missing:** No explicit `secId` (`0P...` format) or `fundShareClassId` mapped with source type `MORNINGSTAR_PRIMARY_SHARE_CLASS_ID` in the Secondary identifiers block.

### Proof by Comparison

Compared to a working fund (e.g., Swedbank Robur Access Asien A), the Secondary identifiers block should contain:

- An `0P...` ID (e.g., `0P00016L22`) - the Morningstar SecID/Performance ID
- An `F00...` ID explicitly labeled as `MORNINGSTAR_PRIMARY_SHARE_CLASS_ID`

The BNP ETF only had the WKN - Morningstar's feed was incomplete for this new instrument.

## Steps to Resolve

1. **Confirm the gap** in Instrument Admin:
   - Search for the ISIN in [Instrument Admin](https://instrument-admin.tools.prod.nntech.io/Instruments)
   - Open the instrument → check **Secondary identifiers** for missing `0P...` ID
   - Open the **MORNINGSTAR** intake row to see raw vendor data
2. **Contact Morningstar** to request full identifier mapping for the ISIN.
   - Use productsupport@morningstar.com for instrument issues
   - Include: ISIN, which product you're looking at, which API call is involved
   - Always open a **new ticket**
3. **Once Morningstar updates their feed**, the enrichment service will automatically pick up the identifiers and populate costs and charges.
4. Alternatively: if Morningstar already has the `0P...` ID on their portal but hasn't sent it in the feed, a **manual mapping refresh** or re-fetch may resolve it faster.

## Diagnostic Guide: Missing Morningstar Identifiers

```
[Enrichment service skips instrument]
       │
       ▼
Check Instrument Admin → Secondary identifiers
       │
       ├──► 0P... ID is present ──► Mapping job failed to promote it
       │     → Check enrichment service config/logs
       │
       └──► 0P... ID is missing ──► Check MORNINGSTAR intake row
              │
              ├──► secId present in intake but not in Secondary identifiers
              │     → Internal mapping bug, escalate to Team Navigator
              │
              └──► secId missing from intake entirely
                    → Morningstar hasn't provided it yet
                    → Contact vendor, or wait for feed update (new instruments)
```

## Notes

### Morningstar ID Patterns

| Pattern | Type | Used For |
|---------|------|----------|
| `F0000...` | Fund/Share Class ID | High-level fund umbrella identifier |
| `0P000...` | SecID / Performance ID | Required by enrichment service for costs & charges |
| `0C000...` | Management Company ID | Fund provider identification |

### Why This Happens with New Instruments

Classic race condition: the exchange listing gets onboarded into the instrument master before Morningstar finishes classifying the new instrument and assigning granular security identifiers. Common for ETFs launched within the last few weeks/months.

## Related Cases

- **INC-002** (Missing Morningstar ETF target_market Data and KID): Also a Morningstar data gap for ETF regulatory data, but caused by ISIN missing from RegXchange entirely. Different root cause, similar downstream symptom.
- **SC-002** (Missing Morningstar OnDemand EMT Data): Morningstar intake gap for mutual fund EMT data. Different pipeline but same vendor dependency.
