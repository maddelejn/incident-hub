---
type: runbook
title: "Securities Financing Troubleshooting Guide (Lending + Borrowing + SFTR)"
category: securities-financing
services: "service-securities-lending, service-securities-borrowing, lending-compensation, service-sftr"
last_updated: 2026-07-13
source: "GitHub prod configs + Confluence operational docs"
---

# Securities Financing Troubleshooting Guide

Combined reference for investigating issues across Securities Lending, Securities Borrowing, SFTR, and Lending Compensation. Sourced from production configs and operational documentation.

---

## Quick Reference: Production Endpoints

### Securities Lending (on-prem)

| Resource | URL |
|----------|-----|
| Service (primary) | http://service-securities-lending1.prod.nordnet.se:8080 |
| Service (failover) | http://service-securities-lending2.prod.nordnet.se:8080 |
| LB | http://service-securities-lending.prod.nordnet.se |
| Swagger | http://service-securities-lending1.prod.nordnet.se:8080/docs/swagger-ui.html |
| Stop a task | Swagger → ScheduleController → stopSchedule → enter task name |
| Check next runs | http://service-securities-lending1.prod.nordnet.se:8080/api/v1/schedules/nextruns |
| Acknowledge alarm | Swagger → ScheduleController → acknowledgeLastExecution |
| Kibana | `application:"service-securities-lending"` |
| Grafana | https://grafana-cd.prod.nordnet.se/d/Nf635PYWk/securities-lending-cpu-memory |
| Alert email | SecuritiesLendingSystem@nordnet.se |

### Securities Borrowing (on-prem)

| Resource | URL |
|----------|-----|
| Service (primary) | https://service-securities-borrowing1.prod.nordnet.se:8443 |
| Service (failover) | https://service-securities-borrowing2.prod.nordnet.se:8443 |
| Admin UI | http://admin-securities-borrowing.prod.nordnet.se |
| Swagger | https://service-securities-borrowing1.prod.nordnet.se:8443/swagger-ui.html |
| Healthcheck | https://service-securities-borrowing1.prod.nordnet.se:8443/healthcheck |
| Kibana | `application:"service-securities-borrowing"` |
| Alert email | SecuritiesBorrowingSystem@nordnet.se |

### Lending Compensation (NNX)

| Resource | URL |
|----------|-----|
| Admin tool | https://tools.prod.nntech.io/ (Securities Lending tool) |
| Grafana overview | https://ops.prod.nntech.io/grafana/d/lending-compensation-uid/lending-compensation |
| Grafana queues | https://ops.prod.nntech.io/grafana/d/ddgqg9lee55a8e/lending-compensation-messages |
| GCP Logs | prod-cluster-25354, namespace security-loan, app lending-compensation |

### SFTR (on-prem)

| Resource | URL |
|----------|-----|
| Service | http://service-sftr1.prod.nordnet.se:8080 |
| Admin GUI | http://service-sftr.prod.nordnet.se |
| Swagger | http://service-sftr1.prod.nordnet.se:8080/docs/swagger-ui.html |
| Kibana | `application:"service-sftr"` |
| Alert email | SFTRSystem@nordnet.se |

---

## SFTP Connections (from prod configs)

| Service | Host | Port | Username | Purpose |
|---------|------|------|----------|---------|
| Lending → Citi | securefiletransfer3.citigroup.com | 22 | nordnba | Loan files, availability, on-loan |
| Lending → Sharegain | sftp.sharegain.com | 22 | nordnetprod | Loan instructions, availability |
| Borrowing → SEB | cft.seb.se | 52222 | spf23330 | Loan requests, responses, positions |
| SFTR → DTCC | (via Delta Capita) | - | - | Regulatory reporting files |

**Citi web UI for manual file access:** https://webui.filetransfer.citi.com (same credentials as SFTP, stored in LastPass shared Shark group)

---

## Combined Daily Timeline (All Services)

