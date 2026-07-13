---
type: product-knowledge
title: "Service-SFTR - Securities Financing Transactions Regulation Reporting"
status: active
source: "https://nordnetbank.atlassian.net/wiki/spaces/DE/pages/80810260"
owning_team: "Team Shark (Domain Execution)"
system_owner: "Madelene Söderström"
product_owner: "Madelene Söderström"
platform: "on-prem"
database: "Oracle (ORAP)"
github: "https://github.com/nordnet-private/sftr"
last_updated: 2026-07-13
---

# Service-SFTR - Regulatory Reporting

**Source:** [Confluence - System Doc](https://nordnetbank.atlassian.net/wiki/spaces/DE/pages/80834341) | [Confluence - Technical Ops](https://nordnetbank.atlassian.net/wiki/spaces/DE/pages/80843397)

## What is SFTR?

**SFTR** = Securities Financing Transactions Regulation. ESMA regulation requiring that all Securities Financing Transactions (SFTs) must be reported to a registered trade repository (TR).

Developed by the Financial Stability Board (FSB) and European Systemic Risk Board (ESRB) to enhance transparency into "shadow banking" and reduce systemic risk.

### SFT types that must be reported:
- Margin lending
- Securities lending
- Buy-sell / sell-buy
- Repurchases

### Reporting requirement: **T+1** (must report by the next business day)

## System Description

Service-SFTR handles three main tasks:

```
AbaSec (generates SFT reporting file)
    │
    ▼ (via SMB file share)
Service-SFTR
    │
    ├──► Upload reporting files to Delta Capita (formerly Report Hub)
    ├──► Generate & upload enrichment data to Delta Capita
    └──► Download & reconcile result data from DTCC
              │
              ▼
         DTCC (Trade Repository)
```

### Data Flow

```
INTERNAL                           EXTERNAL
┌─────────────────┐               ┌──────────────────┐
│ AbaSec          │──SMB──────────►│                  │
│ (SFT report gen)│               │ Service-SFTR     │
└─────────────────┘               │                  │
┌─────────────────┐               │  ┌───────────┐   │      ┌─────────────┐
│ Service-GLEIF   │◄──────────────│  │ Upload    │───SFTP──►│ Delta       │
│ (entity LEI)    │               │  │ Enrich    │   │      │ Capita      │
└─────────────────┘               │  │ Download  │   │      │ (Report Hub)│
┌─────────────────┐               │  └───────────┘   │      └──────┬──────┘
│ Service-Core    │◄──────────────│                  │             │
│ (customer data) │               │  Admin GUI       │             ▼
└─────────────────┘               │  (Reg Officers)  │      ┌─────────────┐
┌─────────────────┐               └──────────────────┘      │    DTCC     │
│ BSPRDDBLS       │◄──(instrument enrichment)               │    (TR)     │
│ (MSSQL DB)      │                                         └─────────────┘
└─────────────────┘
┌─────────────────┐
│ Mail Server     │◄──(warning emails)
└─────────────────┘
```

## External Provider Contacts

### Business Questions

| Person | Role | Email |
|--------|------|-------|
| Carina Shane | Key Account Manager, DTCC Nordnet | CShane@dtcc.com |
| Jemma L. Freeman | Report Hub Contact Nordnet | jfreeman@dtcc.com |

### Technical Questions

| Contact | Email | Purpose |
|---------|-------|---------|
| GTR Connectivity | gtrconnectivity@dtcc.com | SFTP connectivity |
| GTR Support | gtrsupport@dtcc.com | All other tech questions |
| Delta Capita (Report Hub) Support | reporthubsupport@deltacapita.com | Report Hub issues |
| Melanie Abelardo (KAM) | melanie.abelardo@deltacapita.com | Key Account Manager at Delta Capita |

## Scheduled Tasks

| Task | Schedule | Description | Re-run? |
|------|----------|-------------|---------|
| **UploadTask** | Hourly, 08:10-17:15 | Fetches AbaSec report file from SMB share, uploads to DTCC. Archives files before upload. | Yes (won't duplicate) |
| **DownloadTask** | Hourly, 08:15-17:20, Mon-Fri | Downloads result files from DTCC | Yes |
| **LeiEnrichmentTask** | 07:50, Tue-Sat | Generates and uploads LEI enrichment files (3 files) to DTCC | Yes (may create duplicates) |
| **InstrumentEnrichmentTask** | 07:55, Tue-Sat | Fetches instrument data from MSSQL, creates enrichment file, uploads to DTCC | Yes (may create duplicates) |
| **UploadVerificationWeekdayTask** | Hourly, 09:30-17:35, Tue-Fri | Verifies AbaSec has created files. Sends email if missing. | Yes |
| **UploadVerificationWeekendTask** | Hourly, 14:30-17:35, Sat | Same as above but starts later (AbaSec night jobs not done before 14:30 on Saturdays) | Yes |
| **DownloadVerificationTask** | 18:15, Mon-Fri | Verifies all DTCC result data has been downloaded | Yes |

**New files from AbaSec expected:** Tuesday → Saturday (day after trading day). UploadTask runs daily as safety net in case AbaSec is late.

## Infrastructure

| Environment | Machine 1 | Machine 2 |
|-------------|-----------|-----------|
| PROD | service-sftr1.prod.nordnet.se:8080 | service-sftr2.prod.nordnet.se:8080 |
| TEST | service-sftr1.test.nordnet.se:8080 | service-sftr2.test.nordnet.se:8080 |

- **Admin GUI PROD:** http://service-sftr.prod.nordnet.se
- **Admin GUI TEST:** http://service-sftr.test.nordnet.se
- **Swagger:** `{machine}/docs/swagger-ui.html`

### Access Roles

| Role | Permissions |
|------|------------|
| **SFTR_User** | List/download files |
| **SFTR_Administrator** | All SFTR_User permissions + manual uploads, schedule control |

## Database

- **DB:** Oracle (ORAP)
- Key table: `UPLOAD_REPORT_FILE` - log of all uploaded files and their status
- Statuses: `SUCCESS`, `FAILURE`, `PENDING`, `INVALID_FILE`, `MANUALLY_HANDLED`
- File contents stored in DB (can be manually re-uploaded on failure)
- **GDPR:** Reporting files stored 10 years per regulation. Can be deleted on request.

## Monitoring & Alarms

- **OP5:** Alarm triggered if a task fails
- **Warning emails:** Sent to `SFTRSystem@nordnet.se` (Trading Bull + Regulatory Officers)
- **Test env emails:** `SFTRSystemTEST@nordnet.se`
- **Kibana PROD:** `application:"service-sftr"`
- **Logs:** `/var/log/nordnet/service-sftr/service-sftr.log`

### Warning emails sent when:
- Missing files from AbaSec
- Report file from AbaSec is empty or too small (<10,000 bytes)
- Expected reports from Report Hub not received
- File upload to Report Hub fails

## Known Issues & Solutions

### SSH key exchange error on upload
**Symptom:** `[PROTOCOL_ERROR] invalid packet length: 1349676916`
**Fix:** Press "Upload" in admin GUI to retry the file.

### DTCC SFTP cipher update (June 2026)
DTCC upgraded SFTP encryption to version 10.6.0.7. Backward compatibility with 10.5.0.9 available through November 2026. Deprecated ciphers must be updated. Production release: June 6, 2026.

### UAT path issue (May 2026)
TEST environment was using incorrect SFTP path. Delta Capita confirmed correct UAT path:
- Host: `dtcceuuatsftp.trafficmanager.net`
- Path: `/nordnetusftp/ppleu` (not `/nordnetusftp/ClientResponse`)

## Information Classification

| Category | Level | Reasoning |
|----------|-------|-----------|
| **Confidentiality** | Very High | Contains client SFT transaction data |
| **Integrity** | Low | Report generated by AbaSec, service only uploads - no human intermediaries |
| **Availability** | Medium | Reporting required T+1 per regulation |

## Related Documents

- **Securities lending overview:** products/securities-lending/overview.md
- **Service-securities-lending ops:** products/securities-lending/pension/service-securities-lending-operational.md
