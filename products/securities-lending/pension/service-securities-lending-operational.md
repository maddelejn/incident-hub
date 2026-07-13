---
type: product-knowledge
title: "Service-Securities-Lending - Operational Document (On-Prem)"
status: active
source: "https://nordnetbank.atlassian.net/wiki/spaces/DE/pages/80839443"
owning_team: "Team Shark"
product_owner: "Madelene Söderström"
developers: "Hampus Holmertz, Venkatraman Venkatagiri Ravichandran, Joel Berglund, Ricard Kindfalk, Nedo Skobalj, Matti Lundgren"
github: "https://github.com/nordnet-private/securities-lending"
platform: "on-prem"
database: "Oracle, schema: SECURITIES_LENDING"
last_updated: 2026-07-13
---

# Service-Securities-Lending - Operational Document

**Source:** [Confluence - Citi version](https://nordnetbank.atlassian.net/wiki/spaces/DE/pages/80839443) | [Confluence - Sharegain version](https://nordnetbank.atlassian.net/wiki/spaces/DE/pages/1478000641)

## Description

On-prem service that exchanges files with Citi and Sharegain regarding lent securities. Calculates compensation for participating customers based on lending history.

## External Provider Contacts

### Citi (Escalation order)

| Priority | Person | Role | Contact |
|----------|--------|------|---------|
| 1st | ASL Support | 24/7/365 Tech Support | aslsupport@citi.com |
| 2nd | Gavin Callan | ASL EMEA Head of Product Mgmt | gavin.callan@citi.com, +35316226119 |
| 3rd | Peter Lennon | Senior VP, Client Executive | peter.lennon@citi.com, +35316224075 |
| 4th (emergency) | Jemma Finglas | Head of Biz Dev Asset Financing EMEA | jemma.finglas@citi.com, +442079863348 |

### Sharegain

| Priority | Person | Contact |
|----------|--------|---------|
| 1st | Tech Support Team | sltechsupport@sharegain.com, +44 20 3884 2405 (select "client services") |
| Escalation | Patrick Sykes | patrick.sykes@sharegain.com, +44 20 3830 8991 |

### Citi SFTP Migration (Completed June 2025)

Citi migrated to new SFTS platform. Nordnet completed migration on 2025-06-25.
- Contact: Gatien Raffet (gatien1.raffet@citi.com)
- Citi web UI for files: https://webui.filetransfer.citi.com
- SFTP credentials stored in LastPass (shared Shark group)

## Scheduled Tasks

| Task | Schedule | Description |
|------|----------|-------------|
| **LoanDownloadTask** | Every 15 min, 07:00-00:05 | Downloads loan instructions from Sharegain SFTP, stores in DB, deletes from SFTP |
| **BookLoanTask** | Every 20 min, 08:10-00:15 | Books loan instructions into Abasec via service-abatron |
| **CalculateAndUploadEndOfDayAvailabilityTask** | Every 15 min, 23:30-23:50 (runs once) | Fetches positions from Core + loans from Abasec, calculates EOD availability, uploads to Citi and Sharegain SFTP |
| **CalculateAndUploadIntradayAvailabilityTask** | Every 60 min, 09:05-21:30 | Same as EOD but intraday. Uploads to Sharegain only (not Citi) |
| **OnLoanUploadTask** | Every 10 min, 23:40-00:15 (runs once) | Uploads Nordnet's view of loans to Citi SFTP |
| **OpenLoanDownloadTask** | Every 30 min | Downloads Citi's view of all loans. Needed for DailyCompensation and Reconciliation |
| **CollateralDownloadTask** | Every 15 min, 07:30-08:30 | Downloads Citi's collateral reports |
| **ReconciliationTask** | Mon-Fri at 08:45 | Reconciles Nordnet vs Citi view of on-loan positions |
| **DailyCompensationTask** | Tue-Sat at 08:45 | Calculates daily compensation. Saturday run covers Fri+Sat+Sun. Syncs to Lending Data Bucket in NNX |
| **BackOfficeReportTask** | Tue-Sat at 09:00 | Generates report for Corporate Actions via SMB share |
| **CollateralReconciliationTask** | Mon-Fri at 08:45 | Reconciles Citi collateral vs loan market values |
| **InstrumentBlockReleaseTask** | Daily at midnight | Releases blocks when on-loan > available |
| **RateAdjustmentDownloadTask** | DISABLED | Downloads rate adjustments from Citi (disabled, may remove) |

All tasks can be re-run in case of errors.

**Critical note on EOD Availability:** If re-run in the morning, update `start_time` in `SECURITIES_LENDING.AVAILABILITY_CALCULATION` table to previous day, otherwise the daily re-run prevention will block the evening run.

## Dependencies

| Service | Level | Purpose | Repo |
|---------|-------|---------|------|
| service-abatron | Critical | Register/query loans in Abasec | nordnet-private/abatron |
| service-core | Critical | SL-instrument info, accounts, positions | nordnet-private/core |
| service-fx-approximator | Critical | FX rates for compensation calculation | nordnet-private/fx-approximator |
| service-instrument | Critical | Instrument info for downloads/whitelist | nordnet-private/service-instrument |
| service-trading-info | Critical | Trading days for compensation calc | nordnet-private/trading-info |
| Mail | Critical | Task failure notifications | mail.prod.nordnet.se |
| Citi SFTP | Critical | File exchange with Citi | Config in service-securities-lending.yml |
| Sharegain SFTP | Critical | File exchange with Sharegain | Config in service-securities-lending.yml |
| Lending Data Bucket | Critical | Sync compensation data to NNX | - |

### Inbound Callers

**ECBPRDDB** (System owner: Rasmus Reivilä) calls:
- `/api/v1/collateral`
- `/api/v1/openloan`

**Warning:** These endpoints are NOT authenticated.

## Infrastructure

| Environment | Machine 1 | Machine 2 | LB |
|-------------|-----------|-----------|-----|
| CI | service-securities-lending1.ci.nordnet.se:8080 | service-securities-lending2.ci.nordnet.se:8080 | service-securities-lending.ci.nordnet.se |
| TEST | service-securities-lending1.test.nordnet.se:8080 | service-securities-lending2.test.nordnet.se:8080 | service-securities-lending.test.nordnet.se |
| PROD | service-securities-lending1.prod.nordnet.se:8080 | service-securities-lending2.prod.nordnet.se:8080 | service-securities-lending.prod.nordnet.se |

- **Primary/failover:** Machine 1 is primary (runs all tasks). Machine 2 is failover only.
- **Prod locations:** Different physical locations. Check netbox.prod.nordnet.se for details.
- **Swagger:** `{machine}/docs/swagger-ui.html`
- **Best deploy time:** During working hours

## Monitoring

| Resource | Link |
|----------|------|
| Jenkins | https://jenkins.prod.nordnet.se/job/GitHub/job/jenkins-team-equityfinance/job/securities-lending/ |
| Grafana (CPU/Memory) | https://grafana-cd.prod.nordnet.se/d/Nf635PYWk/securities-lending-cpu-memory |
| Kibana PROD | applogs-ui.prod.nordnet.se (query: `application:"service-securities-lending"`) |
| Kibana TEST | applogs-ui.test.nordnet.se |

**Acknowledge alarms via Swagger:** `{machine}/docs/swagger-ui/index.html#/ScheduleController/acknowledgeLastExecution`

## Known Problems & Solutions

### BookLoanTask locks Abasec
**Symptom:** Abasec locks preventing night job sync or transaction notes.
**Fix:**
1. Stop BookLoanTask via Swagger: `ScheduleController/stopSchedule` → enter "BookLoanTask"
2. Verify stopped: check `/api/v1/schedules/nextruns` - BookLoanTask should not be listed
3. If insufficient, kill service-securities-lending1 (stops all tasks, no data loss)
4. Clearing & Settlement follows up next day to verify bookings

### Citi sends incorrect open loans reports
**Symptom:** Wrong daily compensation calculations.
**Fix:** Kristofer Burén requests updated files from Citi → upload manually via admin → re-run DailyCompensationTask via Swagger.

### Citi sends loan instructions with wrong identifier
**Symptom:** Bloomberg identifier instead of ISIN → failed to parse in admin.
**Fix:** Clearing & Settlement makes manual booking in Abasec.

### SFTP connectivity issues (Citi or Sharegain)
**Fix:** Tasks run continuously, single failures are not critical. Wait to see if persistent. If so, contact vendor support. Night failures can wait until morning.

### Daily compensation not visible in admin
**Investigate:** Check if open loan file received for that date. Check logs.