```
00:00  Lending: InstrumentBlockReleaseTask
00:15  Borrowing: LoanTransactionCleanupTask

06:00  Borrowing: LocateAvailabilityTask (send locates to SEB)
07:00  Borrowing: ReportInstrumentAdjustmentsTask
07:00  Lending: LoanDownloadTask starts (every 15 min until 00:05)
07:05  Borrowing: RequestScheduledLocatesTask
07:10  Borrowing: CheckLocateAvailabilityTask (verify locates accepted)
07:15  Borrowing: UpdateShortSellingStatusForInstrumentsTask (+ 09:15)
07:30  Lending: CollateralDownloadTask starts (every 15 min until 08:30)
07:50  SFTR: LeiEnrichmentTask (Tue-Sat)
07:55  SFTR: InstrumentEnrichmentTask (Tue-Sat)

08:10  Lending: BookLoanTask starts (every 20 min until 23:59)
08:10  SFTR: UploadTask starts (hourly until 17:15)
08:15  SFTR: DownloadTask starts (hourly until 17:20, Mon-Fri)
08:45  Lending: ReconciliationTask, DailyCompensationTask, CollateralReconciliationTask
09:00  Lending: BackOfficeReportTask (Tue-Sat)
09:00  Borrowing: ProcessProblemPositionTask ← MOST CRITICAL TASK
09:05  Lending: IntradayAvailabilityTask starts (hourly until 21:30)
09:20  Borrowing: SendOnHoldLoanTransactionsTask starts (hourly until 18:20)
09:30  SFTR: UploadVerificationWeekdayTask starts (hourly until 17:35, Tue-Fri)

10:00  Borrowing: BookLoanTask starts (every 15 min until 22:00, Danish returns)

15:50  Borrowing: ForceBookLoanTask (force book Danish returns)

18:15  SFTR: DownloadVerificationTask

23:00  Borrowing: ReconciliationTask
23:30  Lending: CalculateAndUploadEndOfDayAvailabilityTask (every 15 min until 23:50)
23:40  Lending: OnLoanUploadTask (every 10 min until 00:15)

CONTINUOUS:
  Borrowing: FileDownLoadAndProcessTask (every 1 min, Mon-Fri)
  Lending: OpenLoanDownloadTask (every 30 min)
```

---

## Incident Decision Tree

```
SECURITIES FINANCING ISSUE REPORTED
         │
         ├── Customer can't short sell?
         │    └── Check Borrowing: UpdateShortSellingStatusForInstrumentsTask
         │         - Is instrument in SEB availability file?
         │         - Is it on the exclusion list?
         │         - Does located market value meet threshold?
         │         - Admin: check instrument shortable status
         │
         ├── Auto loans not created?
         │    └── Check Borrowing: ProcessProblemPositionTask (09:00)
         │         - Did it run? Check /api/v1/schedules/nextruns
         │         - Failed? Check PROBLEM_POSITION table
         │         - Abasec timeout? Check admin booking tracker
         │         - ESCALATE: C&S must manually book if task fully failed
         │
         ├── Lending availability wrong?
         │    └── Check Lending: CalculateAndUploadEndOfDayAvailabilityTask
         │         - Did EOD task run at 23:30?
         │         - Check intraday task (hourly 09:05-21:30)
         │         - Can't reach Citi/Sharegain SFTP? Single failures OK
         │         - NOTE: If re-run EOD in morning, update start_time in
         │           SECURITIES_LENDING.AVAILABILITY_CALCULATION table
         │
         ├── Loan booking locks Abasec?
         │    └── Lending: BookLoanTask is the usual culprit
         │         1. Stop via Swagger: ScheduleController/stopSchedule → "BookLoanTask"
         │         2. Verify: /api/v1/schedules/nextruns (should not list BookLoanTask)
         │         3. If not enough: kill service-securities-lending1
         │         4. C&S follows up next day
         │
         ├── Compensation not calculated?
         │    └── Check Lending: DailyCompensationTask (Tue-Sat 08:45, Sat 12:00)
         │         - Was OpenLoan file received? (from Citi, every 30 min)
         │         - Citi sent incorrect data? → Kristofer Burén requests updated files
         │         - Re-upload manually via admin → re-run DailyCompensationTask
         │
         ├── Customer payout failed?
         │    └── Check NNX: Lending Compensation
         │         - Did compensation data arrive via Lending Data Bucket?
         │         - Check Grafana message queues
         │         - Failed deposits? Download prefilled file → run in Anything Goes
         │
         ├── SFTR reporting failed?
         │    └── Check SFTR: UploadTask (hourly 08:10-17:15)
         │         - Missing file from AbaSec? Check SMB share
         │         - Upload to DTCC failed? Re-upload from admin GUI
         │         - SSH key error? Retry from admin
         │         - DEADLINE: T+1 regulatory requirement
         │
         ├── SEB not responding?
         │    └── Borrowing: Check FileDownLoadAndProcessTask logs
         │         - Single failure = OK (runs every minute)
         │         - Persistent? Contact SEB: elis.olsson@seb.se
         │         - Can manually upload response files via Swagger
         │           POST /api/v1/file/loan/responses
         │
         ├── Citi SFTP unreachable?
         │    └── Lending: Jobs run continuously, single failures OK
         │         - Persistent? Contact: aslsupport@citi.com (24/7)
         │         - Manual file access: https://webui.filetransfer.citi.com
         │
         └── Sharegain SFTP unreachable?
              └── Lending: Jobs run continuously, single failures OK
                   - Persistent? Contact: sltechsupport@sharegain.com
                   - +44 20 3884 2405 (select "client services")
```

---

## External Contacts (Escalation Order)

### SEB (Securities Borrowing)

