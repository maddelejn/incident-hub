---
type: investigation
title: "Investigation: Remove or Replace Market Cap Dependency in Intraday Shorting"
status: open
product_owner: "Madelene Söderström"
stakeholders: "Compliance, Risk, Team Shark, Factset (vendor)"
priority: medium
last_updated: 2026-07-13
---

# Investigation: Market Cap Dependency in Intraday Shorting

## Problem Statement

The intraday shorting logic in Securities Borrowing depends on market cap data from **Factset** to determine whether S/SS category instruments qualify for intraday short selling. Factset delivers **incorrect or missing market cap data approximately once per month**, consistently reporting **too-low or zero market cap values**. This causes instruments that should be available for intraday shorting to be **incorrectly blocked**, leading to:

- Customers unable to intraday short instruments that should be allowed
- Support/broker desk inquiries
- Manual investigation time each occurrence
- Lost trading revenue

## Current Logic

Market cap is only checked in **Path B** of intraday shorting (S/SS category instruments):

```
Intraday shorting Path B (S/SS instruments):
  1. Instrument in SEB availability (S or SS category)     ✓
  2. Successfully located from SEB                          ✓
  3. Not in exclusion list                                  ✓
  4. Located market value >= threshold                      ✓
  5. Market cap >= threshold  ← FACTSET DATA USED HERE      ✗ (blocks when data is wrong)
```

### Market Cap Thresholds (from prod config)

| Country | Threshold |
|---------|-----------|
| SE | 100,000,000 SEK |
| NO | 100,000,000 NOK |
| DK | 100,000,000 DKK |
| FI | 10,000,000 EUR |

### Error pattern

- **Frequency:** ~1x per month
- **Error direction:** Too-low or zero market cap (never too-high)
- **Impact:** Instruments incorrectly blocked from intraday shorting
- **No risk exposure:** Errors are conservative (blocking, not allowing)

## Options Under Evaluation

### Option A: Remove market cap check entirely

**Requires:** Compliance approval

**Argument for compliance:**
- The check only applies to Path B (S/SS instruments), which are already a small subset
- Four other safeguards remain in place:
  1. SEB's own risk assessment (they decide what goes in S/SS)
  2. Located market value threshold (100K SEK) - can't short without sufficient located volume
  3. Exclusion list (manual broker override)
  4. SEB can revoke locates at any time
- Current vendor data is causing instruments to be incorrectly blocked ~12x/year
- Error direction is always conservative (too-low), so no actual risk exposure has occurred from the bad data
- Removing one unreliable check while keeping four reliable safeguards arguably improves the system

**Risk:** Removes a defense-in-depth layer. A micro-cap instrument could theoretically be intraday shortable if SEB has it in S/SS.

### Option B: Replace Factset with internal market cap calculation

**Approach:** Calculate market cap as `outstanding shares × current price` using existing internal data.

**Blocker:** Madde's position (and correct architectural stance) is that **market cap should not be calculated independently by the Securities Borrowing team**. If Nordnet needs market cap as a data point, it should be:
- Owned by the team responsible for market data / instrument data
- Available as a shared service/data point for any team that needs it
- Not duplicated with per-team calculations

**Next step:** Investigate whether a central team (e.g., instrument data / Team Wolf / market data team) could own and expose a market cap data point. This would benefit not just Securities Borrowing but any other Nordnet system that needs market cap.

### Option C: Replace Factset with another vendor

**Approach:** Switch market cap source to a more reliable vendor (e.g., SIX, Bloomberg, Morningstar).

**Consideration:** Adding another vendor relationship and data integration just for one monthly data quality issue may be disproportionate effort.

### Option D: Raise located market value threshold as proxy

**Approach:** Keep market cap check removed, but increase the located market value threshold for S/SS from 100K SEK to 500K-1M SEK. This effectively filters out micro-caps through a different mechanism.

**Pro:** No vendor dependency needed.
**Con:** Threshold is a blunt instrument - might be too restrictive for some legitimate S/SS instruments.

## Recommended Approach

**Short term: Option A (remove market cap check) with compliance approval.**

Build a business case showing:
- 12 incidents/year of incorrect blocking
- Error direction is always conservative (no risk events)
- Four other safeguards remain
- Vendor data quality is the root cause, not the logic itself

**Long term: Option B (centralized market cap service) done properly.**

If compliance insists on keeping a market cap check, push for a centralized market cap data point owned by the appropriate team, not a Securities Borrowing-specific calculation. This is the right architectural approach and would serve other teams too.

## Business Case for Compliance

### Current situation

| Metric | Value |
|--------|-------|
| Incidents per year | ~12 (once/month) |
| Impact per incident | Instruments incorrectly blocked from intraday shorting |
| Risk exposure from errors | Zero (errors are always conservative - blocking, never allowing) |
| Vendor | Factset |
| Error type | Too-low or missing market cap |

### What we're asking

Remove the market cap threshold check from intraday shorting eligibility for S/SS instruments, while retaining all other safeguards.

### Safeguards that remain after removal

| Safeguard | Description | Effective? |
|-----------|-------------|-----------|
| **SEB risk assessment** | SEB decides which instruments go in S/SS categories. They won't offer instruments they can't cover. | Yes - external counterparty risk management |
| **Located market value threshold** | Instrument must have ≥100K SEK located value. Can't short without sufficient locatable shares. | Yes - filters out instruments with tiny volume |
| **Exclusion list** | Brokers can manually block any instrument from shorting at any time. | Yes - human override |
| **SEB locate revocation** | SEB can revoke locates at any time if they become uncomfortable with the exposure. | Yes - real-time external control |
| **Hourly recalculation** | UpdateShortSellingStatusForInstrumentsTask runs hourly - exposure window limited. | Yes - self-correcting |

### What we gain

- Eliminate ~12 false-blocking incidents per year
- Remove dependency on unreliable Factset market cap data for this use case
- Reduce broker desk investigation time
- Increase customer ability to trade instruments that are legitimately shortable

### What we don't lose

- No change to auto covered shorting (Path A) - unaffected
- No change to the exclusion list - still available for manual blocking
- No reduction in SEB's own risk controls
- No impact on any other system that uses Factset market cap data

## Open Actions

| Action | Owner | Status |
|--------|-------|--------|
| Draft compliance proposal based on this business case | Madde | Open |
| Quantify lost revenue from false blocks (12 incidents/year) | Madde / Trading Desk | Open |
| Investigate if any central team could own market cap as a shared data point | Madde | Open |
| Get compliance sign-off | Compliance | Pending proposal |

## Related

- **Product doc:** products/securities-lending/securities-borrowing.md (Section 3: Short Selling Criteria)
- **Prod config:** securities-borrowing `shortSellingCriteria.intradayShortingCriteria.minimumMarketCapValues`
- **Troubleshooting:** runbooks/securities-financing-troubleshooting.md
