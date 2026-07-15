---
id: INC-011
title: "Trading Power Unavailable Due to Stale Cache - Derivative Position Missing Instrument"
date: 2026-07-15
detected_at: "2026-07-15 ~10:55"
resolved_at: "2026-07-15 11:06"
severity: P3
status: resolved
service: "service-trading, admin-trading"
team: "Team ET, Team Pre-Trade"
product_owner: "Team Pre-Trade"
category: cache
root_cause_category: stale-cache
on_call_responder: "Sebbe (Team ET)"
tags: [trading-power, cache, service-trading, derivative, options, account-reload, admin-trading, known-issue, recurring]
---

# INC-011: Trading Power Unavailable Due to Stale Cache - Derivative Position Missing Instrument

## Summary

A customer (account 10153914) could not see their trading power or place orders. The root cause was a stale cache in `service-trading` - a derivative position (options contract OMXS306G31Y3200) was loaded into the cache before its instrument data was resolved, leaving the position with no linked instrument. This caused `service-trading` to fail when calculating trading power for the account. Resolved by reloading the account cache via `admin-trading`.

## Timeline

| Time | Event |
|------|-------|
| ~10:55 | Customer reports trading power not visible. Erik Sedenberg (Trading Desk) escalates to #area-securities-brokerage. |
| 10:58 | Initial assessment: derivative positions suspected (known issue pattern) |
| 10:58 | Nisse identifies one instance of `service-trading` has a broken cache |
| 11:00 | Sebbe refreshes the account via admin-trading account reload |
| 11:02 | Erik confirms trading power visible on client web |
| 11:06 | Customer confirms it's working |

## Impact

- **Users affected:** 1 customer (account 10153914)
- **Duration:** ~10 minutes (from escalation to resolution)
- **Business impact:** Customer could not trade. Worked around it by trading on a different account.

## Root Cause

`service-trading` caches account data including derivative positions. When a new derivative position (e.g. options roll, new structured product) is added to an account, the position can be loaded into the cache **before** its instrument data is fully resolved in the instrument service. This leaves the cached position with `instrumentId=0` (no linked instrument).

When `service-trading` later tries to calculate trading power, it fails because it can't resolve the instrument for the derivative position:

```
Failed to get data for tradingpower of account 10153914
(Missing instrument for derivative position
  (instrumentId=0, shortName='OMXS306G31Y3200', isin='SE0029410786', type=O, qty=-1.0))
```

The error is **instance-specific** - some `service-trading` instances may have a clean cache while others have the stale entry. This is why the trading power check in admin-trading succeeded on one attempt and failed on another (hitting different instances).

### Customer observation

The customer noted this is a recurring pattern:
> "Brukar hända när jag handlat eller lagts in nåt nytt värdepapper, nån TR eller terminsrull eller nåt."
> (Translation: "Usually happens when I've traded or a new instrument has been added, some tracker or futures roll or something.")

This confirms the race condition: new derivative positions trigger a cache load before instrument data is available.

## Resolution

Account reload via `admin-trading`:
1. Navigate to `admin-trading.prod.nordnet.se/#!/accountreload`
2. Enter the account number
3. This clears the account from the cache and forces a re-fetch of all positions with current instrument data

## Diagnostic Steps

When a customer reports missing trading power:

1. **Check trading power in admin-trading:** `admin-trading.prod.nordnet.se/#!/tradingpower` - enter account number
2. **Try multiple times** - the error may only appear on some `service-trading` instances
3. **Look for the error pattern:** `Failed to get data for tradingpower of account X (Missing instrument for derivative position...)`
4. **If the error matches:** Reload the account via `admin-trading.prod.nordnet.se/#!/accountreload`
5. **Verify** trading power is visible again and ask customer to confirm

## Known Issue Status

This is a **known recurring issue**. The Pre-Trade team has discussed automating the cache refresh but no solution is implemented yet.

**Potential fix (from Peter / Tea Milardović):** Automate the cache refresh instead of requiring a customer service call and manual admin-tool reset. Options:
- Auto-detect when a derivative position has `instrumentId=0` and trigger a re-fetch
- Proactively refresh the cache when new positions are added
- Add a TTL-based cache invalidation for derivative positions

## Stakeholders

| Role | Person/Team | Action |
|------|-------------|--------|
| Reporter | Erik Sedenberg (Trading Desk) | Escalated customer issue |
| Fix | Sebbe (Team ET) | Identified broken cache instance, reloaded account |
| Diagnosis | Nisse (Team ET) | Identified broken service-trading instance |
| Domain owner | Tea Milardović (Team Pre-Trade) | Explained root cause, confirmed known issue pattern |
| Follow-up | Peter | Raised question about automating the fix |

## Related

- **INC-011 is the same pattern** that was used to resolve a local orders issue the day before (2026-07-14), also fixed via account reload.
- **Admin Trading:** https://admin-trading.prod.nordnet.se
- **Service:** `service-trading` (on-prem, caches account/position data)
