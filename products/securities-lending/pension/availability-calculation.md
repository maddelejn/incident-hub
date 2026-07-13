---
type: product-knowledge
title: "Availability Calculation Algorithm"
source: "https://nordnetbank.atlassian.net/wiki/spaces/DE/pages/689308076"
last_updated: 2026-07-13
---

# Availability Calculation

**Source:** [Confluence](https://nordnetbank.atlassian.net/wiki/spaces/DE/pages/689308076)

## Definitions

| Term | Definition |
|------|-----------|
| **Loan definition** | Per pension company: rules for which accounts allow lending, which Abasec accounts for bookkeeping. Whitelists are associated to loan definitions. |
| **Whitelist item** | An instrument approved for lending, with rules that must be met. Same instrument can be whitelisted in multiple loan definitions. |
| **Buffer** | Percentage of availability held back as safety margin (per whitelist instrument) |
| **Recall buffer** | Hardcoded 10% held back to ensure shares can be recalled if needed |

## Algorithm (Simplified)

### Input (per whitelisted instrument)

1. All recalls made for this instrument
2. All positions for this instrument
3. All current loans booked on the Citi account
4. All block account trades and orders
5. All accounts that have opted out

### Calculation Steps

```
Step 1: Remove excluded positions
  └── Remove: BANKRUPTCY, DECEASED, TERMINATED accounts
  └── Remove: Frozen accounts (locktype F)
  └── Remove: Opted-out accounts

Step 2: Check minimum customer threshold
  └── Count participating customers
  └── Verify meets minimum set for the whitelist instrument

Step 3: Customer-adjusted availability
  └── Split positions into two lists:
       - Above max allowed % of total
       - Within limit
  └── Cap over-limit positions to max allowed quantity
       (max % × total quantity of all positions)
  └── Sum both lists = customerAdjustedAvailabilityQuantity

Step 4: Apply buffers
  └── Recall buffer (hardcoded 10%):
       recallBufferQty = customerAdjusted × 10%
  └── Whitelist buffer (per instrument):
       bufferAdjustedQty = customerAdjusted - (customerAdjusted × buffer%)
  └── Recall buffer adjusted:
       recallBufferAdjustedQty = customerAdjusted - (customerAdjusted × 10%)

Step 5: Calculate remaining availability
  └── totalRemaining = sum of all opted-in positions - currently on loan

Step 6: Determine what to report to Citi
  └── If recall needed → report 0
  └── Otherwise:
       qtyAvailableToLendOut = bufferAdjustedQty - currentlyOnLoan
       └── If negative → report 0
       └── If above max quantity cap → cap it
       └── Otherwise → report qtyAvailableToLendOut
```

### Key safeguards

- **Customer concentration limit:** No single customer can contribute more than a set % of total availability
- **Buffer:** Per-instrument buffer held back from what's reported as available
- **Recall buffer:** Additional 10% always held back to ensure recall capability
- **Max quantity cap:** Hard cap per whitelist instrument on total quantity
- **Minimum customers:** Minimum number of participating customers required before any lending

## How This Differs for Retail

In the retail setup (CSLA/Sharegain), availability is reported **per customer per position** rather than aggregated. Sharegain handles the matching and allocation on their side. The customer concentration limits and buffers would need to be reconsidered for the per-customer model.
