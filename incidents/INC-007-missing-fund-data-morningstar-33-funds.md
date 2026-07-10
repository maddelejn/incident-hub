---
id: INC-007
title: "Missing Fund Data in Morningstar Feed for 33 Funds"
date: 2025-03-24
detected_at: "2025-03-24 07:00 UTC"
resolved_at: "2025-03-24 10:00 UTC"
severity: P1
status: resolved
service: "morningstar-intake, instrument-master, fund-orderbooks, fund-orders, monthly-savings"
team: "Team Wolf"
product_owner: "Team Wolf (Madde)"
category: integration
root_cause_category: dependency-failure
on_call_responder: ""
tags: [morningstar, intake, funds, orderbooks, deletion, secid, performanceid, fund-orders, monthly-savings, partner-web, screener, search, instrument-page]
---

# INC-007: Missing Fund Data in Morningstar Feed for 33 Funds

## Summary

A Morningstar data feed was missing key identifiers (secID, performanceID) for 33 funds, causing them to be deleted from the system. Funds disappeared from search, screener, and instrument pages. Order entry stopped working, ~900 fund orders were rejected between 07:00-10:06, and monthly savings plans were impacted. Resolved by manually reverting to previous day's data.

## Timeline

| Time (UTC) | Event |
|------------|-------|
| 07:00 | Instruments deleted due to missing identifiers in M* feed |
| 08:29 | Ping in #disco channel, investigation started |
| 10:00 | All instruments manually updated with previous day's data |
| 10:06 | Order rejections stopped |
| March 24 | M* download disabled while vendor investigated |
| March 24 | M* believed issue fixed; Wolf verified no missing secID/ISIN combos in file |
| March 24 | File manually triggered after verification |

## Impact

- **Users affected:** All customers trading or viewing the 33 affected funds
- **Duration:** ~3 hours (07:00 - 10:00)
- **Business impact:**
  - 33 funds deleted from web, search, screener, instrument pages
  - ~900 rejected fund orders between 07:00-10:06
  - Monthly savings plans affected (activation impact TBD - Viktor)
  - Funds greyed out on account page
  - Partner web affected
  - Banners couldn't be shown because instrument pages were gone - unable to communicate to customers
- **SLA breached:** Unknown
- **Time spent:** Full day

## Root Cause

Morningstar's data feed was missing key identifiers (`secID`, `performanceID`) for 33 funds. The Morningstar intake service treated the missing identifiers as a signal to delete the funds, which cascaded into orderbook removal, search/screener removal, and order rejection.

This is a **vendor-side data quality issue** combined with an **overly aggressive internal deletion logic** - the intake should not delete instruments just because vendor identifiers are missing from a single feed.

## Resolution

1. **Immediate fix:** Manually updated the database to revert the 33 instruments to previous day's data and reprocessed them.
2. **Containment:** Disabled M* download while vendor investigated.
3. **Vendor fix:** Morningstar investigated and believed they fixed the issue by March 24. Wolf verified the file had no missing secID/ISIN combinations before manually triggering the download.

## Improvement Over INC-006

The orderbook ID retention fix from INC-006 (TP-27841, TP-27845) **worked** - when instruments were restored, they got back the same orderbook IDs. This meant other downstream teams were much less impacted compared to the March 6 incident.

## Stakeholders / Involved from Wolf

- Naiby Bhaskaran
- Peter Dahlsten
- Lars Mattsson
- Madelene Söderström (PO)

## Post-Incident Actions

| Action | Owner | Status |
|--------|-------|--------|
| Investigate M* side to find full scope of missing data | Morningstar | Done |
| Investigate "discard corrupted file" logic in M* intake | Team Wolf | Open |
| Check if FundOffering data can prevent deletion when M* data is lost | Team Wolf | Open |
| Check with Disco/ICC about keeping instrument pages visible for 24h when instruments are deleted (so banners can still show) | Madde | Open |
| Add M* contact persons to Daniel and Johan, check other vendors too | Madde | Open |
| Check monthly savings activation impact | Viktor | Open |
| Check if 900 rejections were monthly savings or regular fund orders | Ben | Open |
| Check partner web impact | Jensen | Open |

## Lessons Learned

- **INC-006 fix paid off:** The orderbook ID retention fix (TP-27841, TP-27845) from the March 6 incident meant restored instruments kept the same IDs, significantly reducing downstream impact.
- **Deletion logic is still too aggressive:** Even after INC-006, the intake still deletes instruments when vendor data is missing/corrupted. A "discard corrupted file" safeguard in the intake could prevent this class of incident entirely.
- **Communication gap:** When instrument pages are deleted, banners can't be shown. Customers couldn't be informed about the issue through the normal channel.
- **Manual fix doesn't scale:** Reverting 33 funds manually worked, but if the number had been larger (like the ~200k in INC-006), this approach wouldn't have been viable. A "reject file if corruption threshold exceeded" approach would be faster.
- **Vendor dependency:** Morningstar data quality issues can directly delete instruments and block trading. No safeguard exists between the vendor feed and destructive internal actions.

## Related Cases

- **INC-006** (Funds Deleted - Morningstar Intake Bug, March 6): Same incident pattern - Morningstar intake causes fund deletion and orderbook removal. INC-006 was caused by internal config change triggering the bug; INC-007 was caused by vendor-side missing data. Same deletion logic, same cascading impact. Orderbook ID fix from INC-006 reduced INC-007 impact.
- **SC-002** (Missing Morningstar OnDemand EMT Data): Also a Morningstar data gap but for EMT data, not instrument identifiers.
