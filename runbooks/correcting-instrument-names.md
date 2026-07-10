---
type: runbook
title: "Correcting Instrument Names (Stocks & ETFs)"
category: instruments
vendor: millistream
service: instrument-admin
last_updated: 2026-07-10
---

# Correcting Instrument Names (Stocks & ETFs)

**Context:** Instrument names displaying incorrectly on the Nordnet platform are primarily the result of a corporate action, such as a name change and/or a merger.

## Step 1: Verify the Correct Name

Investigate and confirm the exact correct name using authoritative sources:

| Source | Reliability |
|--------|------------|
| SIX | Authoritative |
| Company/Issuer documentation | Authoritative |
| Bloomberg | Authoritative |
| Infront | Authoritative |
| Google | Secondary |
| Gemini / AI tools | **Do NOT use as basis for vendor requests** - risk of hallucinations |

**Important:** Only request vendor updates based on verified, authoritative sources.

## Step 2: Locate the Instrument in Instrument Admin

1. Navigate to **Instrument Admin:** https://instrument-admin.tools.prod.nntech.io/Instruments
2. Search for the instrument by name, ISIN, or Symbol.
3. Open the instrument and find the corresponding **Intake instrument**.
4. Look for sources originating from **Millistream**, followed by the instrument ticker (e.g., Tesla = `TSLA` for XNAS, `TL0` for XETRA).
5. Open the specific intake and click on **Intake order book**.
6. Verify that the **Currency** and **MIC** perfectly match the market instrument you're investigating.
   - A stock can have the same ticker and currency across multiple exchanges - make sure the MIC matches (e.g., `XNAS` or `XNYS` for US stocks).
7. Locate the **ID under External Identifier** and copy it.

## Step 3: Contact Millistream

Send the following email:

```
To: tech@millistream.com
Cc: marketdata@nordnet.se, usahandel@nordnet.se
Subject: Incorrect name for [ISIN], Ins Ref [External Identifier Id]

Hello Millistream,

We have noticed that the Long Name for Ins Ref [External Identifier Id]
([ISIN]) appears to be outdated or incorrect.

From our perspective, the correct Long Name should be [Correct Long Name].
Could you please review this on your end and apply the correction if you agree?

Thank you in advance.
```

## Step 4: Platform Synchronization

Once Millistream applies the correction:

```
Millistream (vendor correction) → Instrument Admin → Nordnet App & Web
```

**Note:** Synchronization is not instant. Allow time for the changes to propagate.

## Reference

- **Instrument Admin:** https://instrument-admin.tools.prod.nntech.io/Instruments
- **Vendor:** Millistream (tech@millistream.com)
- **Data flow:** Millistream → Instrument Admin → Platform
