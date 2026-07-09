---
id: SC-002
title: "Missing Morningstar OnDemand EMT Data for Fund Order Placement"
date: 2026-07-09
status: pending
requester: "Internal (system alert)"
requester_team: ""
service: "daemon-intake-funds-morningstar1, fund-order-placement"
team_responsible: "Team Navigator, Team Wolf"
product_owner: "Team Navigator (Clara Montgomery), Team Wolf (Madde)"
category: integration
frequency: one-off
faq_candidate: true
tags: [morningstar, emt, mifid, fund-intake, bdi-mapper, regxchange, ondemand, swagger, manual-sync]
---

# SC-002: Missing Morningstar OnDemand EMT Data for Fund Order Placement

## Question / Problem

"Bdi mapper" lacked matches for "RegXchange" and "OnDemandMorningstar". This breaks the fund order placement system where EMT data is mandatory.

**Root cause is currently unknown.** Morningstar investigated their side and confirmed their API output was completely healthy at the time of failure, pointing to a potential issue on the internal Nordnet side. Team Navigator needs to investigate on-prem logs for `daemon-intake-funds-morningstar1` to determine why the automated intake failed to pull or map the data.

## Answer / Solution

**Immediate fix:** Manual full data sync via the service's Swagger UI (performed by Lars Mattsson, Team Wolf).

**Root cause:** Still open - Team Navigator to investigate.

## Steps to Resolve

1. Open Swagger UI for `daemon-intake-funds-morningstar1`:
   `http://daemon-intake-funds-morningstar1.prod.nordnet.se:8080/docs/swagger-ui/index.html`
2. **Fetch data:** Call the `fetchMifidData` endpoint to initiate a full download of OnDemand data from Morningstar for all funds (~2 hours).
   - [fetchMifidData endpoint](http://daemon-intake-funds-morningstar1.prod.nordnet.se:8080/docs/swagger-ui/index.html#/morningstar-controller/fetchMifidData)
3. **Process data:** Call the `processMifidData` endpoint to trigger data conversion and processing.
   - [processMifidData endpoint](http://daemon-intake-funds-morningstar1.prod.nordnet.se:8080/docs/swagger-ui/index.html#/morningstar-controller/processMifidData)
4. Verify that "Bdi mapper" now shows matches for "RegXchange" and "OnDemandMorningstar".

## Stakeholders

| Role | Person/Team | Status |
|------|-------------|--------|
| Product Owner (Intake) | Team Navigator (Clara Montgomery) | Root cause investigation open |
| Product Owner (Funds) | Team Wolf (Madde) | Informed |
| Manual Fix | Lars Mattsson (Team Wolf) | Data sync completed ✅ |
| External | Morningstar Support (Christel) | Confirmed API healthy on their side ✅ |

## Service Ownership

| Service | Owner |
|---------|-------|
| `daemon-intake-funds-morningstar1` | Team Navigator |

## References

- [fetchMifidData Swagger](http://daemon-intake-funds-morningstar1.prod.nordnet.se:8080/docs/swagger-ui/index.html#/morningstar-controller/fetchMifidData)
- [processMifidData Swagger](http://daemon-intake-funds-morningstar1.prod.nordnet.se:8080/docs/swagger-ui/index.html#/morningstar-controller/processMifidData)

## Notes

### Runbook: Manual Morningstar EMT Data Sync

If "Bdi mapper" fails to show matches for Morningstar EMT/MiFID data:

1. Go to `daemon-intake-funds-morningstar1` Swagger UI
2. Call `fetchMifidData` (full sync takes ~2 hours)
3. Call `processMifidData`
4. Verify Bdi mapper matches are restored

### Morningstar Support Process

- **Always create a completely new ticket** instead of replying to an old/closed one, especially near holiday periods.
- Include the **exact API call/payload** and the **authenticating user email** in the ticket.

## Open Items

- [ ] Team Navigator: Investigate on-prem logs to determine why automated intake failed
- [ ] Team Navigator: Determine if monitoring/alerting exists for this failure mode

## Related Cases

None yet. Watch for recurring Morningstar intake failures or Bdi mapper match issues.
