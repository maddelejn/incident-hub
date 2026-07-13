---
type: product-knowledge
title: "Retail Lending - Initial Technical Assessment"
source: "https://nordnetbank.atlassian.net/wiki/spaces/DE/pages/1601274035"
date: 2026-03-06
last_updated: 2026-07-13
---

# Retail Lending - Initial Technical Assessment

**Source:** [Confluence](https://nordnetbank.atlassian.net/wiki/spaces/DE/pages/1601274035)

## Why a New System is Needed

The current on-prem `service-securities-lending` is **not compatible** with Sharegain's retail requirements:

1. **Availability:** Pension reports aggregated availability. Retail needs per-customer, per-position with real-time intraday updates.
2. **Compensation:** Pension calculates daily compensation internally. Retail gets monthly compensation reports from Sharegain.
3. **Technology:** Current system is on-prem Kotlin - Nordnet strategy is to move to NNX (cloud).

**Decision:** Build a completely new NNX system that supports both retail and pension lending (eventual migration).

## Key Differences: Pension vs Retail

| Feature | Pension (current) | Retail (new) |
|---------|-------------------|--------------|
| **Participation** | Opt-in by default, manual opt-out | Opt-out by default, customer must actively opt-in via web |
| **Availability** | Aggregated per insurance company | Per customer, per position, with real-time updates |
| **Compensation** | Calculated daily by Nordnet | Calculated and reported monthly by Sharegain |
| **Anonymisation** | N/A (aggregate) | Required - map real accounts to anonymised IDs |

## Key Components to Build

### 1. Opt-In Handling
- Backend to store opt-in/opt-out state per customer
- **Simple mode:** Customer is "in" or "out"
- **Advanced mode:** Opt-in but exclude specific instruments
- Web support required (app support TBD)
- May need customer agreement

### 2. Whitelist Handling
- Similar to existing pension whitelist
- May need different settings (buffers, etc.) for retail vs pension

### 3. Availability Calculation
- Daily snapshot + real-time intraday updates to Sharegain
- Per-account reporting (anonymised)
- Need account ID mapping (real ↔ anonymised)

### 4. Compensation Handling
- Sharegain calculates and reports monthly (not Nordnet)
- Adapt existing lending-compensation NNX service for new format

### 5. Loan Bookings in Abasec
- Current path: service-abatron (not reachable from cloud)
- Need: New cloud component to book loans in Abasec via REST (WCF → REST migration)
- **Bonus:** This component is also needed by Securities Borrowing in the future

### 6. Expanded Admin in NNX
- On-prem admin will NOT be reused
- Expand existing NNX admin (currently only handles compensation payout)
- Needs full admin functionality: whitelist management, reconciliation, loan overview, etc.

## Not Yet Addressed

- Reconciliation
- SFTP integration
- Statistics
- Allowed Borrowers list
