---
type: reference
title: "Financial Transaction Tax (FTT) - Data Fields and Vendor Details"
category: regulatory, tax
vendor: "WM Daten"
last_updated: 2026-07-10
---

# Financial Transaction Tax (FTT) Data

## What is FTT?

Financial Transaction Tax is a tax levied on the purchase of certain financial instruments. Different EU countries have implemented their own versions:

- **France (FFTT)** - Tax on purchases of shares in French companies with market cap > €1B
- **Italy (IFTT)** - Tax on shares of Italian companies + derivatives
- **Spain (SFTT)** - Tax on purchases of shares in Spanish companies with market cap > €1B

When a Nordnet customer buys a financial instrument that's subject to FTT in any of these countries, Nordnet must calculate and collect the correct tax. This requires knowing, per instrument:
- Is it FTT-relevant? (relevance flag)
- Which country's FTT applies? (France, Italy, Spain)
- Is there an exemption? (some transactions are exempt)
- What's the tax basis? (how to calculate the tax amount)

## WM Daten FTT Fields

WM Daten provides 18 FTT-related fields. They're organized in three layers:

### Relevance flags (which country's FTT applies)

| WM Field | German Name | English Name | Purpose |
|----------|------------|--------------|---------|
| `GD432` | Relevanz FFTS | FFTT Relevance | Is this instrument subject to **French** FTT? |
| `GD432A` | Relevanz IFTS | IFTT Relevance | Is this instrument subject to **Italian** FTT? |
| `GD649` | Relevanz SFTS | SFTT Relevance | Is this instrument subject to **Spanish** FTT? |

### Country-specific tax rates/details

| WM Field | English Name | Purpose |
|----------|--------------|---------|
| `GV428` | France Financial Transaction Tax | French FTT details/rate |
| `GV433` | Italian Financial Transaction Tax | Italian FTT details/rate |
| `GV648` | Spain Financial Transaction Tax | Spanish FTT details/rate |

### Detailed tax classification (repeated for different instrument types: K, U, V)

Each of these 4 fields appears with three prefixes (`KD`, `UD`, `VD`), likely representing different instrument categories in WM's data model:

| Field Suffix | German Name | English Name | Purpose |
|-------------|------------|--------------|---------|
| `x10` | Kennzeichen Finanztransaktionssteuer | Financial Transaction Tax Identifier | FTT classification flag |
| `x11` | Steuerpflicht FTS | FTT Liability | Is there a tax obligation? |
| `x12` | Steuerbefreiung FTS | FTT Exemption | Is this instrument exempt? |
| `x13` | Bemessungsgrundlage FTS | FTT Levy Basis | How is the tax calculated? |

Full field list with prefixes:

| KD (prefix) | UD (prefix) | VD (prefix) |
|------------|------------|------------|
| `KD610` - FTT Identifier | `UD610` - FTT Identifier | `VD610` - FTT Identifier |
| `KD611` - FTT Liability | `UD611` - FTT Liability | `VD611` - FTT Liability |
| `KD612` - FTT Exemption | `UD612` - FTT Exemption | `VD612` - FTT Exemption |
| `KD613` - FTT Levy Basis | `UD613` - FTT Levy Basis | `VD613` - FTT Levy Basis |

## WM Daten Delivery Options for Universe Data

WM Daten offered two delivery approaches (discussed in Feb 2026 meeting):

| Option | Format | Description | WM Recommendation |
|--------|--------|-------------|-------------------|
| **A** | Daily CSV | Complete list of all active ISINs daily | Recommended - easier to reconcile |
| **B** | XML delta | Only new ISINs, added incrementally | More error-prone (WM's view) |

WM Daten recommended option A (daily full file) over deltas, as reconciling two complete lists is less error-prone than maintaining a running list with additions.

## ISM (International Sanctions Monitoring)

WM Daten also provides sanctions data (separate from FTT):

| File | Content |
|------|---------|
| `92002` | Sanctioned instruments |
| `92003` | Sanctioned issuers |
| `92004` | Sanctioned instruments not in WM universe |

More details: https://www.wmdatenservice.com/wp-content/uploads/2024/10/wmdatenservice_flyer_ism_en.pdf

## WM Daten Contacts (FTT specific)

| Person | Role | Email |
|--------|------|-------|
| Vadim Peters | Technical/Product | V.Peters@wmdaten.com |
| Andreas Knabe | Sales (can create pricing offer) | A.Knabe@wmdaten.com |
| Klappenberger | Involved (role TBD) | (in email thread) |

## Nordnet Contacts (FTT project)

| Person | Role |
|--------|------|
| Madde | PO Instrument Intake |
| Pierre | Co-lead on FTT vendor selection |
| Peter Dahlsten | Technical (Team Wolf) |
| Edward Neptune | Involved |
| Anders Furby | Involved |

## Impact on Vendor Decision

This is significant new information for the SIX vs WM Daten comparison:

**WM Daten has detailed FTT data covering France, Italy, and Spain** with 18 specific fields for tax classification, liability, exemption, and levy basis calculation. This is exactly what Team Match needs for cost and charges calculations when Nordnet customers trade instruments subject to foreign FTT.

**Key question for SIX:** Do they provide equivalent FTT fields? If not, Nordnet may need WM Daten specifically for FTT regardless of the ETP/EMT decision.

**Possible outcome:** Use SIX for EMT/KID/Target Market (better coverage, easier integration) + WM Daten for FTT tax data (authoritative source as German NNA). Or find that one vendor covers everything.
