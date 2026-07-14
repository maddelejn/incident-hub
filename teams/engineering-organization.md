---
type: team-knowledge
title: "Nordnet Engineering Organization - All Teams"
status: active
total_engineers: ~195
last_updated: 2026-07-14
---

# Nordnet Engineering Organization

## Areas Overview

```
┌─────────────────────────────────────────────────────────────┐
│                   Customer Journey (32)                      │
├───────────┬──────────────┬──────────────┬───────────────────┤
│  Wealth   │  Securities  │  Credit &    │     Pension       │
│  Mgmt     │  Brokerage   │  Payments    │                   │
│  (11)     │  (43)        │  (24)        │     (16)          │
├───────────┴──────────────┴──────────────┴───────────────────┤
│                   Product Platform (44)                      │
├─────────────────────────────────────────────────────────────┤
│                  Technology Platform (25)                    │
└─────────────────────────────────────────────────────────────┘
```

---

## Customer Journey Area (32 engineers)

Customer-facing experience and engagement teams.

| Team | Focus | Key for IRT |
|------|-------|-------------|
| **Cats** | Social Investment Experience | |
| **Sparkles** | Customer Facing AI | |
| **Compass** | Inspire, watchlists, news | Owns search - escalate instrument visibility issues here |
| **Activation** | Get me started | Owns suitability/knowledge tests (MiFID II) |
| **Firefly** | Your Nordnet Life in the Mobile App | |
| **Ping** | Customer communications | ICC-related, customer messaging during incidents |
| **Falcon** | Portfolio and Analytics | Portfolio views, returns graphs |

---

## Wealth Management Area (11 engineers)

Mutual funds and partner business.

| Team | Focus | Key for IRT |
|------|-------|-------------|
| **FROG** | Mutual Funds Platform | Fund orders, monthly savings |
| **NAVigators** | Guide & Enable Mutual Funds | Fund enrichment, Morningstar intake (ETF data) |
| **ALPACA** | Partner Business | Partner web |

---

## Securities Brokerage Area (43 engineers)

Trading, instruments, market data, and securities financing. **This is Madde's area.**

| Team | Focus | Key for IRT |
|------|-------|-------------|
| **Postify** | Post Trade | Settlement, clearing |
| **ET** | Exchange Trading | Trade execution, market connectivity |
| **GOAT** | Trading order flow | Order routing |
| **Pre-Trade** | Order validation | Trading power, cash reservation, order validation |
| **Shark** | Securities Lending/Borrowing & Corporate Actions | Madde's team - splitting into CA + SecFinance |
| **Digital Assets** | Digital Assets | Crypto |
| **MAD** | Feeds & APIs | Market data feeds |
| **Marlin** | Prices | Price publishing, valuation prices (Abasec → NNX) |
| **Mint** | Equity data & pages | ETF/ETP data, EMT/target market for exchange-traded products |
| **Wolf** | Real Time Instruments | Instrument intake, Millistream, Morningstar reference data |

---

## Credit & Payments Area (24 engineers)

Lending, payments, and money movement.

| Team | Focus | Key for IRT |
|------|-------|-------------|
| **Wall-e** | Margin Lending | |
| **Nook** | Mortgages & Interest | |
| **Payment Platform** | Payment infrastructure | |
| **Money Movers** | Transfer Money | |
| **CREAM** | Money Inflow | Deposits |
| **FX** | FX stuff | Currency exchange |

---

## Pension Area (16 engineers)

Insurance and pension products.

| Team | Focus | Key for IRT |
|------|-------|-------------|
| **PROvide** | Vital Insurance Data Ensured | |
| **PROfit** | Future Investment Technology | |
| **PROud** | Pension UI Development | |
| **PROmise** | Move Insurance Super Easy | |

---

## Product Platform Area (44 engineers)

Shared platform services used across all product areas.

