# Incident & Support Hub

This is a structured knowledge base for incidents and support cases. When working in this project, follow these conventions:

## Structure

- `incidents/` - One markdown file per incident, named `INC-XXX-short-description.md`
- `support-cases/` - One markdown file per support case, named `SC-XXX-short-description.md`
- `TEMPLATE.md` - Template for new incidents. Always copy this when creating a new incident.
- `SUPPORT-TEMPLATE.md` - Template for new support cases. Always copy this when creating a new support case.
- `reference/` - Reference data (market IDs, MIC codes, team ownership, etc.). Consult these when triaging incidents.
- `runbooks/` - Step-by-step procedures for common operational tasks. Surface these when the user asks "how do I fix X?"
- Both incidents and support cases use YAML frontmatter for structured metadata and markdown body for details.

## Numbering

- Incident IDs are sequential: INC-001, INC-002, etc. Check the highest existing ID before creating a new one.
- Support case IDs are sequential: SC-001, SC-002, etc. Check the highest existing ID before creating a new one.

## When the user reports a new incident

1. Search existing incidents for similar symptoms, services, or root causes using Grep on the `incidents/` directory.
2. If similar incidents exist, surface them immediately - this helps during triage.
3. Create a new incident file from the template with what's known so far (status: open or investigating).
4. Update the file as more information becomes available.

## When the user logs a support case

1. Search existing support cases AND incidents for similar topics using Grep on both directories.
2. If an answer already exists, surface it immediately - this is the core value of the FAQ.
3. Create a new support case file from the SUPPORT-TEMPLATE with the question and answer.
4. If the same question has come up before, update the frequency field on existing cases (one-off -> occasional -> frequent).
5. Mark cases with `faq_candidate: true` when they are frequent or broadly useful.

## When the user asks for patterns or analytics

1. Parse all incident and support case frontmatter to extract structured data.
2. Group and analyze by: service, team, category, root_cause_category, severity, frequency.
3. Calculate metrics like: incident frequency per service/team, mean time to resolve, recurring root causes, most common support questions.
4. Cross-reference: if a service generates both incidents AND frequent support cases, flag it as a prioritization signal.
5. For deeper trend analysis, suggest pushing data to BigQuery.

## When the user asks for FAQ or common questions

1. Search all support cases where `faq_candidate: true` or `frequency: frequent`.
2. Also include cases with `frequency: occasional` if they share a common topic.
3. Group by category and service to produce a structured FAQ.
4. Include the answer/solution from each case as the FAQ answer.

## Severity levels

- **P1**: Complete service outage or data loss affecting many users. Requires immediate response.
- **P2**: Major feature degraded, significant user impact. Requires urgent response.
- **P3**: Minor feature issue, limited user impact. Respond during business hours.
- **P4**: Cosmetic or low-impact issue. Address in normal workflow.

## Categories

- **infrastructure**: Cloud platform, networking, DNS, certificates
- **deployment**: Failed deploys, rollback needed, CI/CD issues
- **data-pipeline**: ETL failures, data quality, missing data, pipeline delays
- **security**: Unauthorized access, vulnerabilities, credential exposure
- **performance**: Latency, timeouts, resource exhaustion
- **configuration**: Wrong config values, feature flags, environment mismatch
- **integration**: Third-party API failures, upstream/downstream breaks
- **monitoring**: Alert fatigue, missing alerts, false positives

## Root cause categories

- **code-bug**: Logic error, race condition, null pointer, etc.
- **config-change**: Incorrect or unvalidated configuration change
- **capacity**: Resource limits hit (CPU, memory, disk, connections)
- **dependency-failure**: External service or library failure
- **human-error**: Manual mistake during operations
- **infrastructure**: Hardware failure, cloud provider issue
- **unknown**: Root cause not determined

## Support case categories

- **access**: Permission issues, role requests, access to environments or tools
- **configuration**: Help with settings, environment variables, feature flags
- **how-to**: General "how do I do X?" questions about services or processes
- **data-question**: Questions about data, reports, dashboards, metrics
- **bug-report**: User-reported bugs that aren't incidents (low impact, edge cases)
- **onboarding**: New team member setup, documentation pointers
- **integration**: Questions about connecting services, APIs, data flows

## Support case frequency

- **one-off**: Asked once, unlikely to recur
- **occasional**: Comes up every now and then
- **frequent**: Recurring question, strong FAQ candidate

## When the user pastes a raw Slack thread

