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
- **Greece** - FTT on Greek equities

### How FTT-relevant instruments are determined

Each country publishes an official list of FTT-liable instruments, typically **once per year** (e.g., France publishes at: https://bofip.impots.gouv.fr/bofip/9789-PGP.html/identifiant=BOI-ANNX-000467-20211229). Data vendors like SIX process these official lists and deliver updated eligibility data before the first applicable trading day.

### Who pays FTT?

- **France, Spain, Italy:** The **custodian bank** must pay the FTT to the tax authorities. This means Nordnet (via its clearing chain) is responsible for calculating, collecting from the customer, and remitting.
- **Greece:** **Clearstream automatically handles** FTT Greek and debits the tax directly to the banks on their Clearstream account. No manual process needed by Nordnet.

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

## Tradegate Context

Lars Mattsson (Team Wolf) checked with Tradegate (Germany's largest OTC trading venue) in January 2026 about FTT data availability. Key findings from Thomas Gasinsky (Tradegate Service):

- FTT data is **not included** in Tradegate's instrument list file
- Tradegate confirmed FTT exists for France, Spain, Italy, and Greece
- Tradegate's own regulatory reporting team provided the information but does not distribute FTT data as a data product

**Implication:** FTT data must come from a dedicated data vendor (SIX or WM Daten), not from the trading venue itself.

### Tradegate Contacts

| Person | Role | Email |
|--------|------|-------|
| Thomas Gasinsky | Tradegate Service | tgasinsky@tradegate.de, +49 30 896 06-393 |
| Maksym Borachok | Tradegate | mborachok@tradegate.de |
| Support | General | support@tradegate.de |

## Impact on Vendor Decision

**Updated (July 2026):** SIX has confirmed FTT data covering 13+ countries with clean CSV format, pricing at 150k SEK/year base. SIX is the recommended vendor - see reference/ftt-six-vs-wm-daten-comparison.md for full comparison.
