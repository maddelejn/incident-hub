---
id: SC-005
title: "Danish Loan Return Stuck Requests - Zero Quantity Retry Loop"
date: 2026-07-14
status: pending
requester: "Internal (error email alert)"
requester_team: "Team Shark"
service: "securities-borrowing"
team_responsible: "Team Shark"
product_owner: "Madde"
category: bug-report
frequency: one-off
faq_candidate: false
tags: [securities-borrowing, danish-loans, seb, return-loan, zero-quantity, rejected, book-loan-task, force-book-loan-task, stuck-requests]
follow_up: "Hampus Holmertz after vacation"
---

# SC-005: Danish Loan Return Stuck Requests - Zero Quantity Retry Loop

## Problem

Error email "Error when processing loan responses" has been firing daily for 3+ working days (since July 9). The system is trying to return 3 Danish loans to SEB with **quantity 0**, which SEB rejects. The failed requests are stuck in the queue and retry every day.

```
ISIN: DK0060636678
Market: XCSE (Copenhagen)
Currency: DKK

Affected SEB Loans:
- 7681080: Partially active (969 original → 690 remaining)
- 7686484: Already returned ✅ (but stuck 0-qty request still retrying)
- 7682384: Already returned ✅ (but stuck 0-qty request still retrying)
```

## Error Email

```
Subject: Error when processing loan responses
From: noreply-securities-borrowing-PROD@nordnet.se
To: SecuritiesBorrowingSystem@nordnet.se
Firing since: July 9, 2026

Failed to process return loan response:
isin             mic   currency  quantity  sebOrderType  valueDate   fee  account  sebLoanId  status    error
DK0060636678     XCSE  DKK       0         Firm          2026-07-14  0.5  NORDNET  7681080    Rejected  null
DK0060636678     XCSE  DKK       0         Firm          2026-07-14  0.5  NORDNET  7686484    Rejected  null
DK0060636678     XCSE  DKK       0         Firm          2026-07-14  0.5  NORDNET  7682384    Rejected  null
```

## Root Cause (Suspected)

The Danish loan return flow has special logic (`BookLoanTask` / `ForceBookLoanTask`) that books returns proportionally. The proportional calculation likely produced **0 quantity** for these three loans, creating invalid return requests that SEB rejects. The requests are stuck in the queue and retry daily.

## Investigation Findings (July 14)

| SEB Loan ID | Status in Admin | Notes |
|-------------|----------------|-------|
| 7681080 | Active | 969 original, 690 remaining (279 returned) |
| 7686484 | Returned | Fully returned, but 0-qty request still retrying |
| 7682384 | Returned | Fully returned, but 0-qty request still retrying |

Two loans resolved themselves through normal flow. One remains partially active. All three have stuck 0-quantity return requests that will never succeed.

## Action Items

| Action | Owner | Status | Due Date |
|--------|-------|--------|----------|
| **Follow up with Hampus after vacation** - clear stuck 0-quantity requests from DB | Hampus Holmertz | Pending | August |
| Investigate why BookLoanTask/ForceBookLoanTask calculated 0 quantity for Danish returns | Hampus / Shark devs | Pending | August |
| Check if loan 7681080 (690 remaining) needs manual return or is still needed | Kristofer Burén / Operations | Open | TBD |
| Consider adding validation: don't send return requests with quantity 0 to SEB | Shark devs | Open | TBD |

## Interim Mitigation

The error email will keep firing daily until the stuck requests are cleared. Options:

1. **Acknowledge alarm via Swagger** (silences but doesn't fix):
   - `ScheduleController → acknowledgeLastExecution → "FileDownLoadAndProcessTask"`
2. **Wait for Hampus** to clear the stuck transactions from the DB after vacation

## Related

- **SC-004** (Expired VP-lån with active loans) - also a loan lifecycle issue requiring manual intervention
- **Securities Borrowing docs** (products/securities-lending/securities-borrowing.md) - Section 11: Danish Loan Returns, Section 19: Known Problems
- **Troubleshooting guide** (runbooks/securities-financing-troubleshooting.md) - Danish returns handled by BookLoanTask/ForceBookLoanTask
