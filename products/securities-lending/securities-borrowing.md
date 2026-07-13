---
title: Securities Borrowing
product_owner: Madelene Söderström
owning_team: Shark - Domain Execution
platform: on-prem
rpo: 4h
rto: 4h
database: Oracle (schema SECURITIES_BORROWING)
source: https://nordnetbank.atlassian.net/wiki/spaces/DE/pages/80823011
last_updated: 2026-07-13
---

# Securities Borrowing

## 1. Overview

Securities Borrowing handles all security loans that customers take up at Nordnet. **SEB is the counterparty** for all loans. Communication between Nordnet and SEB is conducted via **SFTP files**.

In addition to managing loan lifecycle, the system is responsible for:

- Determining which instruments are **enabled for short selling**
- Calculating **borrowable share availability** for each instrument

---

## 2. Short Selling Types

### Auto Covered Shorting

- The system **automatically takes up a loan from SEB** the next morning if a customer's short position was not closed by end of day.
- The loan is **automatically returned** when the customer closes the position.
- The customer **does not need to contact the brokerage desk**.

### Intraday Shorting

- The customer **must close the position before market close**.
- If the customer wants to keep the position overnight, they must **contact the brokerage desk** to arrange a security loan.

---

## 3. Short Selling Criteria

### Auto Covered Short Selling Criteria

All of the following must be met:

1. Instrument is present in the **SEB availability file** (category **GC** or **W**)
2. Instrument has been **successfully located from SEB**
3. Instrument is **not in the exclusion list**
4. **Market value of located quantity** meets or exceeds country limits:

| Country | Minimum Market Value |
|---------|---------------------|
| SE      | 1,000,000 SEK       |
| DK      | 750,000 DKK         |
| NO      | 1,000,000 NOK       |
| FI      | 100,000 EUR         |

### Intraday Short Selling Criteria

An instrument qualifies for intraday shorting if **either** of these paths is satisfied:

**Path A:** Allowed for auto covered shorting AND not excluded from intraday shorting.

**Path B:** All of the following:

1. Instrument is in the **intraday buffer** or in **SEB availability** (category **S** or **SS**)
2. Instrument has been **successfully located from SEB**
3. Instrument is **not in the exclusion list**
4. **Market value** meets or exceeds country limits:

| Country | Minimum Market Value |
|---------|---------------------|
| SE      | 100,000 SEK         |
| DK      | 100,000 DKK         |
| NO      | 100,000 NOK         |
| FI      | 10,000 EUR          |

5. **Market capitalization** meets or exceeds country limits:

| Country | Minimum Market Cap |
|---------|--------------------|
| SE      | 100,000,000 SEK    |
| DK      | 100,000,000 DKK    |
| NO      | 100,000,000 NOK    |
| FI      | 10,000,000 EUR     |

---

## 4. SEB Availability Categories

| Category | Name              | Description                        |
|----------|-------------------|------------------------------------|
| GC       | General Collateral| Large volume available, low fee    |
| W        | Warm              | Large volume available, low fee    |
| S        | Special           | Lower volume available, higher fee |
| SS       | Super Special     | Lower volume available, higher fee |

- **GC and W** instruments are eligible for **auto covered** short selling.
- **S and SS** instruments are eligible for **intraday** short selling (Path B).

---

## 5. Locating

Locating is the process of reserving shares for potential borrowing from SEB. There are three methods:

### Locate Indicative SEB Availability

- Uses the **morning availability file** received from SEB via SFTP.
- Processed by the `LocateAvailabilityTask` at **06:00 Mon-Fri**.
- Result checked by `CheckLocateAvailabilityTask` at **07:10 Mon-Fri**.

### Locate Intraday Availability

- Sources availability from the **securities lending buffer via Citi**.
- Used for intraday shorting instruments.

### Manual Locate

- A **broker requests extra volume** directly from SEB.
- Used when the standard availability is insufficient or for special cases.

---

## 6. Loan Types

### Auto Loans

- Created **automatically** for auto-covered short positions.
- Triggered by the `ProcessProblemPositionTask` at **09:00 Mon-Fri**.
- System handles the full lifecycle without manual intervention.

### Broker Loans

- Created **manually** when a customer contacts the brokerage desk.
- The loan **must be held for a minimum of 1 week**.
- Used for intraday positions that need to be kept overnight or for specific customer requests.

---

## 7. Loan Validation

Before a loan can be created, the following validations must pass:

- Account is **not blocked**
- Account type is **108**, **110**, or **133**
- Agreement **14** is active on the account
- Customer has a **valid KYC**
- Customer has a **valid LEI** (if applicable)

---

## 8. Fee Calculation

The borrowing fee is calculated as:

```
Fee = max(SEB fee + additional fee, minimum fee)
```

Additionally:

- A **one-time charge** is applied per loan.
- Some customers have **negotiated better terms** (custom fee agreements).

---

## 9. Loan Statuses

