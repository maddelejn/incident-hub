---
id: INC-001
title: "Stuck Instrument Blocking Client Order Routing"
date: 2026-07-09
detected_at: ""
resolved_at: ""
severity: P2
status: resolved
service: "instrument-markets, exchange-trading-order-flow"
team: "Team Coresys, Team Wolf, Team ET"
product_owner: "Team Wolf (Madde), Team Hodor (Ben Ammerman)"
category: configuration
root_cause_category: code-bug
on_call_responder: ""
tags: [instrument, order-routing, non_instrumentid, instrument_markets, deregdate, coresys, market-reload, on-prem, core-database]
---

# INC-001: Stuck Instrument Blocking Client Order Routing

## Summary

An invalid or corrupted instrument row (`non_instrumentid = 294259`) was present in the system but missing from `instrument_markets`, creating a blocking state that prevented exchange trading connectivity and order routing. A client was completely unable to trade several instruments, missing same-day trading opportunities until the fix was applied.

## Timeline

| Time (UTC) | Event |
|------------|-------|
| TBD | Client reported inability to trade several instruments |
| TBD | Trading Desk SE (Lars Kormu, Lars Mattsson) escalated |
| TBD | Team Hodor (Josef) identified problematic `non_instrumentid = 294259` |
| TBD | Josef recommended setting a `deregdate` rather than hard deletion |
| TBD | Team Coresys (Christer) manually deregistered the instrument |
| TBD | Coresys reloaded market IDs 11 and 21 |
| TBD | Order routing restored, client able to trade |

## Impact

- **Users affected:** At least one client, potentially more
- **Duration:** Unknown (same-day impact confirmed)
- **Business impact:** Client missed trading opportunities
- **SLA breached:** Unknown

## Root Cause

An invalid or corrupted instrument row (`non_instrumentid = 294259`) existed in the system but was missing from `instrument_markets`. This inconsistency created a blocking state in the exchange trading connectivity layer, preventing order routing for affected instruments across multiple markets.

## Resolution

1. **Team Hodor (Josef Al-Shorji):** Identified the problematic ID and recommended setting a `deregdate` instead of hard deletion.
2. **Team Coresys (Christer Richardsson):** Manually deregistered `non_instrumentid = 294259`.
3. **Team Coresys:** Reloaded market IDs **11** and **21** to propagate the fix.

**Note:** This was an on-prem/core infrastructure database adjustment via Coresys. The Abasec → NNX → Hodor pipeline was **not** required.

## Stakeholders

| Role | Person/Team | Action |
|------|-------------|--------|
| Identification | Team Hodor (Josef Al-Shorji) | Found the problematic ID, recommended fix approach |
| Fix execution | Team Coresys (Christer Richardsson) | Deregistered instrument, reloaded markets |
| Product Owner | Team Wolf (Madde) | Informed |
| Product Owner | Team Hodor (Ben Ammerman) | Informed |
| Escalation | Trading Desk SE (Lars Kormu, Lars Mattsson) | Raised the issue |
| Involved | Team Wolf (Niklas), Team ET (Kalle) | Assisted investigation |

## Runbook: Stuck Instrument Blocking Order Routing

If an instrument is blocking order routing across multiple markets:

1. Check if a stale/corrupted instrument row exists outside `instrument_markets` (look up the `non_instrumentid`).
2. **Do NOT hard delete** - set a `deregdate` instead.
3. Escalate to **Team Coresys** - Teams Hodor and Wolf do not have safe write access to core databases.
4. After deregistration, Coresys must **reload the impacted market IDs** (e.g., 11, 21) to propagate the change.
5. Verify order routing is restored.

## Action Items

| Action | Owner | Status | Due Date |
|--------|-------|--------|----------|
| Investigate how the corrupted instrument row was created | Team Coresys / Team Wolf | Open | TBD |
| Evaluate giving Hodor/Wolf safe access to deregister instruments without Coresys escalation | Team Coresys | Open | TBD |
| Add monitoring for instrument rows that exist but are missing from instrument_markets | Team Hodor / Team ET | Open | TBD |

## Lessons Learned

- **Cross-team dependency:** Teams Hodor and Wolf identified the issue and the fix but could not apply it themselves - Coresys escalation was required. This adds time during live incidents.
- **Soft delete over hard delete:** Setting a `deregdate` is the safe approach for stale instruments, not deletion.
- **No proactive detection:** This was caught because a client complained. A monitoring check for orphaned instrument rows could catch this before it impacts customers.

## Related Cases

None yet. Watch for similar cases involving instrument data inconsistencies blocking order routing.
