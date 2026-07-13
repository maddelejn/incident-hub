---
type: product-knowledge
title: "How Securities Lending Works - Plain Language Summary"
source: "https://nordnetbank.atlassian.net/wiki/spaces/DE/pages/80834635"
last_updated: 2026-07-13
---

# How Securities Lending Works (Plain Language)

**Source:** [Confluence](https://nordnetbank.atlassian.net/wiki/spaces/DE/pages/80834635)

## The Business in 5 Steps

1. **Nordnet lends shares to Citi** (pension/insurance accounts - Kapitalförsäkring)
2. **Citi sells those shares** to borrowers (their clients)
3. **Borrowers pay Citi** for the shares
4. **Citi gives Nordnet 85%** of the revenue (keeps 15%)
5. **Nordnet gives customers 50%** of its share (keeps 50%)

### Who Gets the Money?

- Customers in the lending program with Kapitalförsäkring
- Only if their compensation exceeds 1 SEK
- Some large customers opt out of sharing profit on specific shares

### Revenue Split

```
Borrower pays 100%
    │
    ├── Citi keeps 15%
    │
    └── Nordnet receives 85%
         │
         ├── Nordnet keeps 42.5% (half of 85%)
         │
         └── Customer receives 42.5% (half of 85%)
```

## Process of Lending (Nightly)

1. **Each night:** Calculate availability
   - Fetch all accounts in program and their positions
   - Sum up total shares available
   - **Cap:** Each customer can contribute max 10% of total
   - Send availability to Citi

2. **Citi takes up loans** based on the availability sent

3. **Nordnet books the loan in Abasec**
   - The loan is between Nordnet and Citi (not Citi's end clients)

4. **Settlement:** T+2 (settles after 2 business days)

## Process of Getting Paid

1. **Each morning:** Receive file of all open loans from yesterday
2. **Calculate compensation:**
   - Take 85% of loan values
   - Distribute 50% of that to customers
   - Calculation based on Run Day yesterday -2 values
3. **Citi pays Nordnet:** Monthly (for previous month)
4. **Nordnet pays customers:** Quarterly (first month of each quarter, summing previous quarter's daily compensation)

## Compensation Payout (Current Automation Project)

Previously manual process (Kristofer creates Excel files → Jonathan at Payouts books in Abasec via "Anything Goes").

Now being automated:
- System generates compensation files
- Duality checks and approvals before payout
- Covers NPAB (Sweden, Finland) and NLAS (Norway)
- Finland recently added via Finnish wrapper
- Denmark: life insurance coming

**Anything Goes limitation:** Cannot handle more than 2,000 rows at a time, so files were split into many smaller ones (one reason for automation).
