---
id: SC-006
title: "INET Gateway Decommission - New Instrument Creation Flow for Stockholm/First North"
date: 2026-07-14
status: resolved
requester: "Lars Mattsson (Team Wolf)"
requester_team: "Team Wolf"
service: "instrument-intake, instrument-admin, abasec"
team_responsible: "Team Wolf"
product_owner: "Team Wolf (Madde)"
category: configuration
frequency: one-off
faq_candidate: false
tags: [inet-gateway, decommission, instrument-creation, tracker, bull-bear, ptc, kbu, kbe, stockholm, first-north, abasec-requests]
---

# SC-006: INET Gateway Decommission - New Instrument Creation Flow

## Context

Lars Mattsson (Team Wolf) announced a new instrument creation flow going live July 14, 2026 (after market close). This is part of the INET gateway decommission - moving instrument creation from the legacy INET flow to a new flow.

## What's Changing

### Markets going live
- Nasdaq Stockholm (XSTO, Market ID 11)
- First North Sweden (FNSE, Market ID 53)
- First North Norway

### Instrument types in scope
| Type | Description |
|------|-------------|
| PTC | Tracker certificates |
| KBU | Bull certificates |
| KBE | Bear certificates |

These are structured products (bull/bear certificates and trackers) - typically high-volume instrument creation by issuers.

## Monitoring

**Track new creations:** https://instrument-admin.tools.prod.nntech.io/AbasecRequests

**How to identify instruments from the new flow:**
- Filter by country code
- Currently only Xetra and Expand market instruments exist
- New INET Sweden instruments will have `.SE` in the search name

## Notes

- Volume expected to be low initially (only a few created on the test day)
- Admin GUI still needs tweaking and improvements
- Lars and William Sörensen are monitoring

## Related

- This is part of the broader INET gateway decommission project
- Instrument Admin: https://instrument-admin.tools.prod.nntech.io/Instruments
