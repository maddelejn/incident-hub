---
id: INC-010
title: "Trade Registration Delayed 1 Hour - Missing KBU/CRY Template for NGM MTF ETP Sweden (Market 61)"
date: 2026-07-14
detected_at: "2026-07-14 ~21:00"
resolved_at: "2026-07-14 22:10"
severity: P3
status: investigating
service: "abasec-trade-feeder, abasec-instrument-creator, abasec-instrument-feeder"
team: "Team ET, Team Wolf"
product_owner: "Team Wolf (Madde)"
category: configuration
root_cause_category: missing-template
on_call_responder: "William Sörensen"
tags: [trade-registration, abasec, instrument-creation, template-mapping, kbu, cry, crypto, ngm-mtf, market-61, retry-delay, abasec-trade-feeder, abasec-instrument-creator]
---

# INC-010: Trade Registration Delayed 1 Hour - Missing KBU/CRY Template for Market 61

## Summary

A trade on NGM MTF ETP Sweden (Market ID 61) for a crypto bull certificate (BULL XBT X15 VT3) could not be registered in Abasec because no instrument creation template existed for the combination: marketplace 61 + KBU + CRY + VPC. William Sörensen manually created the instrument at 21:10, but the trade was not registered until 22:10 - approximately 1 hour later. Three issues contributed: (1) a missing template mapping, (2) the instrument creator's Quartz schedule stopping at 20:20 with no runs until 22:10, and (3) the trade feeder's hourly retry cycle for failed trades.

## Timeline

