---
id: INC-006
title: "Funds Deleted and Not Visible on Web Due to Morningstar Intake Bug"
date: 2025-03-06
detected_at: "2025-03-06"
resolved_at: "2025-03-06"
severity: P1
status: resolved
service: "morningstar-intake, instrument-master, fund-orderbooks"
team: "Team Wolf"
product_owner: "Team Wolf (Madde)"
category: deployment
root_cause_category: code-bug
on_call_responder: ""
tags: [morningstar, intake, funds, orderbooks, deletion, instrument-master, customerinstrumentid, swap-orders, monthly-savings, config-shared-envs]
jira: ["TP-27847", "TP-27841", "TP-27845"]
---

# INC-006: Funds Deleted and Not Visible on Web Due to Morningstar Intake Bug

## Summary

On March 5, 2025, Team Wolf added additional fields to the Morningstar download config in test. Because the same config is shared across prod, test, and dev, the next daily file in prod contained the new fields. The Morningstar intake detected ~359,987 instruments as updated (~200,000 incorrectly), and a bug in the intake logic **deleted any fund that had updated data from Morningstar** instead of ignoring updates where no actual field values changed. This cascaded into instrument master removing market data order books, making funds invisible on web.

## Timeline

| Time | Event |
|------|-------|
| 2025-03-05 | Wolf added additional fields to Morningstar download config in test |
| 2025-03-06 morning | Daily Morningstar file in prod contained new fields |
| 2025-03-06 morning | Intake detected ~359,987 instruments as updated (~200k incorrectly) |
| 2025-03-06 morning | Bug caused deletion of funds with updated data → orderbooks removed |
| 2025-03-06 | No funds visible on web |
| 2025-03-06 | Full-day incident response by entire Wolf team |
| 2025-03-06 | Resend fixed most funds, 9 required manual fix |

## Impact

- **Users affected:** All customers viewing or trading funds
- **Duration:** Full business day
- **Business impact:**
  - No funds visible on web
  - Deleted orderbooks across ~200,000 instruments
  - New `customerinstrumentid` values created (breaking references)
  - Swap orders impacted on buy legs
  - Monthly savings affected
- **SLA breached:** Unknown
- **Time spent:** Full day for everyone in Wolf

## Root Cause

Two compounding issues:

1. **Morningstar intake bug:** The intake logic deleted any fund that had updated data from Morningstar, even when the update contained no changes to fields actually used by the system. Adding new fields to the config triggered all ~200,000 funds to appear as "updated."

2. **Shared config across environments:** The same Morningstar config is used in prod, test, and dev. A change intended for test immediately affected prod on the next daily run.

## Resolution

1. Resend of fund data repaired the majority of affected instruments.
2. 9 instruments required manual fix (unclear why resend didn't work for these).
3. Impacted swap orders were handled by enabling customers to remove them.

## Cascading Incidents

- **Swap Orders impacted on Buy Legs** - alarms implemented in service that reads orderbooks to detect missing orderbooks.
- **Allocation request incident** - follow-up with Jensen.

## Stakeholders / Involved from Wolf

- Naiby Bhaskaran
- Niklas Notteman
- Peter Dahlsten
- Lars Mattsson
- Madelene Söderström (PO)

## Post-Incident Actions

### Fixed

| Issue | Jira | Assignee | Status |
|-------|------|----------|--------|
| Bug in Morningstar intake: deletes funds on any M* update even with no field changes | - | - | Fixed |
| Fix "instrument issue alert" to trigger during office hours | TP-27847 | Naiby Bhaskaran | Released |
| Ensure deleted/resurrected fund trading order books retain same identifier | TP-27841 | Lars Mattsson | Released |
| Retain old exchange traded trading order book ID when resurrected | TP-27845 | Niklas Notteman | Released |

### Follow-ups

- Monthly savings team implemented fix on their end (saving `customerinstrumentid`)
- Mutual Fund Order Execution: Introduce alarms on service that showed errors due to orderbook deletion
- Discussion with Instrument team on handling order book removal
- Enabled impacted swap orders to be removed by customers

### Unresolved

- Unclear why 9 funds were not repaired by the resend and needed manual fix

## Lessons Learned

- **Optional fields in Java - be careful.** The additional optional fields triggered the intake to treat records as updated.
- **BigQuery was invaluable.** Helped find old orderbook IDs and saved significant time during troubleshooting.
- **Shared config across environments is dangerous.** A test change immediately affected prod.
- **Too easy to make changes in prod databases.** No four-eye principle and no traceability for Postgres in NNX.
- **Lack of alerts** prevented the team from noticing the issue earlier.

## Related Cases

- **SC-002** (Missing Morningstar OnDemand EMT Data): Also a Morningstar intake issue but caused by vendor-side data gap rather than internal intake bug.
