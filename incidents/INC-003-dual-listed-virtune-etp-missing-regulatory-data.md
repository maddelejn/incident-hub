---
id: INC-003
title: "Dual-Listed Virtune ETP Missing Regulatory Data on Stockholm and Helsinki Markets"
date: 2026-07-09
detected_at: ""
resolved_at: ""
severity: P1
status: resolved
service: "instrument-intake, etp-order-placement, regulatory-data-ingestion"
team: "Team Wolf, Team Mint"
product_owner: "Team Wolf (Madde), Team Mint (Axel Karlsson), Team Pre-Trade (Björn Alenvik)"
category: integration
root_cause_category: code-bug
on_call_responder: ""
tags: [dual-listing, etp, virtune, crypto, emt, kid, target-market, costs-and-charges, xetra, helsinki, stockholm, regulatory, instrument-intake, multi-exchange, cm-ticket]
---

# INC-003: Dual-Listed Virtune ETP Missing Regulatory Data on Stockholm and Helsinki Markets

## Summary

A newly listed Virtune crypto product was dual-listed on Nasdaq Stockholm (STO SEK) and Nasdaq Helsinki (HEL EUR). The automated ingestion pipeline mapped all regulatory data (EMT, KID, costs and charges) exclusively to the existing Xetra version, leaving the STO and HEL instruments without the data needed for order placement. Clients were blocked from trading, and the asset manager's marketing campaign launch was delayed. Resolved via manual data copy from the Xetra instrument to Helsinki.

## Timeline

| Time (UTC) | Event |
|------------|-------|
| TBD | Virtune crypto product listed on STO and HEL markets |
| TBD | Clients reported "missing KID/EMT data" when attempting orders |
| TBD | Trading Desk SE (Lars Mattsson) and business (Joachim Wegebrand) escalated |
| TBD | Investigation confirmed regulatory data only mapped to Xetra instrument (19550499) |
| TBD | Lars Mattsson (Team Wolf) applied manual CM-ticket fix to Helsinki instrument (19700329) |
| TBD | Order placement restored on STO and HEL |

## Impact

- **Users affected:** All clients attempting to buy/sell the Virtune crypto product on STO and HEL
- **Duration:** From listing until manual fix applied
- **Business impact:** Blocked order placement on two markets. Asset manager (Virtune) unable to launch marketing campaign on schedule.
- **SLA breached:** Unknown

## Root Cause

The product was dual-listed across multiple markets. Because it was listed in **EUR on both Xetra and Helsinki**, the automated ingestion pipeline failed to distinguish between them and mapped all regulatory data attributes exclusively to the existing Xetra version (`instrument_id = 19550499`). This left the Stockholm and Helsinki instrument lines without:

- Active target market identifiers
- Costs and charges profiles
- KID document mappings

**This is a systemic issue:** the ingestion pipeline does not correctly handle dual-listed ETPs that share a currency across distinct European exchanges.

## Resolution

Manual data-fix (CM-ticket) applied by Lars Mattsson (Team Wolf). Copied properties from the fully-enriched Xetra instrument to the Helsinki instrument (`instrument_id = 19700329`):

1. Updated instrument asset class to Crypto (`CRY`)
2. Set `target_market_id = 1426485` for the Helsinki instrument
3. Copied 6 rows of KID documentation mapping from Xetra using `instrument.kid_id_seq.nextval` sequence
4. Copied all associated cost and charge attributes (ongoing costs, management fees, transaction fees, etc.) into `instrument.costs_and_charges`

**Note:** Since this was a static metadata reference fix on on-prem tools, the Abasec → NNX → Hodor pipeline was not needed.

## Manual Fix Detail

```
Source instrument (Xetra):    instrument_id = 19550499
Target instrument (Helsinki): instrument_id = 19700329
ISIN: SE0028425314

Data copied:
- asset_class → 'CRY'
- target_market_id → 1426485
- kid mapping → 6 rows via instrument.kid_id_seq.nextval
- costs_and_charges → all rows (ongoing costs, mgmt fees, tx fees, etc.)
```

## Stakeholders

| Role | Person/Team | Action |
|------|-------------|--------|
| Manual fix | Lars Mattsson (Team Wolf) | Applied CM-ticket data fix |
| Product Owner | Madde (Team Wolf) | Coordinated |
| Product Owner | Axel Karlsson (Team Mint) | EMT/target market owner for ETPs |
| Product Owner | Björn Alenvik (Team Pre-Trade) | Informed |
| Business owner | Joachim Wegebrand | Escalated business impact |
| Trading Desk SE | Lars Mattsson | Escalated and fixed |
| Involved | Kalle Pettersson (Team ET) | Assisted investigation |

## References

- **Aggregate endpoint:** `http://service-instrument.prod.nordnet.se/aggregate/instrument/isin_code/SE0028425314`
- **Instrument Admin:** `https://instrument-admin.tools.prod.nntech.io/Instruments?query=SE0028425314&page=1&includeDeregistered=false&pageSize=20`

## Runbook: Dual-Listed ETP Missing Regulatory Data

When a dual-listed ETP is missing KID/EMT/costs data on one or more markets:

1. **Identify all instrument_ids** for the ISIN across markets via instrument-admin.
2. **Find the fully-enriched version** (usually the first-listed exchange, e.g. Xetra).
3. **Apply CM-ticket** to copy from the enriched instrument to the broken one:
   - Asset class
   - `target_market_id`
   - KID mapping rows (use `instrument.kid_id_seq.nextval`)
   - `costs_and_charges` rows
4. **Verify** via aggregate endpoint: `http://service-instrument.prod.nordnet.se/aggregate/instrument/isin_code/{ISIN}?expand=target_market`

## Action Items

| Action | Owner | Status | Due Date |
|--------|-------|--------|----------|
| Architectural sync (post-vacation) to design automated data-mapping for multi-exchange ETP listings | Team Wolf + Team Mint | Open | TBD |
| Investigate ingestion pipeline logic for same-currency dual listings across exchanges | Team Wolf | Open | TBD |
| Evaluate proactive detection for newly listed instruments missing regulatory data | Team Mint / Team Pre-Trade | Open | TBD |

## Lessons Learned

- **Systemic gap:** The ingestion pipeline fundamentally cannot distinguish dual-listed ETPs sharing a currency across different exchanges (Xetra EUR vs Helsinki EUR). Every new dual-listing with this pattern will hit the same problem until this is fixed architecturally.
- **Manual fix is repeatable but costly:** The CM-ticket approach works but requires deep knowledge of the instrument tables and is error-prone. This should not be the steady-state process.
- **Business impact amplifier:** Regulatory data blocks don't just prevent trading - they can delay asset manager marketing campaigns and damage commercial relationships.

## Related Cases

- **INC-002** (Missing Morningstar ETF target_market Data and KID): Also ETF regulatory data missing, but caused by vendor-side gap in RegXchange. INC-003 is an internal pipeline mapping failure - different root cause, same customer-facing symptom ("missing KID/EMT data").
- **SC-002** (Missing Morningstar OnDemand EMT Data): Related regulatory data pipeline, different intake path (mutual funds vs ETPs).
