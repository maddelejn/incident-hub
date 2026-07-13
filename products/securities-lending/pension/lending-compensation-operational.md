---
type: product-knowledge
title: "Lending Compensation - Operational Document"
status: active
source: "https://nordnetbank.atlassian.net/wiki/spaces/DE/pages/80839436"
owning_team: "Team Shark"
product_owner: "Madelene Söderström"
developers: "Hampus Holmertz, Babak Adeli, Venkatraman Venkatagiri Ravichandran, Iryna Kulatska, Joel Berglund"
github: "https://github.com/nordnet-private/lending-compensation"
last_updated: 2026-07-13
---

# Lending Compensation - Operational Document

**Source:** [Confluence](https://nordnetbank.atlassian.net/wiki/spaces/DE/pages/80839436)

## Description

Lending Compensation is a service running in **NNX** that handles payout of compensation to customers taking part in the Securities Lending program (pension).

Every day the system receives the amount each customer is entitled to per instrument. When compensation is paid out, the system aggregates all daily compensation into one sum per instrument per customer.

**Admin tool:** https://tools.prod.nntech.io/ (Securities Lending tool)

**Architecture docs:** [Backstage C4 model](https://backstage.apps.cd.nntech.io/catalog/default/domain/security-loan/docs/c4-model/securities-lending-containers/)

## Access Rights

| Role | Description |
|------|-------------|
| **Business** | Can create a compensation payout plan and mark it as ready to be paid out once verified |
| **Payments** | Can initiate the payout process of a plan marked as ready |
| **Admin** | Super rights - can perform all actions |

## Payment Flow

### Pre-requisite

`service-securities-lending` (on-prem) feeds Lending Compensation with **Daily Compensation objects** each day. A daily compensation object states how much compensation each customer participating in the program is entitled to - one object per customer per instrument.

### Payout Process

```
1. Business user creates payout plan
   └── Select payout period + pension company
   └── System calculates plan
   └── User verifies and marks as "ready for payment"
                    │
                    ▼
2. Payment operator processes payment
   └── System sends deposit messages via PubSub
   └── → Deposit-Withdrawal domain
   └── D-W domain sends status updates back via PubSub
                    │
                    ▼
3. Handle errors (if any)
   └── Failed deposits highlighted in admin tool
   └── Operator downloads prefilled file
   └── Run in "Anything Goes" for failing payments
                    │
                    ▼
4. Operator marks payment as finalized
   └── Process complete
```

## System Dependencies

### On-Prem (Critical)

| Service | Dependency Level | Purpose |
|---------|-----------------|---------|
| `service-securities-lending` | **Critical** | Feeds compensation data via Lending Data Bucket |

### NNX (Critical)

| Service | Dependency Level | Purpose |
|---------|-----------------|---------|
| Lending Data Bucket | **Critical** | Gets compensation data from service-securities-lending |
| Customer Domain | **Critical** | Maps Account Number → Account ID for deposits |
| Deposit-Withdrawal Domain | **Critical** | Executes the actual deposits to customer accounts |
| Instrument Domain | **Critical** | Gets instrument info for correct transaction messages on customer notes |

### Architecture

```
ON-PREM                              NNX (Cloud)
┌─────────────────────┐              ┌──────────────────────┐
│ service-securities-  │              │ Lending Compensation │
│ lending              │──────────────►│                      │
│                      │  Lending     │  ┌──► Customer Domain│
└─────────────────────┘  Data Bucket │  │    (acct mapping)  │
                                     │  │                    │
                                     │  ├──► Deposit-W Domain│
                                     │  │    (payout)        │
                                     │  │                    │
                                     │  └──► Instrument Dom  │
                                     │       (instrument info)│
                                     └──────────────────────┘
```

**Note:** This is the hybrid on-prem/NNX setup that will eventually be fully migrated to NNX (Phase 2 of the retail rollout plan).

## Database

- **Type:** PostgreSQL
- **Schema:** `lending-compensation-db`

## Operating Instructions

- Most critical download/upload and calculation tasks run **early morning or late evening**
- Scheduled execution times: [service-securities-lending.yml](https://github.com/nordnet-private/securities-lending/blob/master/config/prod/service-securities-lending/service-securities-lending.yml)
- Admin GUI used by: internal staff, Trading Desk, Clearing & Settlement
- **Best time to restart/deploy:** During working hours

## Monitoring & Alarms

| Resource | Link |
|----------|------|
| **Jenkins** | https://jenkins.prod.nordnet.se/job/GitHub/job/jenkins-team-shark/job/lending-compensation/ |
| **Grafana - System Overview** | https://ops.prod.nntech.io/grafana/d/lending-compensation-uid/lending-compensation?orgId=1 |
| **Grafana - Message Queues** | https://ops.prod.nntech.io/grafana/d/ddgqg9lee55a8e/lending-compensation-messages?orgId=1 |
| **Logs (PROD)** | GCP Logs Explorer - `prod-cluster-25354`, namespace `security-loan`, app `lending-compensation` |
| **Logs (TEST)** | GCP Logs Explorer - `test-cluster-29260`, namespace `security-loan`, app `lending-compensation` |

## Related Documents

- **Retail integration:** products/securities-lending/retail/integration-overview.md
- **ExCo presentation:** products/securities-lending/retail/exco-presentation-june-2026.md
- **Overview:** products/securities-lending/overview.md
