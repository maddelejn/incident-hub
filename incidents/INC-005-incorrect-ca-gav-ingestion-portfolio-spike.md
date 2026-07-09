---
id: INC-005
title: "Incorrect Corporate Action GAV Ingestion Causing Portfolio Returns Spike"
date: 2026-07-09
detected_at: ""
resolved_at: ""
severity: P2
status: resolved
service: "returns-graph, valuation-prices, corporate-actions"
team: "Team Marlin, Team Hodor"
product_owner: "Team Wolf (Madde), Team Hodor (Ben Ammerman)"
category: data-pipeline
root_cause_category: code-bug
on_call_responder: ""
tags: [corporate-action, gav, valuation-price, returns-graph, abasec, nnx, portfolio-spike, instrument-pricing, zero-value, middle-office]
---

# INC-005: Incorrect Corporate Action GAV Ingestion Causing Portfolio Returns Spike

## Summary

During a Corporate Action (CA) event, shares were moved into a customer's account with a GAV of 10 NOK, but the underlying instrument had a market value of 0 NOK as of June 30. This mismatch caused the graph engine to calculate an artificial 100% gain between June 30 and July 1 instead of the correct minor decrease. Resolved via the standard Abasec → Marlin → Hodor correction pipeline.

## Timeline

| Time (UTC) | Event |
|------------|-------|
| June 30 | Corporate action moved shares into customer account at GAV of 10 NOK |
| June 30 | Instrument market value incorrectly mapped as 0 NOK |
| July 1 | Artificial 100% gain spike appeared on customer's returns graph |
| TBD | Issue identified and escalated |
| TBD | Middle Office (Mathias) corrected historical closing price to 10 NOK in Abasec |
| TBD | Team Marlin published corrected price to NNX via #valuation-prices-on-nnx |
| TBD | Team Hodor ingested corrected data, graph healed |

## Impact

- **Users affected:** At least one customer with visible portfolio distortion
- **Duration:** June 30 - July 1 until correction processed
- **Business impact:** Major artificial asset value spike on returns graph, undermines customer trust in portfolio data
- **SLA breached:** Unknown

## Root Cause

During a Corporate Action event, shares were booked into the customer's account with a **GAV of 10 NOK**. However, the underlying instrument was incorrectly mapped with a **market value of 0 NOK** as of June 30.

The graph engine calculated: asset goes from 0 NOK → 10 NOK = **100% gain**, when the actual performance was a minor decrease (the security was trading below 10 NOK).

**Procedural gap:** There is no systematic validation that a valid valuation price exists on an instrument record before a corporate action moves shares into it. The CA booking and the instrument pricing are independent processes with no cross-check.

## Resolution

Standard backdated correction pipeline (Abasec → NNX → Hodor):

1. **Middle Office (Mathias Persson Almqvist):** Retroactively updated the historical closing price to 10 NOK per June 30 in Abasec.
2. **Team Marlin:** Notified via `#valuation-prices-on-nnx` to publish the corrected historical price to NNX.
3. **Team Hodor:** Graph engine ingested the backdated data from NNX, healing the customer's graph and restoring accurate historical returns.

## Stakeholders

| Role | Person/Team | Action |
|------|-------------|--------|
| Source correction | Middle Office (Mathias Persson Almqvist) | Fixed historical price in Abasec |
| Price publishing | Team Marlin | Published corrected price to NNX |
| Graph processing | Team Hodor (Ben Ammerman) | Ingested correction, healed graph |
| Product Owner | Madde (Team Wolf) | Flagged need for process improvement |

## Runbook: Corporate Action Causing Returns Graph Spike

When a corporate action creates an artificial spike/drop on a customer's returns graph:

1. **Check the instrument's valuation price** on the CA effective date - is it 0 or missing?
2. **Compare against the GAV** booked on the customer's position - a large discrepancy confirms the issue.
3. **Middle Office corrects** the historical closing price in Abasec to match the correct valuation.
4. **Notify Team Marlin** via `#valuation-prices-on-nnx` to publish the corrected price to NNX.
5. **Team Hodor** will automatically ingest and heal the graph once correct data is in NNX.

## Action Items

| Action | Owner | Status | Due Date |
|--------|-------|--------|----------|
| Evaluate process to ensure valid threshold valuation exists on instrument before CA shares are moved | Team Wolf / Middle Office | Open | TBD |
| Investigate automated validation: block or flag CA bookings where instrument market value is 0 or missing | Team Wolf / Team Marlin | Open | TBD |

## Lessons Learned

- **CA and pricing are decoupled but dependent:** Corporate action bookings and instrument valuations are independent processes, but the graph engine assumes both are correct. When one is wrong, the result is visually extreme.
- **Zero-value instruments are a recurring theme:** This is the same underlying pattern as SC-001 (allocation share priced at 0). Both involve assets entering accounts where the instrument's market value is 0, causing graph distortion.
- **Prevention > correction:** The fix pipeline works (Abasec → NNX → Hodor), but the real improvement is validating that instrument pricing is correct *before* the calendar day rolls over.

## Related Cases

- **SC-001** (Returns Graph Drop - Fictitious Allocation Pricing): Nearly identical pattern. IPO allocation share booked at 10 SEK but instrument market value was 0. SC-001 caused a graph *drop*; INC-005 caused a graph *spike*. Same root cause: 0-value instrument during a booking event.
- **INC-004** (Incorrect Valuation Price - Amaroq): Also a valuation pricing error distorting portfolios, but caused by wrong exchange source rather than a missing price. Same correction pipeline.
