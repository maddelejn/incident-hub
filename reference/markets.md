---
type: reference
title: "Market Reference Data - MIC Codes, Market IDs, and Types"
last_updated: 2026-07-14
---

# Market Reference Data

MIC (Market Identifier Codes) follow ISO standards. Market IDs are internal Nordnet classifications used for instrument classification and order validation. Every market has submarkets with Sub MICs ("Segment IDs") and corresponding IDs (e.g., XSTO has 17 submarkets, one being OMX STO Warrants with sub ID 4692).

## Sweden

| # | Name | MIC | Market ID | Trading | Legal Type | Financial Type |
|---|------|-----|-----------|---------|------------|----------------|
| 1 | Nasdaq Stockholm (inc Genium INET, Nasdaq derivative market NASN/NASNNO) | XSTO/NASN | (11 & 40)/(12 & 47) | Electronic | RM/OTF | Capital market, Derivatives market (OM) |
| 2 | Nasdaq First North Growth Market Stockholm | FNSE | 53 | Electronic | MTF | Capital market, Derivatives market (NSDX) |
| 3 | Nordic Growth Market (NGM) inc NDX Sweden/Finland/Norway/Denmark | XNGM | 13 & 35/36/37/48 | Electronic | RM/OTF | Capital market (NGM Equity & Debt Securities), Derivatives market (NDX Sweden) |
| 5 | NGM Nordic MTF Sweden/Finland/Denmark/Norway | NMTF | 61/62/63/64/65 | Electronic | MTF | Capital market, Derivatives market |
| 6 | Spotlight Stockmarket (former Aktietorget) | XSAT | 52 | Electronic | MTF | Capital market |
| 7 | beQuoted | N/A | N/A | Manual | OTC | Capital market |
| 8 | Pepins (former Alternativa aktiemarknaden) | N/A | N/A | Manual | N/A | Capital market |
| 9 | Unlisted stock electronic registered at Euroclear Sweden | N/A | N/A | Manual | N/A | Capital market |

## Norway

| # | Name | MIC | Market ID | Trading | Legal Type | Financial Type |
|---|------|-----|-----------|---------|------------|----------------|
| 10 | Oslo Børs | XOSE | 15 | Electronic | RM/OTF | Capital market, Derivatives market |
| 11 | Oslo Axess | XOSE | 15 | Electronic | RM/OTF | Capital market |
| 12 | Merkur Market | MERK | 49 | Electronic | MTF | Capital market |
| 13 | NOTC (Norway OTC) | N/A | N/A | Manual | OTC | Capital market |
| 14 | Unlisted stock electronic registered at Verdipapirsentralen | N/A | N/A | Electronic | OTC | Capital market |

## Denmark

| # | Name | MIC | Market ID | Trading | Legal Type | Financial Type |
|---|------|-----|-----------|---------|------------|----------------|
| 15 | Nasdaq Copenhagen inc Nasdaq derivative market | XCSE/NASNDK | 14/23 | Electronic | RM/OTF | Capital market, Derivatives market (OM) |
| 16 | Nasdaq First North Denmark | FNDK | 55 | Electronic | MTF | Capital market, Derivatives market |
| 17 | Spotlight Stockmarket (former Aktietorget) | XSAT | 67 | Electronic | MTF | Capital market |
| 18 | Unlisted stock electronic registered at VP Securities | N/A | N/A | Manual | N/A | Capital market |

## Finland

| # | Name | MIC | Market ID | Trading | Legal Type | Financial Type |
|---|------|-----|-----------|---------|------------|----------------|
| 19 | Nasdaq Helsinki inc Genium INET | XHEL | 24/42 | Electronic | RM/OTF | Capital market, Derivatives market |
| 20 | Nasdaq First North Helsinki | FNFI | 54 | Electronic | MTF | Capital market, Derivatives market |
| 21 | Unlisted stock electronic registered at Euroclear Finland | N/A | N/A | Manual | N/A | Capital market |

## Germany

| # | Name | MIC | Market ID | Trading | Legal Type | Financial Type |
|---|------|-----|-----------|---------|------------|----------------|
| 22 | Xetra | XETR | 4 | Electronic | RM/OTF | Capital market, Derivatives market |
| 34 | Tradegate Exchange | XGAT/XGRM | 85 | Electronic | RM | Capital market |

### Tradegate / Xetra Fungibility

Tradegate and Xetra instruments sharing the same ISIN are **fungible** - they represent the exact same position at the clearing house. A customer can buy on one venue and sell on the other without a position transfer. This is different from dual-listing (e.g. INC-003), where separate instrument lines exist per market.

