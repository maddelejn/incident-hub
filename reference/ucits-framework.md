---
type: reference
title: "UCITS Regulatory Framework"
category: regulatory
last_updated: 2026-07-10
---

# UCITS - Undertakings for Collective Investment in Transferable Securities

Created by the EU in 1985. Governs **90%+ of all retail mutual funds and ETFs** structured in Europe.

## The EU Passport

Before UCITS, selling a fund cross-border required regulatory approval in each country individually. UCITS introduced the **EU Passporting rule:**

> If a fund is structured and approved under UCITS rules in one EU country (very often Luxembourg or Ireland), it is legally allowed to be marketed and sold to retail investors in all other EU member states without needing extra local fund approvals.

This is why most iShares ETFs are domiciled in Ireland but tradeable across all Nordnet markets.

## The 4 Pillars of UCITS

### 1. The 5/10/40 Diversification Rule

Forced diversification - a UCITS fund can never be wiped out by a single holding:

- **Max 10%** of fund assets in shares issued by a single company
- **All holdings above 5%** cannot exceed **40%** of total fund value combined
- Result: the fund structurally cannot crash because one stock in its portfolio went bankrupt

### 2. High Liquidity (The Exit Door)

Retail investors must be able to get their money back quickly:

- Investors must be allowed to redeem/sell shares **at least twice a week**
- Most funds offer **daily liquidity**
- The fund **cannot lock up** consumer cash for months or years

### 3. Eligible Assets Only

UCITS funds can only invest in highly liquid, regulated financial assets:

| Allowed | Banned |
|---------|--------|
| Listed stocks | Direct physical real estate |
| Government/corporate bonds | Physical commodities (gold bars) |
| Cash and money market instruments | Crypto assets |
| Regulated ETFs | Unregulated hedge funds |

### 4. Independent Custody

- The **fund manager** (who picks the stocks) must be a **completely separate company** from the **Custodian/Depositary Bank** (who physically holds the assets)
- If the fund management company goes bankrupt, retail investors' assets are **safe and untouched** at the independent depositary bank
- This is the core investor protection that structured products (ETCs/ETNs) do NOT have

## UCITS vs Non-UCITS: Impact on Instrument Intake

| Aspect | UCITS = True | UCITS = False |
|--------|-------------|---------------|
| **Product types** | Mutual funds, most ETFs | ETCs, ETNs, structured products |
| **Retail access** | Automatic green light for retail investors | Requires additional target market checks |
| **Document type** | KIID (legacy) / PRIIPs KID | PRIIPs KID only |
| **Cost data fields** | "Costs — Funds" bucket | "Costs — Structured Products" bucket |
| **Investor protection** | Highest (segregated assets, independent custody) | Lower (debt instrument, issuer credit risk) |
| **Data vendor (current)** | Morningstar covers most | WM Daten needed for German-listed products |
| **EU passport** | Yes - sell across all EU from one approval | No passport - separate registration needed |

## How This Connects to the Platform

```
Instrument enters intake pipeline
         │
         ▼
    Is it UCITS?
         │
         ├── YES ──► Retail safe by default
         │           Use "Costs — Funds" fields
         │           KIID/KID document required
         │           Morningstar typically provides data
         │           EU passport = available across all Nordnet markets
         │
         └── NO ───► Check target market carefully
                     Use "Costs — Structured Products" fields
                     PRIIPs KID required
                     WM Daten needed for Xetra products
                     No passport = check country registration
```

## Related Hub Documents

- **reference/emt-kid-priip-target-market-guide.md** - The EMT/KID data that UCITS funds must provide
- **products/germany-etp-vendor-wm-daten.md** - WM Daten provides the non-UCITS (ETC/ETN) data for Germany
- **runbooks/etf-not-tradable-order-rejections.md** - ETF_KIID_ERROR often relates to missing UCITS disclosure documents