| Priority | Person | Email | Phone |
|----------|--------|-------|-------|
| Business | Elis Olsson | elis.olsson@seb.se | +46 8 506 233 85 |
| Corporate Actions | Tayyibe Tokmak | tayyibe.tokmak@seb.se | +46 8 763 51 63 |
| Tech | Markus Falck | markus.falck@seb.se | +46 70 462 20 92 |
| Settlement | Group | eoseclendsettlements@seb.se | +371 67 770 405 |

### Citi (Securities Lending)

| Priority | Contact | Email | Phone |
|----------|---------|-------|-------|
| 1st | ASL Support (24/7) | aslsupport@citi.com | - |
| 2nd | Gavin Callan | gavin.callan@citi.com | +353 1 622 6119 |
| 3rd | Peter Lennon | peter.lennon@citi.com | +353 1 622 4075 |
| 4th (emergency) | Jemma Finglas | jemma.finglas@citi.com | +44 207 986 3348 |

### Sharegain (CSLA Platform)

| Priority | Contact | Email | Phone |
|----------|---------|-------|-------|
| 1st | Tech Support | sltechsupport@sharegain.com | +44 20 3884 2405 |
| Escalation | Patrick Sykes | patrick.sykes@sharegain.com | +44 20 3830 8991 |

### DTCC / Delta Capita (SFTR)

| Contact | Email | Purpose |
|---------|-------|---------|
| GTR Connectivity | gtrconnectivity@dtcc.com | SFTP issues |
| GTR Support | gtrsupport@dtcc.com | General tech |
| Delta Capita Support | reporthubsupport@deltacapita.com | Report Hub |
| Melanie Abelardo (KAM) | melanie.abelardo@deltacapita.com | Account manager |
| Carina Shane | CShane@dtcc.com | Business (DTCC) |

---

## Key Production Config Values

### Securities Lending

| Config | Value |
|--------|-------|
| Citi SFTP host | securefiletransfer3.citigroup.com:22 |
| Sharegain SFTP host | sftp.sharegain.com:22 |
| Primary hostname | service-securities-lending1.prod.nordnet.se |
| Nordnet compensation % | 50% |
| Recall buffer | 10% |
| Min payout (SEK/NOK/DKK) | 1.0 |
| Min payout (EUR) | 0.1 |
| Compensation payout schedule | 15th of Jan, Apr, Jul, Oct at 22:00 (quarterly) |
| DB | Oracle, schema SECURITIES_LENDING |
| GCP archive bucket | nordnet-prod-securities-lending |
| GCP data bucket | nordnet-prod-securities-lending-data |

### Securities Borrowing

| Config | Value |
|--------|-------|
| SEB SFTP host | cft.seb.se:52222 |
| SEB username | spf23330 |
| Primary hostname | service-securities-borrowing1.prod.nordnet.se |
| SEB Nordnet account | NORDNET |
| SEB account at Nordnet | 2027902 |
| Retail settlement days | 1 |
| Mandatory agreement | ID 14 |
| Allowed account types | 108, 110, 133 |
| Enabled issuer countries | NO, SE, DK, FI |
| Locate categories | GC, W, S, SS |
| Danish market only for BookLoanTask | DK |
| DB | Oracle, schema SECURITIES_BORROWING |

### Short Selling Thresholds (from prod config)

**Auto Covered (GC/W categories):**

| Country | Min Located Market Value |
|---------|------------------------|
| SE | 1,000,000 SEK |
| NO | 1,000,000 NOK |
| DK | 750,000 DKK |
| FI | 100,000 EUR |

**Intraday (S/SS categories):**

| Country | Min Located Market Value | Min Market Cap |
|---------|------------------------|----------------|
| SE | 100,000 SEK | 100,000,000 SEK |
| NO | 100,000 NOK | 100,000,000 NOK |
| DK | 100,000 DKK | 100,000,000 DKK |
| FI | 10,000 EUR | 10,000,000 EUR |

---

## Swagger Quick Commands

### Securities Lending

| Action | Endpoint |
|--------|----------|
| Stop a task | POST `ScheduleController/stopSchedule?task={TaskName}` |
| Check next runs | GET `/api/v1/schedules/nextruns` |
| Acknowledge alarm | POST `ScheduleController/acknowledgeLastExecution?task={TaskName}` |

### Securities Borrowing

| Action | Endpoint |
|--------|----------|
| Fix wrong loan status | PUT `/api/v1/loan/seb/modify` (provide sebLoanId + correct status) |
| Upload response file manually | POST `/api/v1/file/loan/responses` (paste CSV content) |
| Check availability | GET `/api/v1/availability/tradablelocate` |

### SFTR

| Action | Endpoint |
|--------|----------|
| Re-upload report file | POST `/internal/v1/upload-report/upload/{fileName}` |
| Re-upload LEI enrichment | POST `/internal/v1/lei-enrichment/upload/{fileName}` |
| Execute task manually | PUT `/api/v1/schedules/{task}/execute` |
