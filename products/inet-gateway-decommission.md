---
type: product-knowledge
title: "INET Gateway Decommission - Moving to AIP for Deal Registration"
status: active
owning_teams: "Team ET (deal registration), Team Wolf (instrument creation)"
product_owner: "Madde (coordination)"
slack_channel: "#inet-gateway-decommission"
last_updated: 2026-07-14
---

# INET Gateway Decommission

## Overview

The INET Gateway is being decommissioned. Two parallel workstreams:

1. **Team ET:** Moving all deal (trade) registration from INET Gateway's WCF to the new Abasec AIP (REST API)
2. **Team Wolf:** Moving instrument creation for structured products from INET to a new flow

```
BEFORE:                              AFTER:
┌─────────────┐                     ┌─────────────────────────┐
│ INET Gateway│                     │ abasec-trade-feeder     │
│ (on-prem)   │                     │ (uses AIP REST API)     │
│             │──► Abasec (WCF)     │                         │──► Abasec (AIP)
│ - Deals     │                     └─────────────────────────┘
│ - TRS data  │
│ - Instruments│                    ┌─────────────────────────┐
└─────────────┘                     │ Wolf instrument creation│
                                    │ (new flow)              │──► Abasec
                                    └─────────────────────────┘
```

## Market Rollout Progress (Deal Registration)

All markets now migrated to AIP as of July 2026.

### Rollout order (earliest to latest)

| Phase | Markets | Market IDs | Status |
|-------|---------|------------|--------|
| 1 | Euronext Lisbon, Brussels, Dublin, Vienna, Madrid | 73, 75, 76, 81, 82 | Done |
| 2 | CHIX UK, Euronext Paris, Amsterdam, Milan, SIX Swiss | 70, 72, 74, 77, 83 | Done |
| 3 | Spotlight NO, Nasdaq OTC Foreign, NGM MTF ETP (DK/NO/FI/SE), NYSE American, Toronto, Canadian Venture | 68, 21, 63, 64, 67, 62, 18, 26, 25, 61 | Done |
| 4 | Spotlight SE, NGM, Oslo Børs | 52, various, 15 | Done |
| 5 | All remaining markets | All | Done |

**Final milestone:** "All market deals are now reported to abasec through AIP." (Tomas Unestad)

## Instrument Creation (Team Wolf)

### Live (July 14, 2026)

| Markets | Instrument Types |
|---------|-----------------|
| Nasdaq Stockholm (Market ID 11) | PTC (Trackers), KBU (Bull), KBE (Bear) |
| First North Sweden (Market ID 53) | PTC, KBU, KBE |
| First North Norway | PTC, KBU, KBE |

### Monitoring
- Track creations: https://instrument-admin.tools.prod.nntech.io/AbasecRequests
- New INET Sweden instruments have `.SE` in search name
- Currently only Xetra and Expand market instruments exist from this flow

### Coming next
- Mini futures
- Turbo warrants

## Key Technical Details

### AIP vs WCF Differences Found During Testing

| Field | INET Gateway (WCF) | New AIP | Decision |
|-------|-------------------|---------|----------|
| Trade time precision | Microsecond (6 digits) | Was millisecond (3 digits) | **Changed to microsecond** to match INET |
| marketPlaceTradeID format | e.g., `000351043` | Custom format | Aligned |
| orderSystemReference (Nasdaq OrderID) | Set from Nasdaq | Not set initially | Under investigation |
| userReference | `AOR001` (from FIX field 57) | Not set | Under investigation |
| order.type.code | `L` (Limit) | Not set | Under investigation |
| customerReference | Set | Not set (only account.id) | Being aligned |
| Currency on instrument | Not sent (Abasec sets from instrument) | Can be sent | Continue not sending - let Abasec set it |

### AIP Endpoint Details

| Item | Detail |
|------|--------|
| Service | `abasec-trade-feeder` |
| Endpoint | `POST /v1/marketplace/addMarketDeals` |
| Idempotency | Supported via UUID idempotency key (must be UUID format) |
| Batch behavior | **Transactional** - if any deal in batch fails, none are created |
| Response | Returns `entityID` which equals `mpdealid` in Abasec |

### Abasec Field Mapping Issues

The `orderReference` and `orderSystemReference` fields in the AIP don't map cleanly to the Abasec columns:

| AIP Field | Expected Abasec Column | Actual Behavior |
|-----------|----------------------|-----------------|
| `marketPlaceReference.orderReference` | OrderReferens | Sets BOTH OrderReferens AND Marknadsplats OrderID |
| `marketPlaceReference.orderSystemReference` | Marknadsplats OrderID | No effect |

This is a **mapping bug** reported to Stephanie Carp / Tieto.

### TRS Reporting Requirements

When decommissioning INET, ET must also handle TRS data that INET currently populates:

- **PreTradeWaivers** - translated from FIX message (e.g., LRGS, RFPT)
- **executionWithinFirm** - already sent
- **DecisionMakers** - not currently sent, needs adding
- **investmentDecision** - already sent
- **ShortSell flag** - needs sending
- **Marketplace timezone** - must be sent, otherwise Abasec uses its config timezone

### INET Gateway Changes Needed

INET Gateway needs modifications to allow migrating specific markets while keeping others on the old flow during the transition period.

## Key People

| Person | Team | Role |
|--------|------|------|
| Sebbe | ET | Lead developer on AIP migration |
| Felix Palmgren | ET | Developer, testing |
| Nisse | ET | Developer, prod credentials |
| Tomas Hägg | ET | Developer, market rollouts |
| Tomas Unestad | ET | Developer |
| Lars Mattsson | Wolf | Instrument creation flow |
| William Sörensen | Wolf / C&S | Abasec deal verification |
| Alexander Möller | C&S | Abasec deal verification |
| Max Rhodin Edlund | Regulatory Reporting | TRS/waiver verification |
| Rasmus Reivilä | C&S | Deal verification |
| Stephanie Carp | Tieto liaison | AIP bug fixes, Abasec queries |
| Sara Trygve | | Weekly sync coordination |
| Per Hörnfeldt | Postify | Post-trade involvement |
| Madde | Wolf/Shark PO | Coordination between ET and Wolf |

## Verification Process

For each market migration, ET provides sample trades (3-10 per market) comparing WCF vs AIP registration. C&S (William, Alexander, Rasmus) verify the deals look correct in Abasec.

CSV files with trade comparisons shared in Slack for each rollout phase.

## Related

- **SC-006** (INET instrument creation for Stockholm/First North) - the Wolf side of the decommission
- **Market reference** (reference/markets.md) - Market IDs referenced in rollout
