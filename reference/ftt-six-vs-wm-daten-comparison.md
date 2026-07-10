---
type: reference
title: "FTT Data Comparison: SIX vs WM Daten"
category: regulatory, tax, vendor-comparison
last_updated: 2026-07-10
---

# FTT Data Comparison: SIX vs WM Daten

## Key Finding

**SIX provides FTT data.** It covers France, Italy, Spain (the same three countries as WM Daten), plus Switzerland, Liechtenstein, UK, South Africa, Ireland, Finland, Hong Kong, Malaysia, Singapore, and Thailand. SIX has **broader geographic coverage** than WM Daten.

## SIX FTT Data Structure

SIX delivers FTT data as a **single CSV file** (semicolon-delimited) with clean, human-readable field names and enumerated values.

### Fields (15 data fields)

| # | CSV Header | Field Name | Description | Type |
|---|-----------|------------|-------------|------|
| 1 | `messageType` | Message Type | New / mutation / deletion | Enumeration |
| 2 | `instrumentStatus` | Instrument Status | Active or not | Enumeration |
| 3 | `swissValorNumber` | Swiss Valor Number | SIX national identifier | Integer |
| 4 | `ISIN` | ISIN | International identifier | String |
| 5 | `taxRaisingCountry` | Tax Raising Country | Which country raises the tax | Enumeration |
| 6 | `taxName` | Tax Name | Which FTT type applies | Enumeration |
| 7 | `taxApplicability` | Tax Applicability | Liable / exempt / conditionally exempt | Enumeration |
| 8 | `taxableObject` | Taxable Object | What is taxed (transaction, transfer) | Enumeration |
| 9 | `taxValidFrom` | Tax Valid From | Start date of tax applicability | Date |
| 10 | `taxValidTo` | Tax Valid To | End date of tax applicability | Date |
| 11 | `residentNonResidentCode` | Resident / Non-Resident | Whether tax applies to residents, non-residents, or all | Enumeration |
| 12 | `taxAmount` | Tax Amount | Tax rate (e.g., 0.4 = 0.4%) | Real |
| 13 | `descriptionTaxDetails` | Description Tax Details | Free text description (up to 850 chars) | String |
| 14 | `issuerDomicile` | Issuer Domicile | Country of issuer | Enumeration |
| 15 | `instrumentClassificationBySIX` | Instrument Classification | SIX instrument type | Enumeration |

### Tax Types Available (from sample data)

| Tax Name | Description |
|----------|-------------|
| Financial transaction tax (generally applicable) | Standard FTT (France, Italy, Spain) |
| Financial transaction tax (high frequency trading) | Italian HFT-specific tax |
| Financial transaction tax (regulated market or MTF) | Italian market-specific tax |
| Derivatives on shares (generally applicable) | Italian derivatives tax |
| Derivatives on shares (regulated market) | Italian derivatives on regulated markets |
| Derivatives on share yields/baskets/index | Italian derivatives on indices |
| Stamp duty | UK stamp duty |
| Taxable security in CH/LI - Equities | Swiss stamp tax on equities |
| Taxable security CH/LI - Debt instr./claims | Swiss stamp tax on debt |
| Taxable security in CH/LI - Fund unit | Swiss stamp tax on funds |
| Securities Transfer Tax (STT) | South Africa STT |

### Country Coverage (from sample)

| Country | Rows in sample | Tax Types |
|---------|---------------|-----------|
| Switzerland | 103 | Stamp tax (equities, debt, funds) |
| Liechtenstein | 103 | Same as Switzerland |
| United Kingdom | 55 | Stamp duty |
| Italy | 30 | FTT (general, HFT, regulated market, derivatives) |
| France | 28 | FTT (generally applicable) |
| Spain | 11 | FTT (generally applicable) |
| South Africa | 4 | Securities Transfer Tax |
| Ireland | 3 | Stamp duty |
| Finland | 3 | Tax applicable |
| Hong Kong | 2 | Stamp duty |
| Malaysia | 1 | Tax applicable |
| Singapore | 1 | Tax applicable |
| Thailand | 1 | Tax applicable |

### Sample Tax Rates

| Country | Tax Type | Rate |
|---------|----------|------|
| France | FTT (generally applicable) | 0.4% |
| Italy | FTT (generally applicable) | 0.4% |
| Italy | FTT (high frequency trading) | 0.04% |
| Italy | FTT (regulated market/MTF) | 0.2% |
| Spain | FTT (generally applicable) | 0.2% |

### Sample Data Row (French FTT)

```
ISIN=FR0000130213, Country=France, Tax=FTT (generally applicable),
Applicable=Liable, Rate=0.4%, Object=Transaction
```

## WM Daten FTT Data Structure

WM Daten provides 18 proprietary field codes. See reference/ftt-financial-transaction-tax.md for full list.

Key differences from SIX:
- Uses cryptic field codes (`GD432`, `KD610`, etc.) instead of readable names
- Fields are repeated with different prefixes (`KD`, `UD`, `VD`) for different instrument categories
- Only covers France, Italy, Spain (3 countries vs SIX's 13+)
- No sample data file provided yet

## Head-to-Head Comparison

| Aspect | SIX | WM Daten |
|--------|-----|----------|
| **Countries covered** | 13+ (FR, IT, ES, CH, UK, IE, FI, HK, etc.) | 3 (FR, IT, ES) |
| **Data format** | Clean CSV, readable field names, enumerations | Proprietary WM field codes (mapping required) |
| **Tax rate included** | Yes (numeric, e.g., 0.4) | TBD (field exists but no sample) |
| **Valid from/to dates** | Yes | TBD |
| **Resident/non-resident** | Yes | TBD |
| **Free text details** | Yes (850 chars) | TBD |
| **Instrument classification** | Yes (SIX classification) | Yes (WM classification) |
| **Delivery** | CSV snapshot file | Daily CSV (full file recommended) |
| **Sample data provided** | Yes (350 rows, 103 ISINs) | No sample yet |
| **Integration effort** | Low (standard CSV, clean headers) | High (proprietary codes, mapping layer needed) |
| **Existing vendor** | Yes (already providing sanctions data) | No |
