---
type: runbook
title: "ETF Not Tradable & Order Rejections"
category: etf, orders
service: admin-trading, instrument-admin
last_updated: 2026-07-10
---

# Operating Procedure: ETF "Not Tradable" & Order Rejections

**Purpose:** Enables Nordnet employees to independently triage, troubleshoot, and resolve client inquiries regarding ETFs that are unsearchable or orders being rejected because of missing KID or EMT.

## Step 1: Locate the Root Cause in Admin Trading

Do not guess the cause. Look up the exact technical reason code.

1. Open **Admin Trading Order History** (bookmark this link).
2. Under the **Validation** tab, navigate to **Order History**.
3. Search by the client's **account number** or specific **OrderID**.
4. Identify the corresponding order.
5. Read the exact error message in the **"Reason" column**.
6. Proceed to the matching section below.

## Step 2: Error Triage & Resolution Matrix

---

### Category A: Regulatory & Data Document Issues

#### ETF_KIID_ERROR

**System meaning:** Nordnet is missing the regulatory Key Information Document (KID) in the customer's preferred local language (SE, DK, NO, FI, or DE).

**Trading restriction:** Only Professional Clients (signed Prof Elect ETF Agreement) can trade. Retail clients are strictly blocked.

#### EMT_DATA_ERROR

**System meaning:** Nordnet is missing the mandatory European MiFID Template (EMT) data file required to assess target market appropriateness.

#### Workflow for Category A:

1. Google the ETF's **ISIN**.
2. Navigate to the **official issuer's website** (e.g., iShares/BlackRock, Amundi, Invesco, Vanguard, SPDR).
3. Find sections titled "Countries of Registration," "Registered Locations," or "Countries of Distribution" (often shown as names or flags).
4. Evaluate:

**Path 1: Client's country/flag is NOT listed**

- **Diagnosis:** Intentional corporate distribution choice. The issuer does not legally distribute this fund in the client's region.
- **Action:** DO NOT ESCALATE. Close the inquiry.
- **Client script:**
  > "The issuer of this ETF has explicitly chosen not to register or distribute this specific fund in [Sweden/Norway/Denmark/Finland]. Because it is not registered for distribution here, European regulations prevent Nordnet from offering it to retail investors."

**Path 2: Flag IS listed + ETF is BRAND NEW (listed < 1 month)**

- **Diagnosis:** Issuer intends to distribute here, but the regulatory data package hasn't yet been sent to Morningstar by the issuer, or Morningstar hasn't made it available to Nordnet.
- **Action:** DO NOT ESCALATE. Inform client of standard data onboarding lag.
- **Client script:**
  > "The issuer does support our region, but because this is a newly launched ETF, we are not receiving the official regulatory data package that we need to distribute from the right paths. It typically takes a few weeks for new instruments to sync across the ecosystem. Please try again shortly."

**Path 3: Flag IS listed + ETF is OLD (listed > 1 month)**

- **Diagnosis:** Broken data pipe or ingestion error between Issuer, Morningstar, and Nordnet's internal databases.
- **Action:** ESCALATE via **#area-securities-brokerage**.
- **Required ticket details:** ISIN, Error Code, Link to Issuer Page showing local registration, confirmation that the ETF is > 1 month old.

---

### Category B: Manual Blocks & Account Controls

#### MINIMIZING_RISK_ORDERS_ONLY

**System meaning:** A manual risk restriction is active (`active_buy = false`). Clients can sell down positions, but all buy orders are blocked.

**Visibility:** Instrument is hidden from Nordnet's public front-end search.

#### ORDER_INSTR_BLOCKED

**System meaning:** A total manual block is active (`isBlocked = true`). Neither buying nor selling is possible.

**Visibility:** Instrument is hidden from Nordnet's public front-end search.

#### Workflow for Category B:

1. Check the **listing currency** on Nordnet.se.
2. Identify the **exchange** it trades on and its **trading currency**.
3. Evaluate:

**Path 1: XETRA (German) ETF trading in a non-EUR currency (USD, GBP, etc.)**

- **Diagnosis:** Our executing broker does not support trading XETRA-listed ETFs in any currency other than EUR. The manual block is permanent and intentional.
- **Action:** DO NOT ESCALATE. Close the inquiry.
- **Client script:**
  > "Nordnet offers ETF trading on the XETRA (German) exchange exclusively in Euros (EUR). We currently do not support non-EUR currency listings for these specific instruments. However, there is generally an equivalent EUR-denominated listing of this exact same ETF available on our platform that you can trade instead."

**Path 2: XETRA ETF trading in EUR OR trades on a different exchange**

- **Diagnosis:** Manual block is unrelated to broker currency limitation. Could be tied to an active corporate action, custom risk profile, or exchange-level halt.
- **Action:** ESCALATE TO TRADING DESK (not the Business Owner).
- **Required ticket details:** Account number or OrderID, ISIN, Error Code (`MINIMIZING_RISK_ORDERS_ONLY` or `ORDER_INSTR_BLOCKED`), confirmed Exchange/Currency.

---

## Quick-Reference Routing Summary

```
[Customer reports ETF issue]
       │
       ▼
Check Admin Trading "Reason" Column
       │
       ├──► ETF_KIID_ERROR / EMT_DATA_ERROR
       │     │
       │     ├──► Flag missing on Issuer Site ──────────► NO ESCALATION
       │     │     (Tell client: Not distributed in their region)
       │     │
       │     ├──► Flag present + ETF < 1 month old ─────► NO ESCALATION
       │     │     (Tell client: Data vendor lag, try again in a few weeks)
       │     │
       │     └──► Flag present + ETF > 1 month old ─────► ESCALATE
       │           → #area-securities-brokerage
       │           (Include: ISIN, error code, issuer page link)
       │
       └──► MINIMIZING_RISK_ORDERS_ONLY / ORDER_INSTR_BLOCKED
             │
             ├──► XETRA in USD/GBP/etc. ────────────────► NO ESCALATION
             │     (Tell client: Only EUR supported on XETRA)
             │
             └──► XETRA in EUR / Other Exchange ────────► ESCALATE
                   → Trading Desk
                   (Include: account/orderID, ISIN, error code, exchange/currency)
```

## Related Incidents

- **INC-002** (Missing Morningstar ETF target_market Data and KID): ETF_KIID_ERROR / EMT_DATA_ERROR caused by ISIN missing from RegXchange on Morningstar's side. Path 3 case - vendor data gap for an established ETF.
- **INC-003** (Dual-Listed Virtune ETP): EMT_DATA_ERROR caused by internal pipeline mapping failure for dual-listed ETPs. Path 3 case - internal ingestion bug.
