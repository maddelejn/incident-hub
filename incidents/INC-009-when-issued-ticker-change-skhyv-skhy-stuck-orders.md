---
id: INC-009
title: "When-Issued Ticker Change (SKHYV → SKHY) - Stuck Orders and Failed Deletions"
date: 2026-07-13
detected_at: "2026-07-13 14:45 CET"
resolved_at: ""
severity: P3
status: investigating
domain: trading
service: "order-management, instrument-intake, coresys"
team: "Trading Desk SE, Coresys, Team Wolf"
product_owner: "Team Wolf (Madde)"
category: integration
root_cause_category: config-change
on_call_responder: ""
retrospective_held: false
action_items_assigned: false
tags: [when-issued, ticker-change, nasdaq, stuck-orders, skhyv, skhy, ipo, corporate-action, millistream, us-instruments]
---

# INC-009: When-Issued Ticker Change (SKHYV → SKHY) - Stuck Orders and Failed Deletions

## Summary

SK Hynix Inc listed on Nasdaq with a When-Issued ticker `SKHYV` (trading July 10). On July 13, the ticker changed to regular-way `SKHY`. Customer orders placed under the old `SKHYV` ticker became stuck - they couldn't be deleted through normal channels (admin UI, Broken trading system) because the instrument was no longer found under the old identifier. Coresys involvement needed to update orders in the DB.

## Timeline

| Time (CET) | Event |
|-------------|-------|
| July 10 (Fri) | SKHYV (When-Issued) starts trading on Nasdaq. Millistream intake unblocked. |
| July 13 (Mon) | Ticker changes from SKHYV to SKHY (regular-way trading begins) |
| July 14 (Tue) | Official settlement date for when-issued period trades |
| 14:45 | Miro Niemi reports unable to delete orders on SKHYV - "instrument not found" |
| 14:47 | Sebbe confirms same issue - delete not possible since instrument not found |
| 14:49 | Simon Ljungberg confirms delete via Broken (trading system) also fails |
| 14:53 | Sebbe able to delete order after account refresh |
| 14:58 | Sebbe asks about deleting all local orders on SKHYV |
| 15:01 | Madde confirms SKHYV should be updated to SKHY |
| 15:09 | Sebbe confirms Coresys needed to update orders in DB |
| 15:11 | Madde shares context: exchanges don't auto-convert active orders over a weekend for WI→regular-way ticker changes |

## Impact

- **Users affected:** Customers with existing orders (market, limit, stop-limit, stop-trailing) on SKHYV
- **Duration:** Ongoing - orders stuck under old ticker
- **Business impact:** 6+ stop-limit and stop-trailing orders cannot reach the market under old ticker. Customers cannot delete their own orders.

## Root Cause

**When-Issued (WI) ticker handling gap.** When a stock trades on a When-Issued basis, Nasdaq uses a "V" suffix (SKHYV). When trading goes to regular-way, the V is dropped (SKHY). However:

1. Exchanges do **not** automatically convert active market/limit orders over a weekend for WI → regular-way ticker changes
2. Nordnet's system updated the instrument to the new ticker (SKHY) but did not migrate existing open orders from the old ticker (SKHYV)
3. Orders under the old ticker became orphaned - the instrument `SKHYV` no longer exists in the system, so orders can't be found, executed, or deleted

## Resolution (In Progress)

1. **Immediate:** Sebbe found that after account refresh, individual order deletion works
2. **Bulk fix:** Coresys to update remaining stuck orders in the database to point to SKHY
3. **6 stop-limit/stop-trailing orders:** Need decision - update to SKHY or delete entirely (since they would never reach the market under old ticker)

## What is When-Issued Trading?

When-Issued (WI) means shares are trading **before** the official settlement and closing of the public offering. It's conditional trading.

```
Timeline for this instrument:
  July 10 (Fri) ── SKHYV trades (When-Issued, "V" suffix)
  July 13 (Mon) ── SKHY trades (Regular-Way, V dropped)
  July 14 (Tue) ── Settlement date for WI trades
```

- **Nasdaq uses "V" suffix** for When-Issued instruments
- The position carries over automatically (SKHYV → SKHY) - customers don't need to do anything for their holdings
- But **active orders do NOT carry over** - this is the gap that caused this incident

## Stakeholders

| Role | Person | Action |
|------|--------|--------|
| Reported | Miro Niemi | Unable to delete order |
| Investigation | Sebbe | Identified instrument-not-found issue, found account refresh workaround |
| Trading Desk | Simon Ljungberg | Confirmed Broken deletion also fails |
| Fix execution | Coresys | DB update to migrate orders |
| PO / Context | Madde | WI ticker change context, confirmed SKHY is correct |

## The Right Approach: Automated Order Migration

**If we offer WI instruments, we must handle the full order lifecycle.** Cancelling customer orders because of an internal system gap is not acceptable - it's pushing our problem onto the customer. If we can't support it properly, we shouldn't offer it.

The correct solution: **automated order migration when the ticker changes from WI to regular-way.** The instrument is the same, the ISIN is the same - only the ticker symbol changes. Open orders should seamlessly carry over to the new ticker without the customer noticing.

Millistream already delivers the ticker change. The order management system needs to react to that event by migrating all open orders from the old ticker to the new one.

## Action Items

| Action | Owner | Status | Due Date |
|--------|-------|--------|----------|
| Coresys to migrate stuck SKHYV orders to SKHY in DB | Coresys / Sebbe | In progress | ASAP |
| Build automated order migration for ticker changes (WI and general) | Team Wolf / Madde | Open | TBD |
| Update US instrument listing runbook with WI handling | Madde | Open | TBD |

## Lessons Learned

- **If you let customers place orders, you own the lifecycle.** Cancelling orders because of an internal gap is unacceptable. Either support WI instruments fully (with automated order migration) or don't offer them at all.
- **This isn't just a WI problem.** Any ticker change (corporate action name changes, mergers) could orphan open orders the same way. Automated order migration on ticker changes is the systemic fix.
- **The Millistream intake worked fine** - it delivered the instrument under both tickers. The gap is in order management, not instrument intake.
- **Account refresh as a workaround** - refreshing the account allowed individual order deletion, but this doesn't scale for bulk operations.

## Related Cases

- **Runbook: New Listing in US Instruments** (runbooks/new-listing-us-instruments.md) - The SKHYV instrument was unblocked through the standard US instrument process. The runbook should be updated to flag When-Issued instruments as requiring special handling.
- **INC-001** (Stuck Instrument Blocking Order Routing) - Similar pattern of an instrument state change causing downstream order problems, though different root cause.
- **Runbook: Correcting Instrument Names** (runbooks/correcting-instrument-names.md) - Ticker/name changes via Millistream. Same underlying event (ticker change) but this incident shows the order management side isn't handling it.

## Runbook: When-Issued Ticker Changes (Interim Manual Process)

Until automated order migration is built, if this happens again:

1. **Identify** all open orders under the old WI ticker (with V suffix)
2. **Migrate ALL orders** to the new ticker via Coresys DB update - do NOT cancel customer orders
3. **Account refresh** can unblock individual order deletions as a temporary workaround
4. **Inform Trading Desk** so they can handle customer inquiries
5. **Verify** search and graph flows work under the new ticker