| Time | Event |
|------|-------|
| 2026-06-05 06:00:32 | NNX instrument created (instrument exists in cloud) |
| 2026-07-14 ~21:00 | Trade executed on NGM MTF ETP Sweden. Trade feeder attempts to register in Abasec. |
| ~21:00 | Instrument creation fails: `AbasecAddInstrumentFailedException: No template found for marketplace: 61, kind: KBU, exerciseType: NONE, assetClass: CRY, priceType: MonetaryAmount, clearingPlace: VPC` |
| 21:10 | William Sörensen manually creates the Abasec instrument (insid registered at 21:10:18) |
| 21:20 | Instrument creator Quartz job **did NOT run** - last evening run was at 20:20, schedule gap until 22:10 (Lars finding) |
| **21:50** | **ResetTradesTask runs** (scheduled at :50 past each hour) - resets failed trade for retry |
| **21:51** | **Trade feeder retries - still fails** (instrument not synced because instrument creator hasn't run since 20:20) |
| **22:10** | **Instrument creator runs** (next scheduled run after 20:20 gap) - syncs the instrument |
| ~22:10 | ReportingTask (5-min cycle) picks up the trade - **successfully registered** |

## Impact

- **Users affected:** 1 trade (account 6090971)
- **Duration:** ~1 hour delay in trade registration
- **Business impact:** Trade not visible in Abasec for settlement processing until 22:10

## Root Cause

Two contributing factors:

### 1. Missing template mapping (known gap)

The `abasec-instrument-creator` database has no template for:

```
marketplace: 61 (NGM MTF ETP Sweden)
kind: KBU (Bull Certificate)
exerciseType: NONE
assetClass: CRY (Crypto)
priceType: MonetaryAmount
clearingPlace: VPC
```

**What exists for market 61 (V1_028):**
- `KTL` (Turbo warrant long) + CRY → template 411106
- `KTS` (Turbo warrant short) + CRY → template 411109

**What's missing for market 61:**
- `KBU` (Bull Certificate) + CRY ← **this trade**
- `KBE` (Bear Certificate) + CRY ← likely also missing

**Compare with market 11 (Stockholm) which has both (V1_025):**
- `KBU` + CRY → template 415332
- `KBE` + CRY → template 415329

**Repo:** [abasec-instrument-creator](https://github.com/nordnet-private/abasec-instrument-creator)
**Migration files:**
- `db/ora/aba_instr_create/migrations/V1_025__more_mappings_for_KBU_CRY_11.sql` (market 11 - has KBU+CRY)
- `db/ora/aba_instr_create/migrations/V1_028__mappings_for_CRY_market61.sql` (market 61 - only KTL+KTS, missing KBU+KBE)
- `db/ora/aba_instr_create/migrations/V1_029__mappings_for_CRY_market62_and_24.sql` (markets 62+24)

### 2. Trade feeder hourly retry cycle

When a trade fails in `abasec-trade-feeder`, it is not retried immediately. The retry mechanism works as follows:

**ReportingTask** (picks up new/reset trades):
- Runs every 5 minutes from 08:01 to 23:59
- Only processes trades in "ready" state - skips failed trades

**ResetTradesTask** (resets failed trades for retry):
- Runs every hour from 08:50 to 22:55 (at :50 past each hour)
- `threshold-last-reset-in-minutes: 60`
- Calls `tradeReporter.resetAllResend()` to mark failed trades as ready for retry

**Result:** When a trade fails and the instrument is then manually created, the trade sits in "failed" state for up to 1 hour until `ResetTradesTask` resets it. There is no manual "retry now" mechanism and no event-driven trigger when an instrument is created.

**Repo:** [abasec-trade-feeder](https://github.com/nordnet-private/abasec-trade-feeder)
**Config:** `config/prod/service-abasec-trade-feeder/service-abasec-trade-feeder.yml`

### 3. Instrument creator schedule gap (Lars's finding)

The `service-abasec-instrument-creator` Quartz schedule has a gap in the evening. The last run in the evening window was at **20:20**. The next run was not until **22:10** (the night schedule). This means:

- William created the instrument manually at 21:10
- But the instrument creator never ran between 20:20 and 22:10 to sync it
- The trade feeder's retry at 21:51 still failed because the instrument wasn't synced
- At 22:10 the instrument creator finally ran, synced the instrument, and the trade went through

**The schedule needs to be extended to run every hour until 22:00** to cover this gap. Lars is creating a ticket to fix this.

Current evening/night schedule:
```
Evening: 0 00,20,40 16-21 ? * MON-FRI *   → last run at ~20:40 (or 20:20?)
Night:   0 00,15 22 ? * MON-FRI *         → next run at 22:00
```

The gap between ~20:20/20:40 and 22:00 leaves up to ~1.5 hours where no instrument sync runs.

## Trade Details

| Field | Value |
|-------|-------|
| Trade ID | 64335677 |
| Account | 6090971 |
| Side | BUY |
| Order ID | 664795950 |
| Instrument | BULL XBT X15 VT3 |
| Trading code | MYT2 |
| Market | NGM MTF ETP Sweden (Market ID 61) |
| Instrument ID (NNX) | 8b385aca-416c-4029-a4ed-7342a722bd16 |
| Currency | SEK |
| Custody | Euroclear Sweden (973475) |

## Instrument Creation Services (Team Wolf)

### Service architecture for instrument creation

```
                    service-abasec-instrument-creator (on-prem)
                    ┌─────────────────────────────────────────┐
                    │ Quartz scheduler:                       │
                    │   Daily:   10:01-15:01 (hourly)         │
                    │   Evening: 16:00-21:40 (every 20 min)   │
                    │   Night:   22:00, 22:15                 │
                    │                                         │
                    │ Polls service-instrument for warrants   │
                    │ Creates instruments via Abatron → Abasec│
                    │                                         │
                    │ Template mappings in Oracle DB           │
                    │ (TEMPLATE_MAPPINGS table)                │
                    └─────────────┬───────────────────────────┘
                                  │
                                  ▼
                    service-abasec-trade-feeder (on-prem)
                    ┌─────────────────────────────────────────┐
                    │ ReportingTask: every 5 min (08:01-23:59)│
                    │ ResetTradesTask: every hour (:50)        │
                    │                                         │
                    │ Polls trade.trades for new trades        │
                    │ Calls instrument-creator to create instr │
                    │ Sends trades to Abasec via AIP           │
                    └─────────────────────────────────────────┘

NNX cloud:
                    abasec-instrument-feeder
                    ┌─────────────────────────────────────────┐
                    │ Consumes PubSub instrument/trade events  │
                    │ Creates Abasec instruments via AIP proxy │
                    │ Tracks "waiting-for-abasec" (10 min)     │
                    └─────────────────────────────────────────┘
```

### Key GitHub repos

| Repo | Description | Role in this incident |
|------|-------------|----------------------|
| [abasec-instrument-creator](https://github.com/nordnet-private/abasec-instrument-creator) | Template-based instrument creation in Abasec (on-prem) | Missing KBU+CRY template for market 61 |
| [abasec-trade-feeder](https://github.com/nordnet-private/abasec-trade-feeder) | Trade registration to Abasec (on-prem) | Hourly retry cycle delayed trade registration |
| [abasec-instrument-feeder](https://github.com/nordnet-private/abasec-instrument-feeder) | NNX cloud instrument creation via AIP | Alternative path for instrument creation |
| [abaapi-instrument-proxy](https://github.com/nordnet-private/abaapi-instrument-proxy) | Proxy from NNX to Abasec AIP | Used by abasec-instrument-feeder |
| [nnx-instrument-intake](https://github.com/nordnet-private/nnx-instrument-intake) | NNX instrument intake | Instrument existed here since June 5 |
| [xetra-instrument-intake](https://github.com/nordnet-private/xetra-instrument-intake) | Xetra instrument intake via Millistream | Not involved (this is NGM) |
| [instrument-admin](https://github.com/nordnet-private/instrument-admin) | Admin tool for instruments | Used by William to verify instrument |

## Resolution

William manually created the instrument at 21:10. The trade was eventually registered at 22:10 after the hourly `ResetTradesTask` reset the failed trade.

## Action Items

| # | Action | Owner | Status | Details |
|---|--------|-------|--------|---------|
| 1 | **Add KBU+CRY and KBE+CRY templates for marketplace 61** | Team Wolf / Team ET | Open | New migration in `abasec-instrument-creator` similar to V1_025 (market 11). Need to confirm correct `tmpl_insid` values. |
| 2 | **Extend instrument creator schedule to cover evening gap** | Lars Mattsson (Team Wolf) | Ticket being created | The Quartz schedule stops running at ~20:20/20:40 and doesn't resume until 22:00. Needs to run every hour until 22:00. Lars is creating a ticket and aiming to fix this week. |
| 3 | **Evaluate trade feeder retry interval** | Team ET | Open | The 1-hour `ResetTradesTask` interval means manual instrument creation has no immediate effect on failed trades. Options: shorter interval, manual retry endpoint, or event-driven retry when instrument is created. |
| 4 | **Audit all market 61 template mappings** | Team Wolf | Open | Check if other instrument type + asset class combinations are missing for market 61 compared to markets 11, 53, 24, 62. |

## Stakeholders

| Role | Person/Team | Action |
|------|-------------|--------|
| Reporter | William Sörensen (Team Wolf / C&S) | Reported delayed trade registration, manually created instrument |
| Investigation | Lars Mattsson (Team Wolf) | Investigating instrument sync timing |
| Investigation | Madde (Team Wolf PO) | Traced root cause to missing template and trade feeder retry logic |
| Owning team | Team ET | Owns abasec-trade-feeder and ResetTradesTask |

## Related

- **SC-006** (INET Gateway Decommission): Instrument creation flow for Stockholm/First North - same instrument types (PTC, KBU, KBE) but different markets.
- **INET Gateway Decommission** (`products/inet-gateway-decommission.md`): Broader context for instrument creation migration.
- **Instrument Admin:** https://instrument-admin.tools.prod.nntech.io/Instruments?query=BULL%20XBT%20X15%20VT3
- **Order History:** https://admin-trading.prod.nordnet.se/#!/orderhistory?orderId=664795950
