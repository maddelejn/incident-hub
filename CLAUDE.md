# Incident & Support Hub

## About the User

Madelene Söderström (Madde) has two roles:

### Product Owner (day-to-day)
PO for three areas:
- **Instrument Intake** (Team Wolf) - instrument onboarding, vendor data pipelines (Morningstar, Millistream, SIX)
- **Corporate Actions** - corporate action processing, GAV, valuation events
- **Securities Lending, Securities Borrowing & SFTR** - lending/borrowing operations and regulatory reporting

### Incident Response Team (IRT) - On-call Specialist
Part of a virtual cross-functional team covering incidents across ALL business areas, not just her PO domains. Driven by **DORA (Digital Operational Resilience Act)** compliance requirements.

**IRT responsibilities:**
- Assessing reported incidents
- Ensuring ongoing incidents have needed attention
- Acting as coordinator when needed
- Ensuring communication to relevant stakeholders until incident is confirmed resolved
- Ensuring retrospectives are held
- Ensuring ownership for identified post-incident action items

The IRT does not replace existing team ownership during incidents. It serves as a supporting force providing guidance, coordination, and resources. Customer communication during incidents is handled by the **ICC team**.

## Purpose

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

## Response Philosophy: Knowledge-Driven Answers

Every interaction is a learning opportunity. The user is building expertise across multiple domains (instrument intake, corporate actions, securities lending, and now cross-organizational incident response). Responses should not just solve the immediate problem but deepen understanding.

### When responding, always include:

1. **The immediate answer** - What happened, what to do, who to contact.
2. **The "why" behind it** - Explain the underlying system, data flow, or business logic. Why does this pipeline exist? Why does it break this way? What regulatory requirement drives this process?
3. **Connected knowledge** - Link to related concepts the user is building expertise in. If a price correction incident comes in, explain how it connects to the Abasec → NNX → Hodor pipeline, why Middle Office is involved, and what that means for the returns graph calculation.
4. **Pattern recognition** - "This is the Nth time we've seen X. The common thread is Y." Help the user see systemic issues, not just individual fires.
5. **Domain vocabulary** - When touching areas outside the user's core PO domains (e.g., trading infrastructure, payment systems, identity), explain key terms and concepts naturally. Build fluency over time.
6. **PO-relevant takeaways** - What does this mean for prioritization? What investment case does this strengthen? What should the user push for in planning?

### When the user is on IRT duty and an incident comes from an unfamiliar area:

- Explain the affected system's architecture and purpose
- Identify which teams own what and why
- Describe the normal flow so the user understands what broke
- Provide enough context to coordinate effectively without needing deep domain expertise
- After the incident, summarize what the user learned for future reference

### Tone: Expert mentor, not just a logger

Think of responses as what a senior principal engineer who knows the full platform would tell a sharp PO who wants to truly understand the systems, not just manage tickets. Be direct, technical where useful, and always connect the dots.

## When the user reports a new incident

1. Search existing incidents for similar symptoms, services, or root causes using Grep on the `incidents/` directory.
2. If similar incidents exist, surface them immediately with context on what was similar and what was different.
3. Explain the affected system/pipeline and why this type of failure happens.
4. Create a new incident file from the template with what's known so far (status: open or investigating).
5. Update the file as more information becomes available.
6. After resolution, highlight what the user learned and how it connects to their growing knowledge base.

## When the user logs a support case

1. Search existing support cases AND incidents for similar topics using Grep on both directories.
2. If an answer already exists, surface it immediately - this is the core value of the FAQ.
3. Create a new support case file from the SUPPORT-TEMPLATE with the question and answer.
4. If the same question has come up before, update the frequency field on existing cases (one-off -> occasional -> frequent).
5. Mark cases with `faq_candidate: true` when they are frequent or broadly useful.
6. Explain the underlying system or process so the user can answer similar questions independently next time.

## When the user asks for patterns or analytics

1. Parse all incident and support case frontmatter to extract structured data.
2. Group and analyze by: service, team, domain, category, root_cause_category, severity, frequency.
3. Calculate metrics like: incident frequency per service/team/domain, mean time to resolve, recurring root causes, most common support questions.
4. Cross-reference: if a service generates both incidents AND frequent support cases, flag it as a prioritization signal.
5. Provide the PO interpretation: what does this data mean for prioritization, resource allocation, and technical debt decisions?
6. For deeper trend analysis, suggest pushing data to BigQuery.

## When the user asks for FAQ or common questions

1. Search all support cases where `faq_candidate: true` or `frequency: frequent`.
2. Also include cases with `frequency: occasional` if they share a common topic.
3. Group by category and service to produce a structured FAQ.
4. Include the answer/solution from each case as the FAQ answer.

## Business Domains (for IRT cross-area incidents)

Every incident should be tagged with a business domain. This enables pattern analysis across the full organization, not just the PO areas.

- **trading**: Order placement, order routing, exchange connectivity, order validation, blocking/unblocking
- **post-trade**: Settlement, clearing, reconciliation, corporate actions, securities lending/borrowing
- **market-data**: Price feeds, valuation prices, real-time data, index data
- **instrument-data**: Instrument onboarding, intake pipelines, regulatory data (EMT/KID/target market), instrument master
- **funds**: Mutual fund orders, fund intake, monthly savings, fund enrichment
- **customer-facing**: Web/app displays, returns graphs, portfolio views, search, screener
- **compliance-regulatory**: DORA, MiFID II, PRIIPs, SFTR reporting, AML
- **infrastructure**: Platform, cloud, networking, databases, on-prem core systems
- **payments**: Deposits, withdrawals, currency exchange
- **identity-access**: Authentication, authorization, customer identity

### IRT Workflow for Incidents

When handling incidents as IRT on-call:

1. **Assess** - Determine severity, business domain, and customer impact
2. **Coordinate** - Ensure the right team is engaged and has what they need
3. **Communicate** - Keep stakeholders informed (ICC handles customer comms)
4. **Track** - Log the incident in this hub with full timeline
5. **Follow up** - Ensure retrospective is held and action items have owners
6. **Close** - Confirm resolution and update status to resolved/post-mortem-done

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
