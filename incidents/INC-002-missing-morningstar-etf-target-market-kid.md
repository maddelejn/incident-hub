---
id: INC-002
title: "Missing Morningstar ETF target_market Data and KID Document"
date: 2026-07-09
detected_at: ""
resolved_at: ""
severity: P2
status: resolved
service: "fund-enrichment, regxchange, etf-order-placement"
team: "Team Navigator, Team Wolf, Team Mint"
product_owner: "Team Wolf (Madde), Team Mint (Axel Karlsson)"
category: integration
root_cause_category: dependency-failure
on_call_responder: ""
tags: [morningstar, etf, kid, target-market, regxchange, mifid, priip, fund-enrichment, enrichedfundtopic, isin, order-placement]
---

# INC-002: Missing Morningstar ETF target_market Data and KID Document

## Summary

Customer order placement was blocked for ETF ISIN `IE0031442068` due to missing regulatory documentation (KID) and target market verification metadata. The ISIN was missing from RegXchange (RegX) on Morningstar's side, so the downstream fund-enrichment pipeline had no data to process. Resolved after Morningstar corrected their output overnight.

## Timeline

| Time (UTC) | Event |
|------------|-------|
| TBD | Missing KID/target_market data detected for ISIN IE0031442068 |
| TBD | Investigation confirmed ISIN missing from RegXchange |
| TBD | Madde escalated to Morningstar via email |
| TBD | Morningstar corrected output on their end (overnight) |
| TBD | Data flowed through fund-enrichment → EnrichedFundTopic.V2 → fund-bff → AboutETP |
| TBD | Filippa confirmed target_market populated via aggregate endpoint |
| TBD | Order placement unblocked |

## Impact

- **Users affected:** Customers attempting to trade ETF IE0031442068
- **Duration:** Until Morningstar corrected their data (overnight fix)
- **Business impact:** Blocked order placement for impacted ETF - regulatory data is mandatory
- **SLA breached:** Unknown

## Root Cause

The ISIN `IE0031442068` was missing from **RegXchange (RegX)** on Morningstar's side. Morningstar is the primary data vendor for ETF MiFID and PRIIP information. Because the data was absent at the source, the downstream automated infrastructure (`fund-enrichment` processing the `EnrichedFundTopic.V2` topic) had nothing to compile the target market attributes from.

This was a **vendor-side data gap**, not an internal processing failure.

## Resolution

1. **Madde (Team Wolf):** Escalated the data gap to Morningstar via email.
2. **Morningstar:** Corrected the output on their end overnight.
3. **Automatic propagation:** Data flowed through the existing pipeline:
   `fund-enrichment` → `EnrichedFundTopic.V2` → `fund-bff` → `AboutETP` component
4. **Filippa:** Confirmed target_market metadata populated at the aggregate endpoint, unblocking order entry.

**No manual internal fix was needed** - once Morningstar corrected the source data, the pipeline handled it automatically.

## Architecture Reference

```
Morningstar (RegXchange)
    ↓
fund-enrichment (Team Navigator)
    ↓
EnrichedFundTopic.V2 (Kafka topic)
    ↓
fund-bff → AboutETP component
    ↓
service-instrument aggregate endpoint (target_market)
```

ETF KIDs and fund dividends use this **exact same distribution pipeline**. The `EnrichedFundTopic.V2` event payload is the shared transport.

**Circuit breaker:** `GET /v2/funds/{orderBookId}` is handled by `fundEnrichmentClient` circuit-breaker.

## Stakeholders

| Role | Person/Team | Action |
|------|-------------|--------|
| Escalation to vendor | Madde (Team Wolf) | Emailed Morningstar |
| Verification | Filippa | Confirmed data populated |
| Pipeline owner | Team Navigator | Owns fund-enrichment intake |
| Target market owner | Team Mint (Axel Karlsson) | Owns EMT/target market for exchange-traded products |
| Business stakeholders | Pierre, Philip Adriansen | Informed |
| External vendor | Morningstar Support | Corrected RegXchange data |

## References

- **Aggregate endpoint:** `http://service-instrument.prod.nordnet.se/aggregate/instrument/isin_code/IE0031442068?expand=target_market`
- **Internal service:** `GET /v2/funds/{orderBookId}` via `fundEnrichmentClient` circuit-breaker

## Runbook: Missing ETF Regulatory Data (KID / target_market)

When an ETF is missing regulatory parameters like `target_market`:

1. **Check RegXchange first** - verify if the ISIN exists in RegX at all.
2. **If missing from RegX** → this is a Morningstar vendor issue. Escalate to Morningstar immediately. Do not waste time searching internal intakes.
3. **If present in RegX but missing downstream** → investigate `fund-enrichment` processing and the `EnrichedFundTopic.V2` topic (Team Navigator).
4. **Verify fix** via: `http://service-instrument.prod.nordnet.se/aggregate/instrument/isin_code/{ISIN}?expand=target_market`

## Action Items

| Action | Owner | Status | Due Date |
|--------|-------|--------|----------|
| Evaluate alerting for ISINs missing from RegXchange that should have regulatory data | Team Navigator / Team Mint | Open | TBD |
| Document Morningstar escalation process and SLA expectations | Team Wolf | Open | TBD |

## Lessons Learned

- **Check the vendor first:** When ETF regulatory data is missing, the fastest diagnostic is checking RegXchange. If the ISIN is absent there, no amount of internal investigation will find the problem.
- **Pipeline works when fed:** Once Morningstar corrected the data, the entire internal pipeline (fund-enrichment → topic → bff → UI) handled propagation automatically. No manual internal intervention was needed.
- **Shared pipeline:** ETF KIDs and fund dividends share the same `EnrichedFundTopic.V2` pipeline - issues in one could signal issues in the other.

## Related Cases

- **SC-002** (Missing Morningstar OnDemand EMT Data): Also a Morningstar data gap, but for mutual fund EMT data via a different intake path (`daemon-intake-funds-morningstar1`). SC-002 required manual internal sync; this one auto-healed once the vendor corrected. Both highlight Morningstar as a single point of failure for regulatory data.
