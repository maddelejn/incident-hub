---
type: product-knowledge
title: "Germany ETP Expansion - SIX as Alternative Vendor for KID, EMT & FTT Data"
status: in-progress
stakeholders: "Madde (PO Instrument Intake), Joachim Wegebrand (Head of ETP), Edward Neptune"
vendor: "SIX Financial Information"
vendor_contact:
  sales: "Carl Sundman (carl.sundman@six-group.com, +46 76 546 6457)"
last_updated: 2026-07-10
---

# Germany ETP Expansion: SIX as Alternative Vendor for KID, EMT & FTT Data

## Context

SIX Financial Information is being evaluated alongside WM Daten as a potential vendor for KID, EMT (Target Market & Costs and Charges) data for ETPs (ETNs/ETCs) on Xetra.

## Coverage Check Results

SIX completed a coverage check against Nordnet's instrument universe:

- **Total instruments:** 384 ISINs (after removing 24 duplicates from the submitted list)
- **Target Market:** 100% coverage (384/384)
- **Costs & Charges:** 100% coverage (384/384)
- **KIDs:** 379/384 covered. **5 ISINs missing KIDs:**
  - `XS3373438483`
  - `DE000A3GTML1`
  - `CH1327686056`
  - `XS3373414203`
  - `XS3269544030`
- SIX confirmed their sourcing team can easily add the 5 missing KIDs if Nordnet proceeds

## Additional Coverage

SIX also covers other markets beyond Xetra for these instrument types. Relevant since ETPs often trade on multiple exchanges.

## Timeline

| Date | Event |
|------|-------|
| 2026-07-02 | Madde reached out asking about KID/EMT for ETPs on Xetra |
| 2026-07-02 | Carl confirmed coverage available, requested instrument universe for check |
| 2026-07-02 | Madde sent instrument list |
| 2026-07-03 | Carl returned coverage results: 384 ISINs, 5 missing KIDs |
| 2026-07-03 | Follow-up meeting agreed for August after vacations |

## Vendor Contact

| Person | Role | Email | Phone |
|--------|------|-------|-------|
| Carl Sundman | Senior Sales Manager | carl.sundman@six-group.com | +46 76 546 6457 |

**Office:** SIX Financial Information, Grev Turegatan 30, Box 3117, SE-103 62 Stockholm
**Main:** +46 8 5861 6300
**Note:** Carl on vacation 4 weeks from July 7, reachable by phone for urgent matters.

## SIX vs WM Daten Comparison

| Aspect | SIX | WM Daten |
|--------|-----|----------|
| **Coverage (Target Market)** | 100% (384/384) | 100% (confirmed by Funda) |
| **Coverage (Costs & Charges)** | 100% (384/384) | 100% (confirmed by Funda) |
| **Coverage (KIDs)** | 379/384 (5 missing, can be added) | TBD (need to verify from Excel) |
| **Data format** | TBD | Proprietary WM Field Codes (requires mapping layer) |
| **Ease of integration** | TBD | Difficult - legacy German field codes, no standard EMT format |
| **FTT/Tax data** | TBD | Available (German tax flags for Abgeltungsteuer) |
| **Multi-market coverage** | Yes (beyond Xetra) | TBD |
| **Communication language** | Swedish (Stockholm office) | English/German |
| **Existing Nordnet relationship** | Yes (already provides Partnership Shares data) | New vendor |
| **Price** | TBD | Cheaper if buying subset of fields vs full product |

## Open Questions for August Meeting

- [ ] What data format does SIX deliver in? Standard EMT format or proprietary?
- [ ] Does SIX provide FTT/tax data for German instruments (Abgeltungsteuer flags)?
- [ ] What's the delivery mechanism (API, daily file, SFTP)?
- [ ] Pricing comparison vs WM Daten
- [ ] Can SIX deliver delta files (changes only) or full snapshots?
- [ ] Update frequency (daily, real-time, on-change)?
- [ ] Compare actual field-level data from both vendors' sample files

## Related Documents

- **products/germany-etp-vendor-wm-daten.md** - WM Daten as the alternative vendor
- **reference/vendor-contacts.md** - SIX already listed as existing vendor (Partnership Shares)
- **reference/emt-kid-priip-target-market-guide.md** - What the EMT/KID data actually contains
