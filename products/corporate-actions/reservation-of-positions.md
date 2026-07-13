---
type: product-knowledge
title: "Reservation of Positions - Removing Fictitious Instruments Entirely"
status: active
source: "https://nordnetbank.atlassian.net/wiki/spaces/DE/pages/1442578784"
owning_team: "Team Shark"
product_owner: "Madelene Söderström"
engineer_manager: "Aleksandar Sekulic"
developers: "Hampus Holmertz, Joel Berglund, Ricard Kindfalk, Nedo Skobalj, Matti Lundgren"
platform: "NNX"
last_updated: 2026-07-13
---

# Reservation of Positions - Removing Fictitious Instruments Entirely

This document consolidates the project documentation for the Reservation of Positions initiative, which is Phase 2 of the broader reserve-and-protect programme within Corporate Actions. Phase 1 (Reserve Trading Power) is complete. Phase 2 (this project) tackles position reservations to eliminate fictitious instruments from the CA workflow.

---

## 1. Problem Statement

Corporate Actions has never had dedicated development resources to handle reservation of positions properly. The current workaround uses **fictitious instruments** to move customer positions during corporate action events. This approach is:

- **Super error-prone** -- fictitious instrument handling introduces numerous failure points across the processing chain.
- **Cross-departmental impact** -- errors from fictitious instruments affect many departments beyond Corporate Actions, creating ripple effects across operations, customer service, and accounting.
- **Bad customer UX** -- customers see fictitious instruments in their portfolio views, which is confusing and undermines trust.
- **A blocker for the Germany launch** -- fictitious instruments make German tax reporting impossible. The German tax framework cannot accommodate synthetic/fictitious instruments in its reporting model, making this a hard requirement for go-live.

This project removes the need for fictitious instruments by introducing proper position reservation and release functionality directly in the CA workflow. Application instruments (used for display purposes) remain and are not in scope for removal.

---

## 2. Scope

### Must Have

| Capability | Description |
|------------|-------------|
| Reserve positions | Ability to reserve customer positions during CA events, preventing customers from trading reserved shares |
| Release positions | Ability to release reserved positions back to customers once the CA event is settled |
| Bulk file upload in CA-admin | Upload multiple CSV files in a single operation to process batch reservations |

### Nice to Have

| Capability | Description |
|------------|-------------|
| Data to Dataland | Push reservation data to the Dataland platform for analytics and reporting |

### Not in Scope

| Item | Reason |
|------|--------|
| Automated flow within admin | Full automation of the reserve/release workflow inside CA-admin is deferred to later iterations |
| Fictitious instruments for display | Application instruments used for customer display purposes remain; only the position-moving use of fictitious instruments is eliminated |

---

## 3. Iterations

### Iteration 1 -- Germany Go-Live (Deadline: June 8, 2026)

**Scope:** Limited to essential event types required for the German market. Delivers full reserve and release functionality in CA-admin, plus bulk file upload.

**Key Risk:** There is no automated validation that Abasec trades are synced before release in Asset. A manual stop-gap popup has been added requiring operator confirmation before release proceeds.

> "Like driving a car without brakes -- you only want to drive it slowly on a private track (Germany)."

This metaphor captures the intentional constraint: Iteration 1 is safe enough for the limited, low-volume German market but is not yet hardened for full-scale rollout across all Nordnet markets.

**Deliverables:**

- Full reserve/release capability in CA-admin for supported event types
- Batch CSV upload for position reservations
- Manual confirmation popup as sync validation stop-gap
- Customer messaging (static text by event type, language by account country code)

### Iteration 2 -- MVP2

**Scope:** Extends the solution with improved operational tooling and automated safety checks.

**Planned deliverables:**

- Single file release (release positions from a specific file rather than all at once)
- Automated validation options (replace the manual confirmation popup with system checks that Abasec and Asset are in sync before allowing release)

---

## 4. Event Types and Priority

### Must-Have for Germany June 8

