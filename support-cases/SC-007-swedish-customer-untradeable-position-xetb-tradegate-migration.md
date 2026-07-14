---
id: SC-007
title: "Swedish Customer Untradeable Position After XETB Deletion - Tradegate Instrument Mapping"
date: 2026-07-14
status: open
requester: "Rasmus Nilsson (Customer Service)"
requester_team: "Customer Service"
service: "instrument-intake, instrument-admin, trading-platform"
team_responsible: "Team Wolf"
product_owner: "Team Wolf (Madde)"
category: instrument-mapping
frequency: recurring
faq_candidate: true
tags: [tradegate, xetra, xetb, fungible, german-expansion, market-85, venue-cleanup, swedish-customers, non-german, position-mapping, millistream, agrana]
---

# SC-007: Swedish Customer Untradeable Position After XETB Deletion

## Context

Rasmus Nilsson (Customer Service) reported that Swedish customers are unable to sell stocks electronically that they originally purchased on Xetra. The specific case involves account **10791903** holding AGRANA (ISIN: AT000AGRANA3), purchased on Xetra in June 2024.

## What Happened

### The instrument lifecycle

1. **June 2024:** Customer bought 7 shares of AGRANA at €14.10 each on Xetra (XETB - Xetra Freiverkehr / Regulated Unofficial Market).
2. **September 18, 2025:** Millistream deleted the XETB instrument from the data feed. Deutsche Börse had decommissioned the Xetra electronic order book for this ISIN due to a venue cleanup (low volume / no active Designated Sponsor).
3. **Post-deletion:** The customer's position became associated with the Tradegate instrument (AGB2), since Tradegate is the most active German retail venue still trading this ISIN.
4. **Now:** The customer cannot sell electronically because Tradegate (Market ID 85) is restricted to German customers only.

### Key finding

On investigation, the customer's position in this specific case had **not** actually been mapped to the Tradegate instrument on the back-end - it still sat on the old Xetra one. The question back to Rasmus was: how did the customer find/view the Tradegate instrument page if it's not mapped?

## Why This Happens

### XETB vs XETR

AGRANA was never on the main Xetra Regulated Market (MIC: XETR). It was on **Xetra Freiverkehr** (MIC: XETB) - the Regulated Unofficial / Open Market segment on Deutsche Börse's electronic platform. Deutsche Börse periodically cleans up XETB, removing instruments with low volume or no active Designated Sponsor.

### The stock is NOT delisted

AGRANA remains fully listed and actively trading on:
- **Vienna Stock Exchange** (Wiener Börse, Prime Market, MIC: XVIE) - primary listing, ticker AGRv
- **German Freiverkehr venues** (Berlin, Düsseldorf, Munich, Stuttgart, Frankfurt floor, Tradegate) - ticker AGB2

Only the Xetra electronic order book (XETB) was decommissioned.

### Tradegate/Xetra fungibility

Tradegate and Xetra instruments sharing the same ISIN are **fungible** - they represent the exact same position at the clearing house. A customer can buy on one venue and sell on the other. This is fundamentally different from dual-listing (see INC-003), where separate instrument lines exist per market.

The same ISIN is displayed because it truly is the same security and the same clearing position. The German flag on Tradegate instruments indicates the trading venue, not a different security.

### Why non-German customers get stuck

The Tradegate rollout (Market ID 85) has explicit operation rules restricting access to German customers only:

| Customer | Buy | Sell | Search |
|----------|-----|------|--------|
| Non-German | false | false | false |
| German (Tradegate-only) | true | true | true |
| German (Xetra + Tradegate) | true | true | false |

When an instrument migrates from Xetra to Tradegate (due to XETB cleanup), non-German customers holding positions lose electronic trading access.

## Resolution

### Immediate workaround
The customer can sell via the **Trading Desk** (manual broker intervention). Because the shares are fungible, a broker can execute a sell order on the Vienna Stock Exchange (XVIE) where the primary listing is active and liquid.

### Open questions
1. **How widespread is this?** Are there more Swedish/Nordic customers holding positions in instruments that moved from XETB to Tradegate after the September 2025 cleanup?
2. **Should orphaned XETB positions be remapped?** For non-German customers, mapping to Vienna (XVIE) instead of Tradegate would restore electronic trading access - but it's unclear if Vienna is a supported trading venue on the platform.
3. **How did the customer find the Tradegate instrument page?** In this specific case the position wasn't mapped to Tradegate on the back-end. Need to understand if customers are finding it via search or if the UI is showing it incorrectly.

## Tradegate Technical Details (Market ID 85)

For reference, Tradegate is being rolled out as part of the Germany expansion:

| Field | Value |
|-------|-------|
| MICs | XGAT, XGRM |
| On-prem Market ID | 85 |
| Custody | Clearstream Europe via Citi |
| Instrument types | Equities and Depository Receipts |
| Abasec clearing place | TBSX (defaultClearing = false, defaultMarket = false) |
| Country code | DE |
| Default market | TYS |
| Default clearing | CITI |

### Fungibility modeling in the system

The `instrument-id-mapper` uses a `tradableMappings` array to model fungible order books. A single instrument maps to both a Xetra order book (Market ID 4) and a Tradegate order book (Market ID 85):

```json
{
  "tradableMappings": [
    {
      "tradingOrderBookId": "...",
      "marketDataOrderBookId": "...",
      "legacyMarketAndIdentifier": { "marketId": 4, "identifier": "SYZ" }
    },
    {
      "tradingOrderBookId": "...",
      "marketDataOrderBookId": "...",
      "legacyMarketAndIdentifier": { "marketId": 85, "identifier": "SYZ" }
    }
  ]
}
```

### Order book priority

When both Xetra and Tradegate are available, Xetra is preferred. The operation rules enforce this via search visibility (Tradegate hidden when Xetra exists) and priority ordering:

1. Xetra/Tradegate (Xetra before Tradegate)
2. US listings
3. Nordic listings
4. Primary market
5. Paris, Amsterdam, Brussels, Lisbon, Dublin
6. Canada
7. Milan
8. London
9. Switzerland
10. Spain
11. Austria
12. OTC

## Stakeholders

| Role | Person/Team | Action |
|------|-------------|--------|
| Reporter | Rasmus Nilsson (Customer Service) | Reported customer unable to sell |
| Escalation | Lars Mattsson (Team Wolf) | Confirmed Tradegate not live, suggested broker workaround |
| Investigation | Madde (Team Wolf PO) | Investigated back-end mapping, confirmed position not mapped to Tradegate |

## Related

- **INC-003** (Dual-Listed Virtune ETP): Different issue - dual-listing with separate instrument lines, not fungible positions. Same customer symptom (can't trade), different root cause.
- **SC-006** (INET Gateway Decommission): Tradegate instrument creation flow context.
- **Germany Expansion Vendor Strategy** (`products/germany-expansion-vendor-strategy.md`): Broader context for Tradegate rollout.
- **Confluence - Technical Details**: https://nordnetbank.atlassian.net/wiki/spaces/DE/pages/1884717057
- **Confluence - Xetra to Abasec via NNX**: https://nordnetbank.atlassian.net/wiki/spaces/DE/pages/1440153669
- **Confluence - Roll Out Plan**: https://nordnetbank.atlassian.net/wiki/spaces/DE/pages/1866596377