| Team | Focus | Key for IRT |
|------|-------|-------------|
| **Stork** | Customer enrollment | New customer onboarding |
| **Swoosh** | Private+ customer onboarding | Premium customers |
| **Customer Info** | Customers & accounts | Account data, customer domain |
| **Watchmen** | Customer risk | AML, risk management |
| **Zap** | Account & signing | |
| **Perm** | Authorization & Agreements | Access control |
| **LILO** | Login & Logout | Authentication |
| **FAB** | Fee & Billing | |
| **MATCH** | Commission & Segmentation | Costs & charges calculations |
| **HODOR** | Holdings | Holdings, graphs, returns, position management |
| **Ledger** | Ledger | Accounting |

---

## Technology Platform Area (25 engineers)

Infrastructure and developer tooling.

| Team | Focus | Key for IRT |
|------|-------|-------------|
| **CD** | Continuous Delivery | CI/CD pipelines |
| **Cloud Squad** | Cloud Infrastructure | GCP, NNX platform |
| **CMS** | Content Delivery | |
| **Shell** | Web Platform | |
| **Rocket** | App Platform | Mobile app infrastructure |
| **Agents** | Engineering Productivity | |
| **BAP/SRE** | Backend platform & Reliability | Core infrastructure, SRE |

---

## Quick Lookup: Who Owns What

### For Incident Triage

| Issue | Team | Area | Slack Channel |
|-------|------|------|---------------|
| Instrument not showing in search | **Compass** | Customer Journey | #area-customer-journey |
| Instrument intake / Morningstar data | **Wolf** | Securities Brokerage | #area-securities-brokerage |
| ETF/ETP regulatory data (EMT/KID/target market) | **Mint** + **NAVigators** | Sec Brokerage + Wealth Mgmt | #area-securities-brokerage |
| Price data / valuation prices | **Marlin** | Securities Brokerage | #valuation-prices-on-nnx |
| Holdings graph / returns | **HODOR** | Product Platform | |
| Order placement / routing | **GOAT** | Securities Brokerage | |
| Order validation / trading power | **Pre-Trade** | Securities Brokerage | |
| Trade execution | **ET** | Securities Brokerage | |
| Post-trade / settlement | **Postify** | Securities Brokerage | |
| Securities lending/borrowing | **Shark** | Securities Brokerage | |
| Corporate actions | **Shark** | Securities Brokerage | |
| Fund orders / monthly savings | **FROG** | Wealth Management | |
| Fund enrichment / Morningstar ETF | **NAVigators** | Wealth Management | |
| Customer communications | **Ping** | Customer Journey | |
| Costs & charges / commissions | **MATCH** | Product Platform | |
| Fee & billing | **FAB** | Product Platform | |
| Login / authentication | **LILO** | Product Platform | |
| Customer data / accounts | **Customer Info** | Product Platform | |
| Deposits / money inflow | **CREAM** | Credit & Payments | |
| Payments infrastructure | **Payment Platform** | Credit & Payments | |
| FX / currency | **FX** | Credit & Payments | |
| Mobile app issues | **Firefly** + **Rocket** | Customer Journey + Tech Platform | |
| CI/CD / deployment issues | **CD** | Technology Platform | |
| Cloud / GCP infrastructure | **Cloud Squad** | Technology Platform | |
| Platform reliability | **BAP/SRE** | Technology Platform | |
| Portfolio / analytics | **Falcon** | Customer Journey | |
| ETF screener | **Mint** or **Compass** | Securities Brokerage / Customer Journey | Check with both |

### Teams Connected to Madde's Products

| Madde's Product | Primary Team | Key Collaborating Teams |
|----------------|-------------|----------------------|
| Instrument Intake | **Wolf** | Compass (search), Mint (ETF data), NAVigators (fund enrichment), Marlin (prices) |
| Corporate Actions | **Shark** | HODOR (holdings/graphs), Pre-Trade (trading power), ET (trade execution), Postify (settlement) |
| Securities Lending | **Shark** | HODOR (position display), MATCH (compensation), Postify (settlement) |
| Securities Borrowing | **Shark** | Pre-Trade (shorting validation), ET (trade execution), GOAT (order flow) |
| SFTR | **Shark** | HODOR (transaction data) |
| Germany Expansion (FTT/EMT) | **Wolf** + **Mint** | MATCH (costs & charges), NAVigators (fund data) |
