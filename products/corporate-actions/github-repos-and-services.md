---
type: product-knowledge
title: "Corporate Actions - GitHub Repos and Service Map"
status: active
owning_team: "Team Shark"
product_owner: "Madelene Söderström"
last_updated: 2026-07-13
---

# Corporate Actions - GitHub Repos and Service Map

## Service Architecture Overview

```
ON-PREM                                    NNX (CLOUD)
┌──────────────────────────┐              ┌──────────────────────────────────────┐
│ corporate-action         │◄──PubSub────►│ ca-presubscription                  │
│ (Java, on-prem)          │              │ ca-manual-answer                    │
│ - Event creation/mgmt    │              │ event-response-handler              │
│ - Customer responses     │              │ ca-reserve-trading-power            │
│ - Compile file gen       │              │ ca-reserve-position                 │
│ - Abasec integration     │              │ ca-voluntary-event                  │
│ - TRS reporting          │              │                                     │
└──────────┬───────────────┘              │ ca-abasec-poller ──► ca-bookkeeper  │
           │                              │      ──► ca-sectras ──► Hodor       │
           │                              │                                     │
           ▼                              │ admin-corporate-action (TypeScript) │
      Abasec (ORAP)                       │ gcp-domain-corporate-action (Infra) │
                                          └──────────────────────────────────────┘
```

## All Repositories

### On-Prem Services

