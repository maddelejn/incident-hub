---
type: runbook
title: "Whitelist Operations - Adding Missing Tradeable Instruments"
source: "https://nordnetbank.atlassian.net/wiki/spaces/DE/pages/80809000"
last_updated: 2026-07-13
---

# Whitelist Operations: Adding Missing Tradeable Instruments

**Source:** [Confluence](https://nordnetbank.atlassian.net/wiki/spaces/DE/pages/80809000)

## Problem

Sometimes instruments can't be added to the whitelist because a "tradeable" is missing. The system needs the tradeable to look up market → opening hours.

Securities-lending has its own INSTRUMENT table (because old instruments from service-instrument could disappear). When adding a new instrument, it first checks this table, then falls back to service-instrument.

## How to Add an Instrument Manually

**Trigger:** Email with title "Nytt SL-papper" (New SL instrument) reporting that tradeable is missing when adding to whitelist.

### Steps

1. **Get the SL Abasec Instrument ID** from the regular Abasec ID:
   ```
   http://service-core.prod.nordnet.se/service-core/securities_lending/instrument/{abasecInsId}
   ```

2. **Add the instrument** via Swagger POST:
   ```
   http://service-securities-lending.prod.nordnet.se/swagger-ui.html#/instrument-controller/addInstrumentUsingPOST
   ```

3. **Payload:**
   ```json
   {
     "name": "SATS ASA",                    // From service-instrument (not critical)
     "isin": "NO0010863285",                // Must be correct
     "abasecInsId": 514567,                 // Abasec instrument ID
     "slAbasecInsId": 514570,               // SL instrument ID (from service-core lookup above)
     "marketId": 15,                        // Market ID where it trades
     "settlementLocation": "NO3"            // Clearing place (SE1, DK2, FI2, NO3)
   }
   ```

### Settlement Location Mapping

| Code | Market |
|------|--------|
| SE1 | Sweden |
| DK2 | Denmark |
| FI2 | Finland |
| NO3 | Norway |

Full mapping in `SettlementLocationMapper` class in the SecLending project.
