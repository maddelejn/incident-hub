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

## Critical Finding: Orders on SKHYV Do NOT Reserve Cash

Simon Ljungberg discovered that **buy orders placed on SKHYV are not reserving cash** (trading power). This creates a double-execution risk:

```
Scenario:
  1. Customer places buy order on SKHYV (Friday) - no cash reserved
  2. Customer places buy order on SKHY (Monday) - thinking old order is dead
  3. If we migrate SKHYV orders to SKHY → BOTH execute
  4. Customer is overdrawn
```

This means **blindly migrating orders is dangerous.** It's not just a ticker mapping issue - it's a trading power / cash reservation gap that creates financial exposure.

## Decision Needed

| Option | Risk | Customer Impact |
|--------|------|-----------------|
| **Cancel SKHYV orders + notify customers** | Low financial risk | Customers who haven't re-ordered on SKHY miss their trade. Acceptable if communicated properly and promptly. |
| **Migrate SKHYV orders to SKHY** | High - double execution risk if customer already re-ordered on SKHY | Seamless for customers who didn't re-order, dangerous for those who did |
| **Migrate only if customer has no existing SKHY order** | Medium - requires per-customer check | Best of both worlds but operationally complex for a manual DB fix |

**For this incident:** Given the cash reservation gap and the double-execution risk, cancelling with customer notification is the safer choice. However, this highlights that **if Nordnet offers WI instruments, the system must properly handle the full lifecycle** - including cash reservation on WI orders and automated migration on ticker change. Without that, offering WI instruments creates a known gap.

**Post-vacation action (Björn Alenvik):** Evaluate whether to offer When-Issued instruments at all, or invest in proper WI lifecycle handling (cash reservation + automated ticker migration).

## Action Items

| Action | Owner | Status | Due Date |
|--------|-------|--------|----------|
| Decide: cancel or migrate stuck SKHYV orders (considering double-execution risk) | Madde / Trading Desk / Sebbe | In progress | ASAP |
| If cancelling: notify affected customers | Trading Desk / Customer Service | Pending decision | ASAP |
| Post-vacation: evaluate WI instrument offering and proper lifecycle handling | Björn Alenvik / Madde | Open | August |
| Investigate why WI orders don't reserve cash (trading power) | Team Wolf | Open | TBD |
| Update US instrument listing runbook with WI handling | Madde | Open | TBD |

## Lessons Learned

- **The real problem is deeper than ticker mapping.** Orders on the WI ticker don't reserve cash. Even if the ticker migration worked perfectly, this cash reservation gap would still create risk.
- **Normal ticker changes work fine.** The system handles regular corporate action ticker changes correctly. WI instruments break because something in the order/instrument setup treats them differently (likely the V-suffix instrument doesn't have proper trading power integration).
- **Same ISIN, same instrument ID.** Confirmed that SKHYV and SKHY share the same ISIN - only short name and symbol changed. The system should have handled this like any other ticker change, but didn't.
- **The Millistream intake worked fine** - it delivered the instrument under both tickers. The gap is in order management and cash reservation, not instrument intake.
- **Account refresh as a workaround** - refreshing the account allowed individual order deletion, but this doesn't scale for bulk operations.

## Related Cases

- **Runbook: New Listing in US Instruments** (runbooks/new-listing-us-instruments.md) - The SKHYV instrument was unblocked through the standard US instrument process. The runbook should be updated to flag When-Issued instruments as requiring special handling.
- **INC-001** (Stuck Instrument Blocking Order Routing) - Similar pattern of an instrument state change causing downstream order problems, though different root cause.
- **Runbook: Correcting Instrument Names** (runbooks/correcting-instrument-names.md) - Ticker/name changes via Millistream. Same underlying event (ticker change) but this incident shows the order management side isn't handling it.

## Runbook: When-Issued Ticker Changes (Interim Manual Process)

Until proper WI lifecycle handling is built, if this happens again:

1. **Identify** all open orders under the old WI ticker (with V suffix)
2. **Check for double-execution risk:** For each customer with a WI order, check if they also have an order on the new ticker
3. **If customer has orders on BOTH tickers:** Cancel the WI order and notify the customer
4. **If customer only has the WI order:** Migrate to new ticker via Coresys DB update (preferred) or cancel and notify
5. **Verify** that WI orders are actually reserving cash - if not, DO NOT blindly migrate (overdraft risk)
6. **Account refresh** can unblock individual order deletions as a temporary workaround
7. **Inform Trading Desk** so they can handle customer inquiries
8. **Verify** search and graph flows work under the new ticker
