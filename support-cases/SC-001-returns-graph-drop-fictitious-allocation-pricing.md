---
id: SC-001
title: "Drop in Returns Graph Due to Fictitious Allocation Share Pricing"
date: 2026-07-09
status: resolved
requester: "Customer (via support)"
requester_team: ""
service: "returns-graph, price-publishing"
team_responsible: "Team Hodor, Team Marlin"
product_owner: "Corporate Actions (Madde)"
category: data-question
frequency: occasional
faq_candidate: true
tags: [ipo, allocation-share, pricing, abasec, nnx, returns-graph, fictitious-instrument, transaction-type-k]
jira: "https://nordnetbank.atlassian.net/browse/APP-3724"
---

# SC-001: Drop in Returns Graph Due to Fictitious Allocation Share Pricing

## Question / Problem

Customer saw a sudden drop in their returns graph for **Biosergen AB** between **June 9th and June 22nd, 2021**.

During an IPO, an Allocation share was booked as a trade at 10 SEK on June 9th (transaction type "K" / Buy). However, the market value of this temporary fictitious instrument was set to **0 SEK** in Abasec. Buying an asset with "0 value" caused the returns graph to plummet until June 22nd, when the allocation share was properly exchanged for the actual share (which had the correct market value).

## Answer / Solution

A retroactive data fix is required across three systems in sequence:

**Data flow:** Middle Office (Abasec) → Team Marlin (push to NNX) → Team Hodor (fetch from NNX → graph heals)

## Steps to Resolve

1. **Middle Office (Mathias):** Update the price retroactively to 10 SEK (starting June 8th/9th) inside Abasec. ✅ Done
2. **Team Marlin:** Push/publish the updated price data from Abasec to NNX. ✅ Done
3. **Team Hodor:** Fetch the corrected data from NNX - graph will automatically heal once correct prices are available. ✅ Done (auto-healed)

## Stakeholders

| Role | Person/Team | Status |
|------|-------------|--------|
| Product Owner | Corporate Actions (Madde) | Informed |
| Middle Office | Mathias | Price updated in Abasec ✅ |
| Price Publishing | Team Marlin | Price pushed to NNX ✅ |
| Graphs/Transactions | Team Hodor (Josef) | Graph healed ✅ |

## References

- **Jira:** https://nordnetbank.atlassian.net/browse/APP-3724
- **Transaction Admin:** Transaction Search
- **Holdings Admin:** Graph Debugger

## Notes

IPOs handled via transaction type "K" (Buy) using temporary allocation shares must have their initial fictitious instrument price set correctly in Abasec (not left at 0). If the price is left at 0, the returns graph will show a false drop for the period between allocation booking and the exchange to the real share.

If this is missed, the fix requires a manual data sync through the full pipeline: Middle Office corrects price in Abasec → Marlin publishes to NNX → Hodor fetches from NNX and graph heals automatically.

## Related Cases

None yet - first logged case. Watch for similar cases involving IPO allocation shares and returns graph drops.