The user will paste raw Slack threads directly instead of pre-formatting them. When this happens:

1. Extract: timeline, people involved, error messages, services affected, resolution steps, and links.
2. Determine if it's an Incident (service disruption, customer impact) or Support Case (question, missing data, process issue).
3. Cross-reference against existing incidents and support cases for patterns.
4. Create the structured file with all extracted context.
5. Always identify the responsible tech team, product owner, and escalation path.

## Technical Architecture: Key Data Pipelines

### Backdated Price Corrections (Abasec → NNX → Hodor)
When data fixes need to go back in time:
1. **Middle Office** updates the price in **Abasec** (source of truth for historical prices)
2. **Team Marlin** publishes corrected price data from Abasec to **NNX**
3. **Team Hodor** fetches from NNX to heal customer-facing graphs and balances

This pipeline is used for: wrong valuation prices, fictitious instrument pricing, corporate action pricing errors.
It is NOT used for: static metadata fixes (instrument names, regulatory data), on-prem core database fixes (those go via Coresys).

### ETF/Fund Regulatory Data (Morningstar → fund-enrichment → Platform)
```
Morningstar (RegXchange / MDS feed)
    ↓
fund-enrichment / morningstar-fund-enrichment (Team Navigator)
    ↓
EnrichedFundTopic.V2 (Kafka topic)
    ↓
fund-bff → AboutETP component
    ↓
service-instrument aggregate endpoint (target_market, costs_and_charges)
```

### Morningstar Fund Reference Data (M* file → intake → instrument master)
```
Morningstar daily file (Basic Reference Data CSV)
    ↓
mstar-fund-overview-intake (Team Compass / Team Wolf)
    ↓
Instrument Master → Orderbooks → Search
```
**Known recurring failure:** M* file contains deletions → intake deletes instruments → orderbooks removed → orders fail. Three incidents (INC-006, INC-007, INC-008).

### Instrument Name Corrections
```
Millistream (vendor correction) → Instrument Admin → Nordnet App & Web
```

## Key Services and Owners

| Service | Owner | Purpose |
|---------|-------|---------|
| Abasec | Middle Office | Source of truth for historical prices and transactions |
| NNX (Nordnet X) | - | Central data platform, receives price data from Abasec |
| instrument-master | Team Wolf | Core instrument database |
| instrument-admin | Team Wolf | Internal admin UI for instrument lookup |
| admin-trading | - | Order history, validation, blocking/unblocking |
| mstar-fund-overview-intake | Team Compass / Team Wolf | Morningstar fund reference data intake |
| morningstar-fund-enrichment | Team Navigator | Morningstar ETF/fund enrichment (costs, target market, KID) |
| fund-bff | Team Navigator | Fund backend-for-frontend |
| daemon-intake-funds-morningstar1 | Team Navigator | Morningstar mutual fund EMT/MiFID data intake (on-prem) |
| service-instrument | - | Instrument aggregate API (target_market, costs_and_charges) |
| search | Team Compass | Instrument search (runs on schedule, needs market reload after fixes) |

## Key Slack Channels for Escalation

| Channel | Use For |
|---------|---------|
| #area-securities-brokerage | ETF regulatory data issues (KID/EMT), instrument intake issues. Ping @wolf-goalie |
| #area-customer-journey | Instrument not showing in search after unblocking. Ping @compass-be |
| #valuation-prices-on-nnx | Notify Team Marlin to publish corrected prices from Abasec to NNX |
| #low-pressure | Active incident coordination |
| #disco | Instrument visibility issues |

## Key Internal Tools

| Tool | URL | Purpose |
|------|-----|---------|
| Instrument Admin | https://instrument-admin.tools.prod.nntech.io/Instruments | Look up instruments, check intake rows, secondary identifiers |
| Admin Trading | Order History → Validation tab | Check order rejection reason codes |
| service-instrument aggregate | http://service-instrument.prod.nordnet.se/aggregate/instrument/isin_code/{ISIN}?expand=target_market | Verify target market / regulatory data |

## Morningstar Identifier Patterns

| Pattern | Type | Purpose |
|---------|------|---------|
| `F0000...` | Fund/Share Class ID | High-level fund umbrella identifier |
| `0P000...` | SecID / Performance ID | Required by enrichment service for costs & charges |
| `0C000...` | Management Company ID | Fund provider identification |

When enrichment skips an instrument due to "missing secId or fundShareClassId", check the Secondary identifiers block in Instrument Admin for the `0P...` ID.
