---
type: reference
title: "External Vendor Contact Information and Responsibilities"
last_updated: 2026-07-10
---

# External Vendor Contacts

## Quick Lookup: Who to Contact for Issues

| Vendor | Primary Contact for Issues | Notifications to marketdata? |
|--------|---------------------------|------------------------------|
| **Nasdaq** | emo@nasdaq.com + dataeurope@nasdaq.com (always include both) | No |
| **NGM** | support@ngm.se (general), listings@ngm.se (instrument listings) | No |
| **Euronext Oslo** | clientsupport@euronext.com | No |
| **Millistream** | tech@millistream.com | Yes |
| **Morningstar** | See detailed breakdown below | Yes |
| **Spotlight** | - | No |
| **SIX** | - | - |

## Vendor Details

### Nasdaq

- **European Market Operations:** emo@nasdaq.com
- **Market Data (always include for fast reply):** dataeurope@nasdaq.com
- **KAM - David Gerby:** David.Gerby@nasdaq.com / +46 73-449 65 36
- **Signup:** marketupgrades@nordnet.se → http://www.nasdaqomxnordic.com/News/marketnotices/Subscribe/

### NGM / Spotlight

- **General support:** support@ngm.se
- **Instrument listings:** listings@ngm.se
- **Signup:** marketupgrades@nordnet.se → https://status.ngm.se/

### Euronext Oslo

- **Client support:** clientsupport@euronext.com
- **Signup:** marketupgrades@nordnet.se → https://connect2.euronext.com/en/user/register

### Millistream

- **Tech support:** tech@millistream.com
- **Note:** No info on continuous upgrades - unclear if Nordnet needs to self-monitor.

### Morningstar

| Purpose | Contact |
|---------|---------|
| Realtime price issues | realtimehelpdesk@morningstar.com |
| Market data support | marketdatasupport@morningstar.com |
| Instrument issues | productsupport@morningstar.com |
| MDS issues | support.nordics@morningstar.com |
| Internal contacts | Daniel, Johan |
| Nordics support contact | Christel (from SC-002) |

**Important (from SC-002):** Always create a **new ticket** when raising issues with Morningstar. Never reply to old/closed tickets, especially near holiday periods. Include the exact API call/payload and authenticating user email.

### SIX

- Contact details TBD
- Sends Partnership Shares BULK daily delivery to Clearing and Settlements
- Nordnet currently pays SIX for partnership identification data

### Citi

- Contact details TBD

## Vendor Email Distribution

### Operational Emails

| Email | Purpose | Recipients |
|-------|---------|------------|
| Instrument Report (US/CA/DE) | New non-Nordic instruments for manual review by US Trading Desk + Corporate Actions | See: Dashboards for Non-Nordic Instrument Block Process |
| Partnership Shares BULK daily (SIX) | Workaround for identifying Partnerships. Forwarded to Clearing and Settlements | No Wolf process today |
| Max price gap exceeded | Alert Corporate Actions of suspected reverse split/split. Sends at 15:30 before NA market open | Corporate Actions |
| Spärra VP YYYY-MM-DD | Internal Nordnet email for corporate actions coordination | Corporate Actions, US Trading Desk |

### Vendor Notification Emails

| Vendor | Senders |
|--------|---------|
| **Morningstar** | marketdatasupport@morningstar.com, realtimehelpdesk@morningstar.com |
| **Millistream** | tech@millistream.com, Jonas Lundberg (jonas.lundberg@millistream.com), Mats Fors (mats.fors@millistream.com), Henrik Holst (henrik.holst@millistream.com), Peter Fors (peter.fors@millistream.com) |
| **SIX** | Commercial Notification |
| **NGM** | Philip Bäckman (Philip.Backman@ngm.se), NGM Support (support@NGM.SE), Stefan Cederlund (stefan.cederlund@ngm.se), Mats O Eklund (mats.o.eklund@ngm.se), Joel Boija (Joel.Boija@NGM.SE), ngm-dev@googlegroups.com |
| **Euronext** | oheiberg@euronext.com, tokristoffersen@euronext.com, MembersInfo (membersinfo@euronext.com), Client Services (clientsupport@euronext.com) |
| **Nasdaq** | operator@nasdaq.com, dataeurope@nasdaq.com |

## Responsibilities (Market Data Intake Owners)

- Subscribe to information from provider
- Understand where the relevant documentation is and how to get it
- Do first analysis and filtering of received communication (when in doubt, let through rather than filter out)
- Inform team of upcoming changes, including updating calendar
- If all responsible persons are on vacation/sick simultaneously, they must delegate to someone else
- Attend bi-weekly market upgrade meeting