The system models this via `tradableMappings` in the instrument-id-mapper, where a single instrument has order book entries for both Market ID 4 (Xetra) and Market ID 85 (Tradegate).

**Access restriction:** Tradegate is restricted to **German customers only**. Non-German customers cannot buy, sell, or search Tradegate instruments. When both Xetra and Tradegate are available, Xetra is the preferred venue (Tradegate hidden from search).

**Known issue:** When instruments are delisted from Xetra Freiverkehr (XETB) but remain active on Tradegate, non-German customers holding old Xetra positions may lose electronic trading access. See SC-007.

## USA

| # | Name | MIC | Market ID | Trading | Legal Type | Financial Type |
|---|------|-----|-----------|---------|------------|----------------|
| 23 | Nasdaq | XNAS | 19 | Electronic | RM/OTF equivalent | Capital market, Derivatives market |
| 24 | New York Stock Exchange inc NYSE American (XASE) | XNYS/XASE | 17/18 | Electronic | RM/OTF equivalent | Capital market, Derivatives market |
| 25 | OTC Markets (limited selection) | XOTC/OOTC | 20/21 | Electronic | Not applicable | Capital market |

## Canada

| # | Name | MIC | Market ID | Trading | Legal Type | Financial Type |
|---|------|-----|-----------|---------|------------|----------------|
| 26 | Toronto Stock Exchange | TO | 25 | Electronic | RM/OTF equivalent | Capital market, Derivatives market |
| 27 | Toronto Venture Exchange | V | 26 | Electronic | MTF equivalent | Capital market |

## Europe (Other)

| # | Name | MIC | Market ID | Trading | Legal Type | Financial Type |
|---|------|-----|-----------|---------|------------|----------------|
| 28 | London Stock Exchange (Only FTSE350) | XLON | 70 | Electronic | RM/OTF | Capital market |
| 29 | Euronext Paris | XPAR | 72 | Electronic | RM | Capital market |
| 30 | Euronext Lisbon | XLIS | 73 | Electronic | RM | Capital market |
| 31 | Euronext Amsterdam | XAMS | 74 | Electronic | RM | Capital market |
| 32 | Euronext Brussels | XBRU | 75 | Electronic | RM | Capital market |
| 33 | Euronext Dublin | XDUB | 76 | Electronic | RM | Capital market |

## Quick Lookup: Market ID → Name

| Market ID | Market |
|-----------|--------|
| 4 | Xetra |
| 11 | Nasdaq Stockholm |
| 12 | Nasdaq Stockholm Derivatives (NASN) |
| 13 | NGM |
| 14 | Nasdaq Copenhagen |
| 15 | Oslo Børs / Oslo Axess |
| 17 | NYSE |
| 18 | NYSE American (XASE) |
| 19 | Nasdaq US |
| 20 | OTC Markets (XOTC) |
| 21 | OTC Markets (OOTC) |
| 23 | Nasdaq Copenhagen Derivatives (NASNDK) |
| 24 | Nasdaq Helsinki |
| 25 | Toronto Stock Exchange |
| 26 | Toronto Venture Exchange |
| 35 | NGM NDX Sweden |
| 36 | NGM NDX Finland |
| 37 | NGM NDX Norway |
| 40 | Nasdaq Stockholm (Genium INET) |
| 42 | Nasdaq Helsinki (Genium INET) |
| 47 | Nasdaq Stockholm Derivatives (NASNNO) |
| 48 | NGM NDX Denmark |
| 49 | Merkur Market |
| 52 | Spotlight Stockmarket (SE) |
| 53 | First North Stockholm |
| 54 | First North Helsinki |
| 55 | First North Denmark |
| 61 | NGM Nordic MTF Sweden |
| 62 | NGM Nordic MTF Finland |
| 63 | NGM Nordic MTF Denmark |
| 64 | NGM Nordic MTF Norway |
| 65 | NGM Nordic MTF (additional) |
| 67 | Spotlight Stockmarket (DK) |
| 70 | London Stock Exchange |
| 72 | Euronext Paris |
| 73 | Euronext Lisbon |
| 74 | Euronext Amsterdam |
| 75 | Euronext Brussels |
| 76 | Euronext Dublin |
| 78 | Funds |
| 85 | Tradegate Exchange |

## Incident-Relevant Market IDs

Markets that have appeared in incidents and may need reloading during fixes:

| Market ID | Market | Incident | Context |
|-----------|--------|----------|---------|
| 11 | Nasdaq Stockholm | INC-001 | Reloaded after stuck instrument deregistration |
| 21 | OTC Markets (OOTC) | INC-001 | Reloaded after stuck instrument deregistration |
| 78 | Funds | INC-008 | Reloaded in search after M* fund data revert |
