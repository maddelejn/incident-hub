---
type: runbook
title: "New Listing in US Instruments"
category: instruments
service: admin-trading, instrument-admin, search
last_updated: 2026-07-10
---

# New Listing in US Instruments

**Context:** New US instruments are delivered by Millistream earlier in the day but are **kept blocked by default**. This prevents premature unblocking during ongoing corporate actions where shares haven't been delivered yet, which could lead to accidental short selling.

## Step 1: Trading Desk Review & Unblocking

- Every day at **11:30 CET**, an automated "US instrument report" is sent to the Trading Desk.
- Review the report to determine if the new listing should be unblocked or remain blocked.
- If safe to proceed, navigate to **Admin Trading** and manually unblock the instrument.

## Step 2: Monitor Search & Verification

After unblocking, the instrument must go through an automated search run before becoming visible to customers.

**Search run times (CET):**

| Time | Note |
|------|------|
| 08:01 | |
| 09:30 | |
| 10:30 | |
| 11:30 | During report review |
| **14:00** | **First run after 11:30 unblocking** |
| 15:45 | |
| 17:45 | |
| 19:40 | |

**Action:** Wait until **14:00-14:30 CET** after unblocking. Check web/app - the instrument should be fully searchable if correctly unblocked in Admin Trading.

## Step 3: Troubleshooting Checklist (Before Escalation)

If the instrument is still not appearing in search after **14:30 CET**, check the following before escalating:

- [ ] **Instrument Admin:** Is the instrument visible and existing in Instrument Admin?
- [ ] **Report:** Was the instrument included in today's "US instrument report" email?
- [ ] **Trading block:** Was the instrument successfully unblocked in Admin Trading?

## Step 4: Escalation Paths

| Scenario | Symptoms | Action |
|----------|----------|--------|
| **A** | Instrument exists in Instrument Admin, all checks pass, past 14:30 CET but not searchable | Escalate to **#area-customer-journey**, ping **@compass-be** |
| **B** | Instrument cannot be found anywhere in Instrument Admin | Escalate to **#area-securities-brokerage**, ping **@wolf-goalie** |

## Reference

- **Instrument Admin:** https://instrument-admin.tools.prod.nntech.io/Instruments
- **Vendor:** Millistream (delivers instruments earlier in the day)
- **Daily report:** "US instrument report" sent at 11:30 CET
