---
type: recommendation
title: "Germany ETP & FTT Vendor Recommendation: SIX over WM Daten"
status: draft
prepared_by: "Madde (PO Instrument Intake), Pierre"
audience: "Joachim Wegebrand, Edward Neptune, Team Match, Pierre"
date: 2026-07-10
decision_needed_by: August 2026
---

# Vendor Recommendation: SIX for Germany ETP & FTT Data

## Executive Summary

Nordnet is entering the German market and needs a data vendor for three purposes: FTT (Financial Transaction Tax), ETP regulatory data (EMT/KID/Target Market), and Costs & Charges. We evaluated two vendors - SIX Financial Information and WM Daten. **SIX is the recommended vendor based on superior coverage, cleaner data format, lower integration cost, and an existing vendor relationship.**

Two open items remain before final decision: SIX's EMT/KID data format and combined pricing. These should be resolved in the August meeting with SIX.

## The Three Data Needs

| Need | Consumer | Regulatory Driver |
|------|----------|-------------------|
| **FTT/Tax data** | Team Match, Tax/Clearing | Required to enter German market (Abgeltungsteuer, foreign FTT) |
| **ETP regulatory data** (EMT, KID, Target Market) | Instrument Intake | Cannot offer ETPs to retail without KID/EMT (MiFID II, PRIIPs) |
| **Costs & Charges** | Team Match | Ex-ante cost disclosure mandatory before every trade (MiFID II) |

All three needs can be served by a single vendor, avoiding duplicate contracts and overlapping data purchases.

## Evaluation Summary

### Coverage

| Data Point | SIX | WM Daten | Advantage |
|------------|-----|----------|-----------|
| Target Market | 384/384 (100%) | 379/383 (99%) | SIX |
| Costs & Charges | 384/384 (100%) | 382/383 (99.7%) | SIX |
| KID documents | 379/384 (98.7%) | 379/383 (98.9%) | Tie |
| FTT countries | 13+ (FR, IT, ES, CH, UK, IE, FI, HK, and more) | 3 (FR, IT, ES) | SIX |

Both vendors have small KID gaps on the same ISINs (likely very new/obscure structured products). SIX confirmed their sourcing team can add the 5 missing KIDs if Nordnet proceeds.

### Data Format & Integration

| Aspect | SIX | WM Daten |
|--------|-----|----------|
| **Format** | Clean CSV, human-readable field names | Proprietary WM field codes (GD432, KD610, etc.) |
| **Integration effort** | Low - standard CSV parsing | High - requires custom mapping/translation layer |
| **Ongoing maintenance** | Minimal | Every WM field code change requires mapper updates |
| **Sample data provided** | Yes (350 FTT rows with actual values) | No (coverage check only - 5 yes/no fields) |

**This is the most significant differentiator.** WM Daten's proprietary field codes require building and maintaining a translation layer that maps cryptic codes to standard EMT fields. This is not a one-time cost - it's ongoing engineering overhead.

### FTT Specifics

| Aspect | SIX | WM Daten |
|--------|-----|----------|
| **Fields** | 15 (clean, structured) | 18 (proprietary, repeated with KD/UD/VD prefixes) |
| **Countries** | 13+ | 3 |
| **Tax rates included** | Yes (numeric) | TBD (no sample data) |
| **Valid from/to dates** | Yes | TBD |
| **Resident/non-resident** | Yes | TBD |
| **Delivery** | SIX Flex, API, Data Cloud | Daily CSV |
| **Technical Q&A answered** | Yes (tax currency, netting, calculation) | No |

### Vendor Relationship

| Aspect | SIX | WM Daten |
|--------|-----|----------|
| **Existing Nordnet vendor** | Yes (already providing sanctions data) | No (new vendor onboarding required) |
| **Language** | Swedish (Stockholm office) | English/German (Frankfurt) |
| **Sales contact** | Carl Sundman, local, responsive | Funda Üstün / Andreas Knabe, remote |
| **Responsiveness** | Provided sample data, answered detailed Q&A, completed coverage check in 1 day | Provided coverage check, referred to documentation for field details |

### Pricing

| | SIX | WM Daten |
|--|-----|----------|
| **FTT** | 150,000 SEK/year (up to 9,000 instruments), tiered above | Not provided |
| **EMT/KID/Target Market** | TBD (to be discussed August) | Not provided |
| **Bundle discount potential** | Likely (adding to existing vendor relationship) | Unknown |

SIX's FTT base price of 150k SEK/year comfortably covers the ~384 Xetra ETP universe within the base tier.

## Recommendation

**Proceed with SIX as the single vendor for FTT, EMT/KID, Target Market, and Costs & Charges.**

### Rationale

1. **Better coverage across all data categories** - 100% target market and costs vs 99%, 13+ FTT countries vs 3
2. **Dramatically lower integration cost** - clean CSV vs proprietary field codes requiring a custom mapping layer
3. **Existing vendor relationship** - no new procurement, legal, or technical onboarding
4. **Proven delivery** - sample data provided, detailed technical Q&A answered
5. **Future-proof** - multi-market coverage beyond Xetra, multiple delivery channels (Flex, API, Data Cloud)

### Risk assessment

| Risk | Likelihood | Mitigation |
|------|-----------|------------|
| SIX EMT/KID format is proprietary/difficult | Low (their FTT format is clean CSV) | Request sample file in August meeting |
| SIX combined pricing is too high | Medium | Leverage existing vendor relationship and bundle discount |
| WM Daten has deeper German-specific data we're missing | Low | WM Daten's only advantage was being the German NNA; SIX covers more FTT countries and had better ETP coverage |

## Open Items for August Meeting with SIX

These must be resolved before final contract:

1. **"Can you send a sample EMT/KID/Target Market delivery file for 10 Xetra ETPs?"**
   - Confirms data format and field structure
   - Validates that both fund (ETF) and structured product (ETC/ETN) cost buckets are included

2. **"What is the combined pricing for FTT + EMT/KID/Target Market/Costs & Charges?"**
   - Aim for bundle discount given existing sanctions data relationship
   - Compare against WM Daten if they provide pricing by then

3. **"How do you handle the fund vs structured product split for ETPs?"**
   - ETFs use "Costs — Funds" fields, ETCs/ETNs use "Costs — Structured Products"
   - Critical that the delivery covers both (see INC-002, INC-003 for what happens when regulatory data is missing)

4. **"What's the update frequency and delivery mechanism for EMT/KID data?"**
   - Daily? On-change?
   - Same SIX Flex channel as FTT?

## Decision Timeline

| Date | Action |
|------|--------|
| August 2026 | Team sync (Madde, Pierre, Joachim, Edward) |
| August 2026 | Meeting with Carl Sundman (SIX) - resolve open items |
| August 2026 | Request WM Daten pricing for comparison leverage |
| August 2026 | Final vendor decision |
| September 2026 | Contract negotiation and signing |
| Q4 2026 | Integration and testing |

## Appendix: All Supporting Documentation

| Document | Location |
|----------|----------|
| Consolidated vendor strategy | products/germany-expansion-vendor-strategy.md |
| SIX vendor details | products/germany-etp-vendor-six.md |
| WM Daten vendor details | products/germany-etp-vendor-wm-daten.md |
| FTT field comparison (SIX vs WM Daten) | reference/ftt-six-vs-wm-daten-comparison.md |
| FTT reference guide | reference/ftt-financial-transaction-tax.md |
| EMT/KID/PRIIP knowledge guide | reference/emt-kid-priip-target-market-guide.md |
| UCITS framework | reference/ucits-framework.md |
