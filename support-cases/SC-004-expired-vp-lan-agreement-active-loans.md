---
id: SC-004
title: "Expired VP-lån Agreement with Active Loans Still Running"
date: 2026-07-14
status: resolved
requester: "Marcus Moore (via Slack)"
requester_team: "Operations"
service: "securities-borrowing"
team_responsible: "Operations / Clearing & Settlement"
product_owner: "Team Wolf (Madde)"
category: how-to
frequency: occasional
faq_candidate: true
tags: [securities-borrowing, vp-lan, expired-agreement, loan-return, seb, abasec, broker-loan]
---

# SC-004: Expired VP-lån Agreement with Active Loans Still Running

## Question / Problem

Marcus Moore reported a customer with an expired "VP-lån avtal" (securities borrowing agreement, expired June 3rd) who still has two active loans (SAAB + BOL). The customer is paying interest on loans they don't need. The admin wouldn't return the loans once the agreement was expired.

## Answer / Solution

The loans can be returned manually through a two-step process:

### Steps to Return Loans with Expired Agreement

1. **Return in Securities Borrowing Admin:**
   - Go to **SEB Loans → Return Loan Quantity**
   - Enter the **ISIN**, **quantity**, and **settlement date**
   - Press Return
   - SEB must accept the return

2. **Return in Abasec:**
   - Match the return in Abasec and close it manually

**Important:** Both steps are required. The admin sends the return request to SEB, and Abasec needs to be updated separately.

### Who Can Do This

| Step | Who |
|------|-----|
| Return in admin (SEB loan) | Operations (Marcus), Trading Desk cannot do it if no broker loans available for return |
| Return in Abasec | Operations (Marcus) |

### Resolution Timeline

| Time | Event |
|------|-------|
| 09:40 | Marcus reported expired agreement with active loans |
| 09:51 | Kristofer Burén confirmed loans should be returned if customer is not short |
| 10:25 | Erik Sedenberg (Trading Desk) confirmed no broker loans available for return - must be done manually by Operations |
| 10:37 | William Sörensen provided the admin steps: SEB Loans → Return Loan Quantity |
| 11:55 | Marcus confirmed loans returned and SEB accepted |
| Next day | No error emails - confirmed resolved |

## Key People

| Person | Role |
|--------|------|
| Marcus Moore | Operations - reported and fixed the issue |
| Kristofer Burén | SME - confirmed loans should be returned |
| Erik Sedenberg | Trading Desk - confirmed manual process needed |
| William Sörensen | Developer - provided admin instructions |
| Ricard Kindfalk | Developer - initially looked into it |

## Notes

- The agreement expired on **June 3rd** - the loans had been running for 6+ weeks unnecessarily with the customer paying interest
- The admin system doesn't automatically return loans when an agreement expires - this is a manual process
- Trading Desk (brokers) cannot return these loans through their normal interface - only Operations can via the admin's manual return flow

## Open Questions

- Should the system alert when a customer has active loans but their VP-lån agreement has expired?
- Should there be an automated process to return loans when agreements expire?
- How many other customers might have this same situation?

## Related Documents

- **Securities Borrowing operational doc:** products/securities-lending/securities-borrowing.md (Section: Broker Loans, Loan Statuses)
- **Troubleshooting guide:** runbooks/securities-financing-troubleshooting.md