| Repo | Language | Description | Prod URL |
|------|----------|-------------|----------|
| [corporate-action](https://github.com/nordnet-private/corporate-action) | Java | Core CA service: event creation, customer responses, compile file generation, Abasec integration, TRS reporting | http://service-corporate-action.prod.nordnet.se |
| [ca-abasec-poller](https://github.com/nordnet-private/ca-abasec-poller) | Java | Polls Abasec database for corporate action event data (for German tax reporting pipeline) | On-prem service |

### NNX Cloud Services

| Repo | Language | Description | Purpose |
|------|----------|-------------|---------|
| [ca-presubscription](https://github.com/nordnet-private/ca-presubscription) | Java | Manages presubscription types on CA events | Replaces Excel for presubscriber markups |
| [ca-manual-answer](https://github.com/nordnet-private/ca-manual-answer) | Java | Manages manual answers on CA events | Replaces Excel for phone-in answers |
| [event-response-handler](https://github.com/nordnet-private/event-response-handler) | Java | Handles customer responses and validation for CA events | Core response processing |
| [ca-reserve-trading-power](https://github.com/nordnet-private/ca-reserve-trading-power) | Java | Reserves/releases trading power for CA events | Phase 1: Replace allocation fictitious instruments |
| [ca-reserve-position](https://github.com/nordnet-private/ca-reserve-position) | Java | Reserves/releases positions for CA events | Phase 2: Replace accept/tender fictitious instruments |
| [ca-voluntary-event](https://github.com/nordnet-private/ca-voluntary-event) | Java | Handles voluntary corporate action events | Voluntary event processing |
| [ca-sectras](https://github.com/nordnet-private/ca-sectras) | Java | Serves CA data to Team Hodor for Sectras German tax reporting | German tax CEG/TRDG interface |
| [ca-bookkeeper](https://github.com/nordnet-private/ca-bookkeeper) | Java | Transforms Abasec CA data into domain objects | Data transformation layer for Sectras pipeline |

### Admin & Infrastructure

| Repo | Language | Description | URL |
|------|----------|-------------|-----|
| [admin-corporate-action](https://github.com/nordnet-private/admin-corporate-action) | TypeScript | NNX admin frontend for all CA operations | https://corporate-action-admin.tools.prod.nntech.io |
| [gcp-domain-corporate-action](https://github.com/nordnet-private/gcp-domain-corporate-action) | HCL (Terraform) | GCP infrastructure for CA domain (PubSub topics, buckets, IAM) | - |

## Service Dependencies Map

### corporate-action (on-prem)

From prod config (`config/prod/service-corporate-action/service-corporate-action.properties`):

| Dependency | URL | Purpose |
|------------|-----|---------|
| service-trs | service-trs.prod.nordnet.se:80 | TRS reporting (decision maker) |
| service-core | service-core.prod.nordnet.se:80 | Customer/account data |
| service-nasa | service-nasa.prod.nordnet.se:80 | Universe data |
| service-instrument-suitability | service-instrument-suitability.prod.nordnet.se:80 | Suitability checks |
| service-kyc | service-kyc.prod.nordnet.se:80 | KYC validation |
| service-fx-approximator | service-fx-approximator.prod.nordnet.se:80 | FX rates |
| service-instrument | service-instrument.prod.nordnet.se | Instrument data |
| service-trading | service-trading.prod.nordnet.se | Trading integration |
| service-mc | service-mc.prod.nordnet.se | Messaging/communication |
| service-abatron (WCF) | service-abatron.prod.nordnet.se | Abasec loan/response booking |
| Abasec AIS | ABAPRDAIS1.pilen.nordnet.se:5505 | Direct Abasec integration |
| Mail | mail.prod.nordnet.se:25 | Email notifications |
| GCP PubSub | prod-cluster-25354 | Sync to NNX services |

### Key Scheduled Jobs (on-prem corporate-action)

| Job | Schedule | Description |
|-----|----------|-------------|
| event_response.reporting | 00:02 daily | Send responses to Abasec (2 min after midnight) |
| event_response_outbox.sync | Every 3 seconds | Sync responses to NNX |
| event_abasec_instrument_id_populator | 05:30 and 14:30 daily | Add AbasecInsId to active events |
| answer_compiler | 06:00 daily | Compile customer answers |
| delete_old_idempotency_ids | 01:00 daily | Cleanup |

### German Tax Reporting Pipeline

```
ca-abasec-poller (on-prem, polls Abasec DB)
    │
    ▼ PubSub
ca-bookkeeper (NNX, transforms to domain objects)
    │
    ▼ PubSub
ca-sectras (NNX, stores and serves to Hodor)
    │
    ▼ REST / PubSub
Team Hodor / transaction-gateway → Sectras (external vendor)
```

### Reserve & Protect Pipeline

```
Phase 1 - Trading Power:
  admin-corporate-action ──► ca-reserve-trading-power ──► Pre-Trades API

Phase 2 - Positions:
  admin-corporate-action ──► ca-reserve-position
    ──► PubSub: holdings.v1.asset-reserver-requests
    ◄── PubSub: holdings.v1.asset-reserver-responses
    ──► Assets-service (Hodor)
    ──► Abasec (via reservation-coordinator)
```

### Presubscription & Manual Answer Flow

```
Customer Service (via admin-corporate-action)
    │
    ├──► ca-presubscription ──PubSub──► corporate-action (on-prem)
    │                                    └──► compile file
    │
    └──► ca-manual-answer ──REST──► event-response-handler
                                    └──► corporate-action (on-prem)
                                         └──► compile file
```

## Key Config Values (from prod)

| Config | Value |
|--------|-------|
| Compile answers | 06:00 daily |
| Send responses to Abasec | 00:02 daily |
| Sync responses to NNX | Every 3 seconds |
| Mail (SE) | corporate.actions@nordnet.se |
| Mail (FI) | operations.fi@nordnet.fi |
| Danish 20% max percentage | 20% |
| Danish 20% max amount (2026) | 63,200 DKK |
| Danish 20% fallback amount | 63,200 DKK |
| GCP project | prod-cluster-25354 |
| Presubscription enrichment | Enabled |
| Virtual threads | Enabled (max 20 concurrent) |

## Quick Reference: Which Repo for Which Issue

| Issue Type | Check This Repo | Notes |
|-----------|----------------|-------|
| Event creation/modification problems | corporate-action | On-prem, Java |
| Customer can't answer a CA event | event-response-handler | NNX, handles validation |
| Presubscriber not showing in compile file | ca-presubscription | PubSub sync to on-prem |
| Manual answer not in compile file | ca-manual-answer | REST call to ERH |
| Trading power not reserved/released | ca-reserve-trading-power | Pre-Trades API integration |
| Position not reserved/released | ca-reserve-position | PubSub to Assets-service |
| German tax data missing for Hodor | ca-sectras | Check Sectras event status |
| German tax event not polled from Abasec | ca-abasec-poller | On-prem, polls Abasec DB |
| German tax data transformation issue | ca-bookkeeper | NNX, PubSub consumer/producer |
| Admin UI issues | admin-corporate-action | TypeScript, Next.js |
| Tender offer / voluntary event issues | ca-voluntary-event | NNX |
| Infrastructure (PubSub, buckets, IAM) | gcp-domain-corporate-action | Terraform |
| Compile file generation | corporate-action | On-prem, runs at 06:00 |
| TRS decision maker wrong | corporate-action | TRS Client via REST |
| Danish 20% rule blocking | corporate-action | Check config thresholds |

## Swagger / Admin URLs

| Service | URL |
|---------|-----|
| corporate-action (on-prem) | http://service-corporate-action.prod.nordnet.se/swagger-ui.html |
| CA Admin (NNX) | https://corporate-action-admin.tools.prod.nntech.io |
| German Tax Admin | https://corporate-action-admin.tools.prod.nntech.io/german-tax-reporting |
