---
id: SC-003
title: "Missing Fee and Cost Data on New ETF due to Incomplete Morningstar Feed"
date: 2026-07-10
status: pending
requester: "Christopher Barham (via #area-securities-brokerage)"
requester_team: ""
service: "morningstar-fund-enrichment, instrument-admin, etf-page"
team_responsible: "Team Navigator (Filippa Engstedt, Clara Montgomery), Team Wolf"
product_owner: "Securities Brokerage"
category: integration
frequency: occasional
faq_candidate: true
tags: [morningstar, secid, fundshareclassid, costs-and-charges, emt, enrichment, new-instrument, race-condition, bnp-paribas, etf, secondary-identifiers]
---

# SC-003: Missing Fee and Cost Data on New ETF due to Incomplete Morningstar Feed

## Question / Problem

Christopher Barham raised in **#area-securities-brokerage** at 09:35 that ETF `LU3243907741` ([BNP Paribas Easy MSCI ACWI UCITS ETF EUR Capitalisation](https://www.nordnet.se/etf/lista/bnp-paribas-easy-msci-eeab-xeta)) was missing "avgifts data" (cost/fee data). Urgency was marked as timely but not urgent.

Axel Karlsson (Team Mint) confirmed completely missing cost data and empty ex-ante (kostnadsredovisning). Filippa Engstedt (Team Navigator) found the `morningstar-fund-enrichment` log:

> "Skipping CostsAndCharges creation for isin LU3243907741 / instrumentId 86075d2d-8fe3-4552-9821-9e5e363aa86a due to missing secId or fundShareClassId"

The affected instrument is **BNP Paribas Easy MSCI ACWI UCITS ETF EUR Acc** (ISIN: `LU3243907741`), launched January 2026. Missing cost data blocks customers from trading.

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

## Timeline

| Time (CET) | Event |
|-------------|-------|
| 09:35 | Christopher Barham raised missing cost data in #area-securities-brokerage, tagged @wolf_goalie |
| 09:36 | Madde suggested it might be @navigators-goalie / @match-goalie |
| 09:38 | Axel Karlsson (Team Mint) confirmed missing cost data and empty ex-ante, pointed to Morningstar intake → Team Navigator |
| 10:06 | Filippa Engstedt (Navigator) confirmed data is being received from M*, began investigation |
| 10:25 | Filippa found enrichment log: "Skipping CostsAndCharges... due to missing secId or fundShareClassId" |
| 10:27 | Filippa asked @wolf_goalie to check instrument mapping |
| 10:27 | Madde: will look after solving #low-pressure issue first |
| 13:58 | Clara Montgomery (Navigator): team going on vacation today, would like to solve it today |
| 14:04 | Madde asked Lars Mattsson for help |
| 14:32 | Lars: doesn't know this flow, assumed it would work after re-creation |
| 15:21 | Clara: looking again |
| 15:24 | Filippa: reran costs and charges, got same error. Asked Lars to look together |
| 19:24 | Madde posted investigation findings: Morningstar only provided F00001TCGP and WKN (A41XQA), missing 0P... secId. Compared to working fund. Suggested contacting M*. |

## Stakeholders

| Role | Person/Team | Action |
|------|-------------|--------|
| Requester | Christopher Barham | Raised missing cost data |
| Investigation | Filippa Engstedt (Team Navigator) | Found enrichment log error, confirmed M* data received |
| Investigation | Axel Karlsson (Team Mint) | Confirmed empty cost data and ex-ante |
| Root cause identification | Madde (Team Wolf) | Identified missing secId in Secondary identifiers |
| Attempted fix | Clara Montgomery (Team Navigator) | Reran enrichment, same error |
| Consulted | Lars Mattsson (Team Wolf) | Unfamiliar with this flow |

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

### Standard Operating Procedure for This Pattern

Newly launched or added ETFs/funds from Morningstar may occasionally ingest with incomplete identifier mappings (missing SecID). If CostsAndCharges logs throw a skipping error due to missing IDs:

1. Verify the secondary identifiers in the Morningstar intake row in Instrument Admin
2. Confirm the `0P...` secId is missing (not just unmapped)
3. Escalate to Morningstar support for re-mapping (use productsupport@morningstar.com)
4. Include: ISIN, instrument ID, which product, which API call

## Open Actions

| Action | Owner | Status |
|--------|-------|--------|
| Contact Morningstar to request full secId/fundShareClassId mapping for LU3243907741 | Team Navigator / Team Wolf | Pending |
| Verify cost data populates after M* updates feed | Team Navigator | Pending |

## Links

- **Nordnet ETF page:** https://www.nordnet.se/etf/lista/bnp-paribas-easy-msci-eeab-xeta
- **ISIN:** LU3243907741
- **Internal Instrument ID:** 86075d2d-8fe3-4552-9821-9e5e363aa86a
- **Instrument Admin:** https://instrument-admin.tools.prod.nntech.io/Instruments
- **Slack thread:** #area-securities-brokerage (2026-07-10)

## Related Cases

- **INC-002** (Missing Morningstar ETF target_market Data and KID): Also a Morningstar data gap for ETF regulatory data, but caused by ISIN missing from RegXchange entirely. Different root cause, similar downstream symptom.
- **SC-002** (Missing Morningstar OnDemand EMT Data): Morningstar intake gap for mutual fund EMT data. Different pipeline but same vendor dependency.
