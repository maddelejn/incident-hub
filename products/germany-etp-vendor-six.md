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

## FTT Pricing (from Carl Sundman, May 5 2026)

| Tier | Instruments | Cost |
|------|-------------|------|
| Base | Up to 9,000 instruments | 150,000 SEK/year (minimum) |
| 9,001 - 12,000 | Per instrument/year | 15.88 SEK |
| 12,001 - 15,000 | Per instrument/year | 15.36 SEK |
| 15,001 - 18,000 | Per instrument/year | 14.67 SEK |
| 18,001 - 21,000 | Per instrument/year | 13.81 SEK |
| 21,001 - 24,000 | Per instrument/year | 12.94 SEK |
| 24,001 - 27,000 | Per instrument/year | 12.08 SEK |
| 27,001 - 30,000 | Per instrument/year | 11.22 SEK |
| 30,001 - 33,000 | Per instrument/year | 10.36 SEK |
| 33,001 - 36,000 | Per instrument/year | 9.49 SEK |
| 36,001 - 39,000 | Per instrument/year | 8.97 SEK |
| 39,001+ | Per instrument/year | 8.63 SEK |

**Calculation example:** 12,500 instruments = 150,000 + (3,000 × 15.88) + (500 × 15.36) = **205,320 SEK/year**

**Note:** Instrument count is based on instruments in the output file that are FTT-relevant, not total universe.

Delivery via **SIX Flex** (most cost-effective option).

## FTT Technical Details (from Carl Sundman, April 30 2026)

### Q&A with SIX

| Question | SIX Answer |
|----------|-----------|
| **Coverage** | See factsheet - comprehensive multi-jurisdiction |
| **Instrument identification** | Marked on ISIN level. For ES/FR/IT, official lists published yearly are processed by dedicated teams and delivered before first trading day |
| **Tax rates provided?** | Yes - classifications and applicable FTT rates included |
| **Tax currency** | Corresponds to the currency of the asset (instrument) |
| **Netting rules** | Data indicates information is pertinent to "Transactions" |
| **Alternative models (floor/ceiling, tiered)?** | No |
| **Delivery methods** | SIX VDF, SIX Flex (file), SIX API, SIX Data Cloud (coming) |
| **Delivery modes** | Bulk "push" (all FTT instruments) or request/reply (securities of interest) |

### Nordnet contact for FTT pricing
- **Peter Engman** (peter.engman@nordnet.se) - received pricing from Carl

## Open Questions for August Meeting

- [ ] What data format does SIX deliver EMT/KID data in? Standard EMT format or proprietary?
- [ ] Pricing for EMT/KID/Target Market data (separate from FTT pricing above)
- [ ] Combined pricing if buying FTT + EMT/KID from same vendor?
- [ ] Compare actual EMT field-level data from SIX vs WM Daten

## Related Documents

- **products/germany-etp-vendor-wm-daten.md** - WM Daten as the alternative vendor
- **products/germany-expansion-vendor-strategy.md** - Consolidated vendor strategy
- **reference/ftt-six-vs-wm-daten-comparison.md** - Field-level FTT comparison
- **reference/ftt-financial-transaction-tax.md** - WM Daten FTT fields
- **reference/vendor-contacts.md** - SIX already listed as existing vendor