| Event Type | Eligibility | Notes |
|------------|-------------|-------|
| Tender Offer - Cash | Eligible, must-have | Core event type for German market |
| Tender Offer - Shares | Not strictly must-have | Similar handling to Cash variant; included due to low additional risk and rarity |
| Tender Offer - Combo | Must-have | Involves sell/removal with payments, cannot be deferred |
| Buy Back Offer | Eligible, must-have | Core event type for German market |
| Redemption | Eligible, must-have | Core event type for German market |
| Market Conversion | Low risk | Included due to low implementation risk |
| Share Class Conversion | Low risk | Included due to low implementation risk |
| Transformation Conversion | Low risk | Included due to low implementation risk |

### NOT for Germany Launch

| Event Type | Reason |
|------------|--------|
| Subscription Warrants | Needs more investigation before implementation |
| VPS Tender Offers | Norwegian market only, super manual process -- not relevant for Germany |
| Special Dividends | Likely not traded in fictitious instruments; low priority |
| Subscription Rights | German customers will not have "sell of TR" automation; Team Shark will rebuild the RightsDisposal service in fall 2026 |

---

## 5. Limitations

| Limitation | Impact | Mitigation |
|------------|--------|------------|
| **Instrument data latency** | Cannot reserve positions on newly created instruments within the first hour after creation | Operations must wait at least one hour after instrument creation before attempting reservation |
| **Restricted event scope** | Only essential German market event types are supported for June 8 | Phased rollout adds event types in subsequent iterations |
| **Validation gap** | No automated sync-check between Abasec and Asset before release | Manual confirmation popup requires operator to verify sync before proceeding |

---

## 6. Key Decisions

### German Customers and Subscription Rights

German customers cannot participate in "sell of TR" (subscription rights) until the RightsDisposal service is rebuilt by Team Shark in fall 2026. This is an accepted limitation for the Germany launch.

### Failed Position Bookings

When a position booking fails during batch upload:

1. The system displays an error for the failed row.
2. Operations deletes the failed row and re-uploads.
3. Single row retry is supported.
4. If the row keeps failing: release all successful reservations from that batch and delete the failing row entirely.

### Customer Messaging

- **Content:** Static text based on event type (not customizable per CA event in Iteration 1).
- **Language:** Determined by the account's country code.
- **Audit trail:** Stored in PING's service (ORAP).
- **Future:** Ability to customize messaging per CA event and preview messages before sending.

### Validate Against Reserved Positions When Customers Answer CA Events

Agreed that validation should occur, but deferred to MVP 2-3.

### Reservation Type

Only one reservation type/level is needed for this project. No need for multiple reservation categories.

### Nightly Reconciliation

A nightly reconciliation process runs between Abasec and ASSET, managed by the reservation-coordinator service. This catches any drift between the two systems overnight.

---

## 7. Technical Architecture

### Batch Reservation Flow

```
CSV file upload
    |
    v
CA-admin (NNX admin UI)
    |
    v
PubSub message published
    |
    +---> holdings.v1.asset-reserver-requests (reserve topic)
    |
    v
Assets-service (processes reservation)
    |
    v
holdings.v1.asset-reserver-responses (response topic)
    |
    v
CA-admin (updates status)
```

### Key Integration Points

