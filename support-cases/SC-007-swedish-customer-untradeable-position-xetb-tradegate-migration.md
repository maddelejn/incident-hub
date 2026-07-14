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

### Confirmed finding

The customer's position **is** linked to the Tradegate instrument (Abasec insid **469388**). This was confirmed by Lars Mattsson.

**What the customer sees:**
- German flag on the position in account overview
- Greyed out Buy/Sell buttons (because Tradegate is German-only)
- Real-time pricing from Tradegate (15 min delayed) - previously the position likely had once-a-day pricing or no live price when it was on the deleted Xetra instrument

The price change from once-a-day to real-time Tradegate pricing may itself be a source of customer confusion (Lars Mattsson's observation).

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

### Potential permanent fix
**Vienna Stock Exchange is available for electronic self-service trading on Nordnet.** This means if the customer's position were remapped from the deleted Xetra instrument to the corresponding Vienna instrument (AT000AGRANA3 / AGRv on XVIE), the customer should be able to sell electronically without needing the Trading Desk.

This requires:
1. Confirming that a Vienna instrument for ISIN AT000AGRANA3 exists in the instrument database
2. Remapping the customer's position from the deleted XETB instrument to the Vienna one
3. Evaluating whether this should be done systematically for all orphaned XETB positions where a Vienna (or other supported market) instrument exists

### Why the customer can't sell electronically

The customer's position is linked to the Tradegate instrument (insid 469388). The account overview shows a German flag with greyed out Buy/Sell buttons.

| Venue | Status | Why they can't sell |
|-------|--------|-------------------|
| Tradegate (position is here) | Active | German customers only - buy/sell greyed out |
| Xetra (XETB) | Deleted Sept 2025 | Instrument no longer exists |
| Vienna (XVIE) | Active, primary listing | Position not linked to Vienna instrument |

Vienna is available for electronic trading on Nordnet. If the position were remapped to a Vienna instrument, the customer could sell online.

### Customer communication guidance

**From Pierre (Product):** To Swedish customers, only say that **this instrument is not currently offered to be traded electronically via Nordnet.** Do NOT mention Tradegate, regional German exchanges, or the venue migration. Keep it simple.

**For selling:** The Trading Desk (confirmed by Simon Ljungberg) can sell on regional German exchanges (Frankfurt floor, Munich, Stuttgart, etc.) or Vienna.

### Open questions
1. **How widespread is this?** Are there more Nordic customers holding positions that got mapped to Tradegate after the September 2025 XETB cleanup?
2. **Should Nordic customers' positions be remapped away from Tradegate?** For non-German customers, positions on Tradegate instruments should ideally point to a venue they can actually trade on (e.g. Vienna). Need to find a logic for this.
3. **Does a Vienna instrument exist for AT000AGRANA3?** If so, remapping would immediately restore electronic trading.
4. **Pricing confusion:** Lars noted that positions mapped to Tradegate now show real-time (15 min delayed) Tradegate prices, whereas before they may have had once-a-day or no live pricing. This change in price behavior (plus the German flag appearing) is likely what triggers customer calls.

## Nordnet Supported Electronic Trading Markets

For reference, these markets are available for self-service electronic trading (web and app):

| Region | Markets |
|--------|---------|
| Nordics | Sweden, Norway, Denmark, Finland (excl. Iceland) |
| Germany | Xetra (+ Tradegate for German customers, rolling out) |
| USA | NYSE, Nasdaq |
| Canada | TSX, Venture |
| UK | London Stock Exchange (FTSE350 only) |
| Belgium | Euronext Brussels |
| France | Euronext Paris |
| Ireland | Euronext Dublin |
| Italy | Milano Stock Exchange |
| Netherlands | Euronext Amsterdam |
| Portugal | Euronext Lisbon |
| Switzerland | SIX Swiss Exchange |
| Austria | Vienna Stock Exchange |
| Spain | Madrid Stock Exchange |
| Poland | Warsaw Stock Exchange (phone orders only) |

## Tradegate UI Behavior (Search, Instrument Page, Positions)

Understanding how Tradegate instruments appear in the UI is important for diagnosing customer confusion.

### Search behavior

The search has two modes:

| Mode | Behavior | Example (TRATON SE) |
|------|----------|---------------------|
| **Preferred tradable(s)** | Shows one result per instrument. Xetra is preferred over Tradegate. Tradegate is hidden when Xetra exists. | Shows "TRATON SE" in SEK and one EUR result (Xetra) |
| **All tradable(s)** | Shows all venues separately with venue labels | Shows "8TRA - Xetra" EUR and "8TRA - Tradegate" EUR as separate rows |

For non-German customers, Tradegate instruments should not appear in search at all (search = false in operation rules).

### Instrument page and order dialog

- German customers see a **venue selector dropdown** on the instrument page showing available venues (e.g. "Xetra" with checkmark, "Tradegate")
- The **Order Dialog** has a "Trade venue" selector where the customer can pick Xetra or Tradegate before placing the order
- Xetra is the default/preferred venue when both are available

### Position page (fungible vs non-fungible display)

| Display mode | Behavior | What the customer sees |
|-------------|----------|----------------------|
| **Fungible** | One position row regardless of venue | "TRATON SE" EUR with Buy/Sell buttons. Customer picks venue at trade time. |
| **Non-fungible** | Separate rows per venue | "TRATON SE - Xetra" EUR and "TRATON SE - Tradegate" EUR, each with own Buy/Sell |

For fungible instruments (which is the normal Xetra/Tradegate case), the customer sees a single position and chooses the venue when they trade. This is consistent with the `tradableMappings` model in the instrument-id-mapper.

## Tradegate Trading Details (from Horizon Demo)

### Trading hours

Tradegate offers significantly extended trading hours compared to Xetra:

| Venue | Trading hours (CET) |
|-------|-------------------|
| Xetra | 09:00 - 17:30 |
| Tradegate | 07:30 - 22:00 |

This is a key selling point for German retail customers - the ability to react to market events outside standard Xetra hours.

### Order flow architecture

```
Customer → Nordnet Platform → Virtu (Service Bureau) → Tradegate BSX
                                                            ↓
                                              Clearstream (CCP/CSD)
                                                            ↓
                                                   Citi (Account Operator)
```

- Nordnet holds the **Direct Membership** at Tradegate BSX
- **Virtu** provides the technical connectivity and order routing under Nordnet's membership credentials
- Orders are routed via Virtu's infrastructure but executed under Nordnet's exchange member ID
- Settlement flows through **Clearstream** to **Citi** (Nordnet's account operator)

### Instrument universe

Tradegate covers approximately **10,000+ instruments** including:
- German equities (DAX, MDAX, SDAX constituents)
- International equities (US, European blue chips)
- ETFs and ETPs

Only **equities and depository receipts** are in scope for the Nordnet rollout. ETFs/ETPs are excluded from the initial launch.

### Key differences from Xetra

| Aspect | Xetra | Tradegate |
|--------|-------|-----------|
| Trading model | Central order book (auction + continuous) | Quote-driven (specialist/market maker) |
| Trading hours | 09:00 - 17:30 CET | 07:30 - 22:00 CET |
| Order types (Nordnet) | Limit | Limit initially, Market orders later |
| Primary users | Institutional + retail | Retail-focused |
| MIC | XETR | XGAT/XGRM |
| Market ID | 4 | 85 |

### Fungibility in practice

When a German customer holds a position bought on Xetra and wants to sell on Tradegate (or vice versa):
- No position transfer needed - it's the same clearing position at Clearstream
- The UI shows one position row (fungible display)
- Customer picks the venue in the order dialog at trade time
- Settlement happens seamlessly through the same CSD

### What German customers see (end-to-end flow)

1. **Search**: Customer searches for a stock (e.g. "TRATON"). Sees one result in preferred view (Xetra). Can expand to "All tradables" to see Tradegate option.
2. **Instrument page**: Shows market data. Venue selector dropdown shows Xetra (default) and Tradegate.
3. **Order dialog**: Customer selects Buy/Sell, picks trade venue (Xetra or Tradegate), enters order details.
4. **Position page**: Shows one row for the holding. Buy/Sell buttons open order dialog with venue selection.

### What non-German customers see

Nothing. Tradegate instruments are completely hidden:
- Not visible in search
- No venue selector on instrument pages
- Cannot place orders

This is why the SC-007 case is confusing - the Swedish customer somehow encountered a Tradegate instrument despite these restrictions.

## NPAP Change Record

Tradegate was formally approved through NPAP **Change 362: Trading: Tradegate BSX**.

| Field | Detail |
|-------|--------|
| Change Business Owner | Quincy Curry (Director of Securities Brokerage) |
| Change Initiative Owner | Björn Alenvik (Area Product Owner, Securities Brokerage) |
| Target timeline | H1 2026 (June) |
| Scope | German customers only, equity instruments only |
| Exchange membership | Nordnet as Direct Member of Tradegate BSX |
| Order routing | Via Virtu (service bureau under Nordnet's membership) |
| Settlement | Tradegate → Clearstream → Citi (account operator) |
| Order types | Limit Orders initially. Market Orders planned for later stage. |
| Applicable regulations | MiFID II/MiFIR, BörsG (German Stock Exchange Act), MAR, CSDR, EU PFOF Ban |
| FTT impact | French (0.40%), Italian (0.20%), Spanish (0.20%) FTT applies based on issuer jurisdiction - but Nordnet does not plan to offer those shares on Tradegate |
| Reporting | TRS and ORK reporting required. ORK short-code mapping routed via Virtu. |
| Key vendors | Tradegate BSX, Virtu, Citi, Clearstream, Millistream, Morningstar, ICE, Deutsche Börse Group |
| Confluence | https://nordnetbank.atlassian.net/wiki/spaces/RC/pages/1611759680 |

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
| Reporter | Rasmus Nilsson (Customer Service) | Reported customer unable to sell, identified instrument in instrument admin |
| Investigation | Lars Mattsson (Team Wolf) | Identified position on Tradegate insid 469388, noted pricing change impact |
| Investigation | Madde (Team Wolf PO) | Investigated XETB deletion history, back-end instrument mapping |
| Trading Desk | Simon Ljungberg | Confirmed Trading Desk can sell on regional German exchanges |
| Product guidance | Pierre | Customer communication guidance: don't mention Tradegate/regional exchanges to Swedish customers |

## Related

- **INC-003** (Dual-Listed Virtune ETP): Different issue - dual-listing with separate instrument lines, not fungible positions. Same customer symptom (can't trade), different root cause.
- **SC-006** (INET Gateway Decommission): Tradegate instrument creation flow context.
- **Germany Expansion Vendor Strategy** (`products/germany-expansion-vendor-strategy.md`): Broader context for Tradegate rollout.
- **Confluence - Technical Details**: https://nordnetbank.atlassian.net/wiki/spaces/DE/pages/1884717057
- **Confluence - Xetra to Abasec via NNX**: https://nordnetbank.atlassian.net/wiki/spaces/DE/pages/1440153669
- **Confluence - Roll Out Plan**: https://nordnetbank.atlassian.net/wiki/spaces/DE/pages/1866596377
