---
type: product-knowledge
title: "CSLA Steering Committee - October 2025 (with Minutes)"
status: completed
date: 2025-10
source: "2025_10 Nordnet Final (Minutes Included).pdf"
last_updated: 2026-07-13
---

# CSLA Steering Committee - October 2025

## Context

Steering committee meeting covering the migration from direct Citi integration to the CSLA (Sharegain) intermediated model for pension lending. This was the foundational step before retail lending could be considered.

## Key Topics

### Migration Progress
- Migration of pension lending from direct Citi SFTP to new Sharegain-intermediated flow
- Nordnet sending availability calculations to both Citi and Sharegain
- Loan instructions now coming from Sharegain instead of directly from Citi
- Sharegain handles borrower matching and allocation

### Operational Model
- Sharegain acts as the technology layer between Nordnet and Citi
- Citi remains the custodian and settlement counterparty
- MT54X message format unchanged for settlement

### Program Performance
- Review of lending revenue and program metrics
- Discussion of market coverage and expansion opportunities

### Technical Integration Status
- SFTP connectivity to Sharegain established
- File format alignment completed
- Reconciliation between Nordnet, Sharegain, and Citi views being monitored

## Relationship to Retail Project

The pension migration to CSLA was the prerequisite for retail lending:
1. **Pension migration (completed):** Move from direct Citi → CSLA-intermediated
2. **Retail launch (planned):** Build new NNX service using CSLA's per-client capabilities

The same tri-party model (Nordnet/Sharegain/Citi) applies to both, but retail adds per-customer tracking, opt-in management, and customer-facing transparency.
