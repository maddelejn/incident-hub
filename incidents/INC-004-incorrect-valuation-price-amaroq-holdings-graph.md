---
id: INC-004
title: "Incorrect Valuation Price for Amaroq Ltd (AMRQ) Distorting Holdings Graph"
date: 2026-07-09
detected_at: ""
resolved_at: ""
severity: P1
status: resolved
service: "price-data, holdings-graph, valuation-prices"
team: "Team Marlin, Team Hodor"
product_owner: "Team Marlin (David), Team Hodor (Ben Ammerman)"
category: data-pipeline
root_cause_category: human-error
on_call_responder: ""
tags: [valuation-price, holdings-graph, multi-listed, amaroq, amrq, abasec, nnx, middle-office, quarterly-valuation, gbx, iceland, lse, portfolio-distortion]
---

# INC-004: Incorrect Valuation Price for Amaroq Ltd (AMRQ) Distorting Holdings Graph

## Summary

Middle Office applied a quarterly valuation change for Amaroq Ltd (AMRQ) using a closing price from Nasdaq Iceland (137 GBX) instead of the primary London Stock Exchange rate (~88.5 GBX). This incorrect price was pushed retroactively to June 30, 2026, creating severe portfolio spikes in customer holdings graphs. High volume of customer calls resulted. Resolved via the standard Abasec → Marlin → Hodor correction pipeline.

## Timeline

| Time (UTC) | Event |
|------------|-------|
| TBD | Middle Office executed quarterly valuation change using Nasdaq Iceland price (137 GBX) |
| TBD | Incorrect price pushed retroactively to June 30, 2026 |
| TBD | Customer calls spiked due to extreme artificial portfolio increases |
| TBD | Trading Desk SE (Simon Ljungberg) escalated |
| TBD | Root cause identified: wrong exchange source used for valuation |
| TBD | Middle Office corrected historical rate in Abasec to LSE price (~88.5 GBX) |
| TBD | Team Marlin published corrected price from Abasec to NNX |
| TBD | Team Hodor consumed corrected data from NNX, graphs healed |

## Impact

- **Users affected:** All holders of Amaroq Ltd (AMRQ) - high volume of customer calls
- **Duration:** From valuation push until Hodor processed the correction
- **Business impact:** Extreme artificial portfolio value spikes visible to customers, high support call volume, reputational risk
- **SLA breached:** Unknown

## Root Cause

Middle Office executed a quarterly valuation change using the **wrong exchange source**. They used the closing price from **Nasdaq Iceland (137 GBX)** instead of the primary **London Stock Exchange (LSE)** rate (~88.5 GBX). This is a ~55% price discrepancy.

The incorrect price was pushed retroactively to **June 30, 2026**, creating severe distortions in portfolio valuations and holdings graphs for all affected customers.

This is a **manual process error** - multi-listed instruments carry high risk for incorrect source selection during quarterly corporate valuation events.

## Resolution

Standard backdated correction pipeline (Abasec → NNX → Hodor):

1. **Middle Office:** Corrected the historical transaction rate in Abasec to the correct LSE valuation price.
2. **Team Marlin:** Picked up and published the updated price files from Abasec to NNX.
3. **Team Hodor (@hodor_goalie):** Consumed the corrected historical update from NNX, healing affected user account balances and resolving graph distortions.

**Note:** Customer-facing graphs do not update instantaneously when a price fix is declared. Team Hodor must wait until Team Marlin completes data transmission from Abasec before processing can begin.

## Stakeholders

| Role | Person/Team | Action |
|------|-------------|--------|
| Source correction | Middle Office | Fixed historical rate in Abasec |
| Price publishing | Team Marlin (David) | Published corrected data to NNX |
| Graph processing | Team Hodor (Ben Ammerman, @hodor_goalie) | Consumed correction, healed graphs |
| Escalation | Trading Desk SE (Simon Ljungberg) | Flagged customer impact |

## References

- **LSE listing:** https://www.londonstockexchange.com/stock/AMRQ/amaroq-ltd/company-page
- **Nasdaq listing:** https://www.nasdaq.com/european-market-activity/shares/amrq?id=TX4619398

## Runbook: Incorrect Valuation Price on Multi-Listed Instrument

When a multi-listed instrument has wrong valuation pricing causing portfolio distortions:

1. **Identify the correct exchange source** - check which exchange is the primary listing for the account's domestic market.
2. **Middle Office corrects in Abasec** - update the historical transaction rate to the correct price.
3. **Team Marlin publishes** - pushes corrected price files from Abasec to NNX.
4. **Team Hodor processes** - consumes corrected data from NNX, graphs heal automatically.
5. **Note:** Steps are sequential. Hodor cannot process until Marlin has completed the push.

This is the same **Abasec → NNX → Hodor pipeline** used in SC-001 (fictitious allocation pricing).

## Action Items

| Action | Owner | Status | Due Date |
|--------|-------|--------|----------|
| Implement cross-check validation for multi-listed instruments during quarterly valuations | Middle Office / Team Marlin | Open | TBD |
| Evaluate automated source-exchange validation when prices diverge significantly across listings | Team Marlin | Open | TBD |

## Lessons Learned

- **Multi-listed instruments are high risk for manual valuations:** When an instrument trades on multiple exchanges at different prices, manual source selection is error-prone. A cross-check against the domestic account listing's primary exchange could prevent this.
- **Customer impact is immediate, fix is not:** The wrong price shows up in portfolios instantly, but the correction must flow through the full Abasec → Marlin → Hodor pipeline sequentially. This asymmetry drives high call volumes.
- **Validation gap:** There is no automated sanity check that flags when a valuation price diverges significantly from the instrument's primary exchange price.

## Related Cases

- **SC-001** (Returns Graph Drop - Fictitious Allocation Pricing): Same Abasec → NNX → Hodor correction pipeline. Different trigger (IPO allocation share vs wrong exchange source) but identical fix flow.
- **INC-003** (Dual-Listed Virtune ETP): Also a multi-listing issue, but affecting regulatory data mapping rather than pricing. Both highlight that multi-listed instruments are a recurring source of incidents.