| Status               | Description                                                        |
|----------------------|--------------------------------------------------------------------|
| Created              | Loan request has been created in the system                        |
| Accepted             | SEB has accepted the loan request                                  |
| Settled              | Loan has been settled and shares delivered                         |
| Rejected             | SEB has rejected the loan request                                  |
| Error                | An error occurred during processing                                |
| Unanswered           | No response received from SEB                                      |
| Returned             | Return request has been sent to SEB                                |
| Returned Settled     | Return has been settled and shares returned                        |
| Missing              | Loan exists in SEB records but not in Securities Borrowing         |
| Unknown Loan         | Loan exists in Securities Borrowing but not in SEB records         |
| Unknown Instrument   | Loan references an instrument not recognized by the system         |
| Discarded            | Loan has been discarded/cancelled                                  |

---

## 10. Loan Pool Management

SEB loans are treated as a **pool** rather than individual discrete loans from the customer's perspective.

When returning shares, the system selects the loan to return against using this strategy:

> **Choose the smallest loan with the highest fee where the first return date has occurred.**

This strategy is designed to **maximize Nordnet's profit** by returning the most expensive small loans first.

---

## 11. Danish Loan Returns

Danish loan returns are handled differently from other markets:

- The system **waits for all return requests** in an instrument to be **accepted by SEB** before booking the return in Abasec.
- The `BookLoanTask` runs every **15 minutes between 10:00-22:00 Mon-Fri** to check for accepted returns.
- A `ForceBookLoanTask` runs at **15:50 Mon-Fri** which **force books proportionally** if not all returns have been accepted by that time.

This ensures that partial acceptances do not cause booking mismatches.

---

## 12. Reconciliation

A daily reconciliation runs at **23:00 Mon-Fri** via the `ReconciliationTask`.

The reconciliation compares three views:

1. **SEB view** - SEB's record of outstanding loans
2. **Securities Borrowing view** - The application's internal loan records
3. **Abasec view** - The settlement system's record

### Error Types

| Error Type                          | Description                                                  |
|-------------------------------------|--------------------------------------------------------------|
| Missing                             | Loan exists at SEB but not in Securities Borrowing           |
| Unknown Loan                        | Loan exists in Securities Borrowing but not at SEB           |
| Unknown Instrument                  | Loan references an unrecognized instrument                   |
| SEB Mismatch                        | Discrepancy between SEB and Securities Borrowing records     |
| Not Settled                         | Loan should be settled but is not                            |
| Abasec Mismatch                     | Discrepancy between Securities Borrowing and Abasec records  |
| Broker Loans Abasec Mismatches      | Broker loan discrepancies in Abasec                          |
| Abasec Loans With Possible Loss     | Loans in Abasec that may result in a financial loss          |

---

## 13. External Contacts (SEB)

| Role                | Name             | Email                              | Phone               |
|---------------------|------------------|-------------------------------------|----------------------|
| Business            | Elis Olsson      | elis.olsson@seb.se                  | +46 8 506 233 85     |
| Corporate Actions   | Tayyibe Tokmak   | tayyibe.tokmak@seb.se               | +46 8 763 51 63      |
| Tech                | Markus Falck     | markus.falck@seb.se                 | +46 70 462 20 92     |
| Settlement          | -                | eoseclendsettlements@seb.se         | +371 67 770 405      |

---

## 14. Scheduled Tasks

| Task                                      | Schedule                          | Description                                                |
|-------------------------------------------|-----------------------------------|------------------------------------------------------------|
| FileDownLoadAndProcessTask                | Every minute, Mon-Fri             | Polls SEB SFTP for new files                               |
| LoanTransactionCleanUpTask                | 00:15 daily                       | Cleans up old loan transaction records                     |
| LocateAvailabilityTask                    | 06:00 Mon-Fri                     | Processes SEB morning availability file for locates        |
| CheckLocateAvailabilityTask               | 07:10 Mon-Fri                     | Verifies locate availability results                       |
| SendOnHoldTransactionsTask                | Hourly 08:40-17:45 Mon-Fri       | Sends queued/on-hold transactions to SEB                   |
| ProcessProblemPositionTask                | 09:00 Mon-Fri                     | **CRITICAL** - Handles auto loans for short positions      |
| ReconciliationTask                        | 23:00 Mon-Fri                     | Daily reconciliation across SEB, app, and Abasec           |
| ReportInstrumentAdjustmentTask            | 07:00 Mon-Fri                     | Reports instrument adjustments                             |
| UpdateShortableInstrumentsTask            | Hourly 07:15-17:20 Mon-Fri       | Updates which instruments are shortable                    |
| UpdateShortSellingStatusForInstrumentTask | Hourly 07:15-17:20 Mon-Fri       | Updates short selling status per instrument                |
| BookLoanTask                              | Every 15 min, 10:00-22:00 Mon-Fri| Books accepted Danish loan returns                         |
| ForceBookLoanTask                         | 15:50 Mon-Fri                     | Force books Danish returns proportionally                  |

**Note:** The `ProcessProblemPositionTask` at 09:00 is the most critical task. If it fails, auto loans for the day will not be created, requiring manual intervention by C&S.

---

## 15. Dependencies

### Internal Services

