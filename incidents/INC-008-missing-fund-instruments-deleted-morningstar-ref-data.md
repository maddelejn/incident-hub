---
id: INC-008
title: "Missing Fund Instruments due to Deleted Morningstar Reference Data"
date: 2026-07-10
detected_at: "2026-07-10 08:06 CET"
resolved_at: "2026-07-10"
severity: P1
status: resolved
service: "mstar-fund-overview-intake, fund-orders, instrument-search, partner-web"
team: "Team Compass Devs, Core Tech"
product_owner: "Mutual Funds, Partner Web"
category: integration
root_cause_category: dependency-failure
on_call_responder: ""
tags: [morningstar, intake, funds, deletion, reference-data, orderbooks, search, partner-web, mstar-fund-overview-intake, recurring, vendor, opsgenie]
opsgenie_alert: "#419500"
alert_name: "Legacy_instruments_not_mapped_to_a_Nordnet_X_market_data_order_book"
pr: "https://github.com/nordnet-private/mstar-fund-overview-intake/pull/191"
---

# INC-008: Missing Fund Instruments due to Deleted Morningstar Reference Data

## Summary

Morningstar explicitly deleted certain instruments from their daily delivery file (`Basic Reference Data_2026071004265188.csv`), causing 13 funds to disappear from the system. This led to failed fund trading orders, missing instruments in search, and broken partner-web interfaces. Resolved by pausing the automated scheduler, reverting to yesterday's valid data, and reprocessing. **This is the third occurrence of this pattern** (INC-006 March 6 2025, INC-007 March 24 2025).

## Timeline