| Component | Detail |
|-----------|--------|
| Reserve topic | `holdings.v1.asset-reserver-requests` |
| Response topic | `holdings.v1.asset-reserver-responses` |
| Communication | PubSub (async messaging) |
| Async API spec | [holdings-async-api on Backstage](https://backstage.nordnettech.se/catalog/holdings/api/holdings-async-api) |
| API symmetry | The same API is used for both reserving and releasing in Assets and Abasec |

### Reservation Coordinator

The `reservation-coordinator` service (owned by Team Hodor) handles:

- Nightly reconciliation between Abasec and ASSET
- Orchestrating the reserve/release lifecycle

---

## 8. Key Technical Findings

| Finding | Detail |
|---------|--------|
| **Selling reserved positions is a bug** | Tieto confirmed that being able to sell a reserved position in Abasec is a BUG, not a feature. A ticket has been filed for Tieto to fix this. |
| **TradingPower NOT affected** | Position reservations do not impact TradingPower calculations. Trading power and position reservations are independent mechanisms. |
| **Zero-position reservations** | Cannot reserve where a customer has 0 positions -- rejected by Abasec. However, Asset does allow this (inconsistency). |
| **Modifying reservations in Abasec** | To change a reservation value in Abasec, you must first unreserve and then re-reserve. In-place modification is not supported. |
| **ourReference field** | `ourReference` = broker who made the deal. Cannot be repurposed as a reservationId. |
| **accXref deprecated** | `accXref` has not been used for stocks since 2004. Not a viable identifier. |
| **externalVerNo** | `externalVerNo` (string) could potentially be used as a reference field, but functionality has not been built yet. |
| **Hodor can reserve in Asset** | Team Hodor's services have the capability to perform reservation operations in Asset. |

---

## 9. Cross-Team Dependencies

### Team Hodor

| Area | Detail |
|------|--------|
| reservation-coordinator service | Owns the service that orchestrates reservations and runs nightly reconciliation |
| Position display | Responsible for how reserved positions appear in web and app |
| acs-x-ref handling | Manages the cross-reference identifiers used in the reservation flow |

### Pre-Trade

| Area | Detail |
|------|--------|
| Trading power impact | Must confirm and handle any edge cases where reservations interact with trading power calculations |
| Customer cache invalidation | Race condition risk: if a customer is logged in when their positions are reserved, the cached state may be stale. Requires cache invalidation strategy. |

### ET (Execution & Trading)

Long-term, ET wants to be the source of truth for trades. This has implications for where reservation data ultimately lives and who owns it.

### Tieto (Abasec Vendor)

Must deliver a bug fix to prevent customers from selling reserved positions in Abasec. This is a critical dependency for the integrity of the reservation mechanism.

### Operations (Securities Brokerage Operations)

- Routine updates for the new workflow are extensive.
- Operations needs time to adapt workflows, update procedures, and train staff on the new CA-admin functionality.
- Cannot rush the rollout without giving ops time to prepare.

---

## 10. PO Recommendation -- Phased Approach

**Madde's documented recommendation: Phased rollout, NOT "Big Bang".**

### Rationale

| Reason | Detail |
|--------|--------|
| **Risk containment** | Easier to find and fix bugs iteratively than to withdraw an entire system that has replaced the old process across all markets |
| **Operational readiness** | Operations needs time for routine updates and workflow changes; a big bang would overwhelm them |
| **Preventing market exposure** | Without automated validations (deferred to Iteration 2), there is a risk of customers selling short if positions are released before Abasec/Asset are in sync |
| **Summer volume** | Summer trading volume is historically low -- there is no business pressure to rush a full-market rollout during this period |

The phased approach means Germany launches first with a constrained set of event types, giving the team time to validate the system under real but low-volume conditions before expanding to other markets.

---

## 11. Stakeholders

| Name | Role / Team |
|------|-------------|
| Martin Favre | Stakeholder |
| Hannes Kuchelmeister | Stakeholder |
| Josef Al-Shorji | Team Hodor |
| Anders Furby | SME / Stakeholder |
| Shayan Karimi | Stakeholder |
| Eva von Bell | Stakeholder |
| Max Wallinder | Stakeholder |
| Emelie Eriksson | Securities Brokerage Operations |

---

## Connection to Reserve Trading Power

This project is **Phase 2** of the broader reserve-and-protect programme:

| Phase | Project | Status | Description |
|-------|---------|--------|-------------|
| Phase 1 | Reserve Trading Power | **Done** | Prevents customers from using trading power derived from positions that are about to be affected by a CA event |
| Phase 2 | Reserve Positions (this project) | **Active** | Prevents customers from trading positions that are locked for a CA event, eliminating the need for fictitious instruments |

Phase 1 (Trading Power) was a prerequisite for Phase 2 (Positions). Together, they form the complete solution for protecting customer positions during corporate action processing without relying on fictitious instruments.