| Service                         | Purpose                                     |
|---------------------------------|---------------------------------------------|
| service-abatron                 | Settlement system integration (Abasec)      |
| service-core                    | Core platform services                      |
| service-instrument              | Instrument data and lookup                  |
| service-securities-lending      | Securities lending coordination             |
| service-kyc                     | KYC validation for loan eligibility         |
| service-validationadjustment    | Validation and adjustment processing        |
| service-shorting                | Short selling functionality                 |
| service-trading-info            | Trading information                         |
| service-instrument-search       | Instrument search capabilities              |
| service-cdb                     | Customer database                           |
| service-gleif                   | LEI validation (Global LEI Foundation)      |

### External Dependencies

| Dependency | Purpose                                     |
|------------|---------------------------------------------|
| SEB SFTP   | File-based communication with SEB           |
| Mail       | Email notifications and alerts              |

---

## 16. Infrastructure

### Production

| Component   | URL / Host                                              |
|-------------|---------------------------------------------------------|
| Service 1   | service-securities-borrowing1.prod.nordnet.se:8443      |
| Service 2   | service-securities-borrowing2.prod.nordnet.se:8443      |
| Admin UI    | admin-securities-borrowing.prod.nordnet.se              |

### Test

| Component   | URL / Host                                              |
|-------------|---------------------------------------------------------|
| Service 1   | service-securities-borrowing1.test.nordnet.se:8443      |
| Service 2   | service-securities-borrowing2.test.nordnet.se:8443      |
| Admin UI    | admin-securities-borrowing.test.nordnet.se              |

### Architecture Notes

- **Hot standby setup** (not true load balancing) due to AD session handling requirements.
- **Database:** Oracle, schema `SECURITIES_BORROWING`

### Source Code

| Repository                              | Purpose            |
|-----------------------------------------|--------------------|
| nordnet-private/securities-borrowing       | Backend service    |
| nordnet-private/admin-securities-borrowing | Admin UI           |

---

## 17. Access Roles

| Role                              | Description                             |
|-----------------------------------|-----------------------------------------|
| SECURITIES_BORROWING_BROKER       | Broker-level access for loan management |
| SECURITIES_BORROWING_OPERATIONS   | Operations team access                  |
| SECURITIES_BORROWING_USER         | Read-only access                        |
| SECURITIES_BORROWING_ADMIN        | Full administrative access              |

---

## 18. Monitoring

| Channel         | Details                                                    |
|-----------------|------------------------------------------------------------|
| Jenkins         | Scheduled task monitoring and build pipelines              |
| Kibana          | Log search: `application:"service-securities-borrowing"`   |
| Email Alerts    | Warning emails sent to SecuritiesBorrowingSystem@nordnet.se|
| RPO             | 4 hours                                                    |
| RTO             | 4 hours                                                    |

---

## 19. Known Problems and Solutions

### ProcessProblemPositionTask Failure

**Symptom:** Auto loans are not created for the day.

**Solution:**
1. Check the `PROBLEM_POSITION` table in the database for errors.
2. C&S must perform **manual booking** of the required loans.

### No Responses from SEB

**Symptom:** Loan requests remain in "Created" or "Unanswered" status.

**Solution:**
- This is **usually a problem on the SEB side**.
- Contact SEB to investigate.
- Response files can be **manually uploaded** to the SFTP if SEB provides them out-of-band.

### Duplicate Loans

**Symptom:** The same loan appears twice in the system.

**Cause:** The **loan position file** was received from SEB **before the response file**, causing the system to create a duplicate entry.

**Solution:** Identify and reconcile the duplicate entries manually.

### Service Hangs on FileDownLoadAndProcessTask

**Symptom:** The service becomes unresponsive while polling SEB SFTP.

**Solution:** **Restart the service.** The task will resume normal polling after restart.

### Abasec Timeouts

**Symptom:** Booking requests to Abasec time out.

**Solution:**
- The admin UI **tracks booking requests** and their status.
- Failed bookings can be **re-run** from the admin interface.
- Alternatively, mark the booking as **handled** if it was processed through other means.

### Cannot Move Loan

**Symptom:** Attempting to move a loan between accounts fails.

**Cause:** **Parameter mismatch** between the admin interface and Abasec.

**Solution:** Verify and correct the parameters in both systems before retrying.

### Wrong Status on Loans

**Symptom:** A loan has an incorrect status that prevents normal processing.

**Solution:** Fix the status directly via the Swagger API endpoint:
```
/api/v1/loan/seb/modify
```

### No SL-Instrument for Broker Loan

**Symptom:** A broker loan cannot be created because no securities lending instrument exists.

**Solution:** Redirect the request to **Kristofer Buren** for instrument setup.

---

## 20. Disaster Recovery

### Infrastructure Resilience

- **Dual data center** setup for redundancy.
- **CI/CD pipeline** from GIT enables rapid redeployment.
- **DBA team** handles database backups and restoration.

### Manual Fallback Procedure

If the Securities Borrowing system is completely unavailable:

1. **C&S verifies** loan needs by checking positions in AbaSec.
2. **C&S contacts SEB** directly via email or phone to arrange loans manually.
3. **C&S books loans manually** in AbaSec.

This manual process ensures that critical loan operations can continue even during a full system outage.