| Time | Event |
|------|-------|
| 2026-07-10 | Morningstar daily file delivered with instruments explicitly deleted |
| 08:06 CET | Opsgenie P1 alert fired: `Legacy_instruments_not_mapped_to_a_Nordnet_X_market_data_order_book` - 13 legacy instruments not mapped to NNX market data order book. Routed to Team Wolf. (#419500) |
| TBD | Fund orders started failing, instruments missing from search |
| TBD | Investigation identified corrupted M* file as root cause |
| TBD | Automated scheduler paused via PR (#191) |
| TBD | Corrupted file/raw data deleted from intake DB (download ID: `bd66b002-5b7e-4de9-9aef-5fca3eeee031`) |
| TBD | Fell back to yesterday's valid data file |
| TBD | Port-forwarded to `mstar-fund-overview-intake`, triggered `processLatest` with `enforcedLatest = true` |
| TBD | Market 78 reloaded in search |
| TBD | Failed trading orders reprocessed successfully |
| TBD | Service restored |

## Impact

- **Users affected:** Customers trading affected funds, partner web users
- **Duration:** TBD
- **Business impact:** Failed fund trading orders, 13 funds missing from search/screener/partner-web
- **SLA breached:** Unknown

## Root Cause

Morningstar explicitly deleted certain instruments from their daily delivery file (`Basic Reference Data_2026071004265188.csv`). The downstream intake service (`mstar-fund-overview-intake`) processed these deletions, removing the instruments from the system and cascading into order failures and search/mapping breakage.

**This is a vendor-side issue.** Morningstar deleted the data at source.

## Resolution

1. **Paused automated scheduler** via PR to prevent downloading further corrupted files.
   - PR: https://github.com/nordnet-private/mstar-fund-overview-intake/pull/191
2. **Deleted corrupted raw data** from the local intake database via SQL:
   - Download ID: `bd66b002-5b7e-4de9-9aef-5fca3eeee031`
3. **Fell back to yesterday's valid data file.**
4. **Port-forwarded** to `mstar-fund-overview-intake` and triggered `processLatest` endpoint via Swagger with `enforcedLatest = true`.
5. **Reloaded Market 78** in search.
6. **Reprocessed failed trading orders** successfully.

## Runbook: Morningstar Fund Deletion Incident (Recurring)

This is the established procedure based on three occurrences (INC-006, INC-007, INC-008):

### Immediate response
1. **Pause the M* automated scheduler** - submit PR to disable downloads and prevent next run from making things worse.
2. **Delete today's corrupted data** from the intake database using the specific download ID.
3. **Fall back to yesterday's valid file** - this is the proven safe state.
4. **Trigger reprocessing:**
   - Port-forward to `mstar-fund-overview-intake`
   - Call `processLatest` endpoint via Swagger with `enforcedLatest = true`
5. **Reload affected markets** in search (e.g., Market 78).
6. **Reprocess failed orders** once instruments are restored.

### Vendor escalation
7. **Open a NEW Morningstar ticket** (never reply to old/closed ones).
8. Include: affected ISINs, the corrupted filename, authenticating user email.
9. **Do not re-enable the scheduler** until M* confirms the fix and Wolf verifies the next file.

### Verification
10. Confirm instruments visible in search, screener, instrument pages.
11. Confirm order placement works for affected funds.
12. Check partner web.
13. Check monthly savings impact.

## Stakeholders

| Role | Person/Team | Action |
|------|-------------|--------|
| Fix execution | Team Compass Devs / Core Tech | Paused scheduler, reverted data, reprocessed |
| Product Owner | Mutual Funds / Partner Web | Informed |
| External vendor | Morningstar | Deleted instruments from delivery file |
| Impacted | Customer Success (KAM) | Customer-facing impact |

## References

- **PR:** https://github.com/nordnet-private/mstar-fund-overview-intake/pull/191
- **Swagger:** `http://localhost:8080/swagger-ui/index.html#/MDS/processLatest`
- **Corrupted file:** `Basic Reference Data_2026071004265188.csv`
- **Download ID:** `bd66b002-5b7e-4de9-9aef-5fca3eeee031`

## Incident History - This Is a Pattern

| Incident | Date | Trigger | Funds affected | Root cause |
|----------|------|---------|---------------|------------|
| INC-006 | 2025-03-06 | Internal config change added new fields | ~200,000 | Intake bug + shared env config |
| INC-007 | 2025-03-24 | M* feed missing secID/performanceID | 33 | Vendor data quality |
| **INC-008** | **2026-07-10** | **M* explicitly deleted instruments from file** | **13** | **Vendor data quality** |

**Same outcome every time:** funds deleted, orderbooks removed, orders fail, customers impacted. The fix procedure is now well-established, but the prevention is still missing.

## Action Items

| Action | Owner | Status | Due Date |
|--------|-------|--------|----------|
| Build automated "yesterday's fallback" mechanism to eliminate manual DB deletion and PR overrides | Team Compass / Team Wolf | Open | TBD |
| Implement file-level corruption threshold: reject file if >N instruments would be deleted | Team Compass / Team Wolf | Open | TBD |
| Re-enable M* scheduler once vendor confirms fix | Team Compass | Pending | TBD |
| Escalate pattern to Morningstar - three incidents, request root cause analysis on their delivery pipeline | PO (Madde) | Open | TBD |

## Lessons Learned

- **Third time, same pattern.** The manual recovery procedure works and is getting faster, but this should not require manual engineering intervention every time.
- **Orderbook ID retention fix from INC-006 continues to pay off** - restored instruments keep the same IDs, limiting downstream cascade.
- **The "discard corrupted file" safeguard flagged in INC-007 was never built.** If a threshold check existed ("reject file if it would delete >X instruments"), all three incidents would have been prevented or auto-mitigated.
- **Automated fallback is the key investment.** The manual steps (pause scheduler, delete raw data, fall back to yesterday, trigger reprocess) are well-understood and should be automatable.

## Related Cases

- **INC-006** (Funds Deleted - March 6, 2025): First occurrence. ~200k funds, internal trigger (config change). Full day incident.
- **INC-007** (Missing Fund Data - March 24, 2025): Second occurrence. 33 funds, vendor trigger (missing secID). 3-hour incident.
- **SC-002** (Missing Morningstar OnDemand EMT Data): Different Morningstar intake path (EMT vs reference data) but same vendor dependency risk.
