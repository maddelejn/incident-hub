---
type: product-knowledge
title: "Service-Corporate-Action - Complete Documentation"
status: active
source: "https://nordnetbank.atlassian.net/wiki/spaces/DE/pages/80835485"
owning_team: "Team Shark"
product_owner: "Madelene Söderström"
platform: "on-prem"
database: "Oracle (ORAP), schema SORC"
service_url: "http://service-corporate-action.prod.nordnet.se"
source_code: "https://scm.prod.nordnet.se/projects/TP/repos/corporate-action/browse"
last_updated: 2026-07-13
---

# Service-Corporate-Action - Complete Documentation

**Source:** [Confluence - Corporate Action](https://nordnetbank.atlassian.net/wiki/spaces/DE/pages/80835485)

## 1. Description

Handles creation and modification of corporate action events. The CA Admin interface is used by the Corporate Actions department to manage events. Customer responses are handled on **Next** and **ClientManager**.

Sending responses for new emissions to **Abasec** is done through **AIS** and **WCF** (`service-abatron`).

> **Note:** The old web interface "Now" has been shut down. All customer-facing interactions now go through Next.

---

## 2. TRS Reporting & Decision Maker

When reporting to the **TRS system**, Excel macros require a "decisionmaker" column. The logic for populating this field has been moved into the corporate-action service on-prem and uses the **TRS Client via REST**.

### Exception: PARTNER Initiator Source

When the initiator source is **PARTNER**, the `TrsDecisionMaker` is set to the **Elector ID** using JWT data from `event-response-handler` (ERH).

### Decision Maker Rules

| Scenario | Decision Maker | Transmitting Firm | Investment Decision Within Firm |
|----------|---------------|-------------------|--------------------------------|
| **FKF** (Fondkonto Fond) | Always the **Owning Company** | N/A | N/A |
| **Partners** | Partner's **LEI** | Partner's **LEI** | **Stakeholder in AbaSec** |

---

## 3. Database

Uses the **SORC** schema in the **ORAP** Oracle database.

### Tables

| Table | Description |
|-------|-------------|
| `emission_prospekt` | Main prospect/event table |
| `emission_order` | Customer orders/responses |
| `emission_log` | Audit/event log |
| `emission_prospekt_ipo` | IPO-specific prospect data |
| `emission_settings` | Sell emission rights settings |
| `emission_settings_hist` | Historical emission settings |
| `instruments` | Instrument reference data |

### Important Notes

- The `instruments` table is **truncated nightly and reloaded**. This can cause issues with the cronjob that sends responses if the table is empty at the time of execution.
- **No purging** is done on any of the tables. Data accumulates over time.

---

## 4. Dataland (BigQuery)

Corporate action data is sent to **Dataland** on create, update, and delete operations. Customer responses are also sent.

### BigQuery Tables

| Table | Classification | Access Requirements |
|-------|---------------|---------------------|
| **Corporate Events** | Secret | Standard secret access |
| **Customer Answers** | Secret | Standard secret access |
| **Customer Answers active IPO** | Secret | Requires membership in the `active-ipo` secret group |
| **Customer Answers inactive IPO** | Confidential | Standard confidential access |

---

## 5. Operating Instructions

The service must be **always running** for external users (customer-facing responses on Next and ClientManager).

| Component | Availability | Notes |
|-----------|-------------|-------|
| Customer-facing (Next/ClientManager) | 24/7 | Must always be available for customer responses |
| Abasec integration | Late night | Used late at night, before night jobs sync answers to Abasec |
| Internal admin (CA Admin) | Business hours | Used by Corporate Actions department during working hours |

---

## 6. Monitoring

### Health Checks

| Health Check | Description |
|-------------|-------------|
| `dbHealthIndicator` | Database connectivity check |
| `diskSpaceHealthIndicator` | Disk space availability |
| `garbageCollectionHealthCheck` | JVM garbage collection health |
| `httpStatusHealthCheck` | Becomes **unhealthy** when too many 5XX responses are detected |
| `oracleObjectHealthCheck` | Checks for invalid Oracle objects in the schema |

### Kibana

- Application filter: `application:corporate-action`

---

## 7. Danish 20% Rule

**Puljebekendtgorelsen** - No shareholder can own more than 20% of the same company at the time of purchase on a pension account.

### Threshold

- **2024:** 58,100 DKK - a customer can invest up to this amount regardless of the 20% limit.

### Validation

- **Soft validation** on customer response (warns but does not block).

### Current Coverage

| Covered | Not Covered (Scoped Out) |
|---------|--------------------------|
| Rights issues | Other |
| IPO | Conversion |
| Subscription warrants | Repurchase |
| | Redemption |
| | Tender offer |

### Known Limitations

| Limitation | Current Behavior | Future Plan |
|-----------|-----------------|-------------|
| Account vs. customer level | Checks on **account level** only | Change to **customer level** when Bull updates |
| Timing of check | Checks on **response day** | Validate on **last day of offer** |
| Calculation basis | Calculates on **subscription value** | N/A (by design) |

> **Note:** The calculation uses subscription value, not market value.

---

## 8. Hidden Events with Automatic Presubscription

Designed for **large IPOs with many employee presubscribers**.

### How It Works

1. **Hidden events** are created via the on-prem CA Admin using a checkbox.
2. Hidden events are **not visible** in the web or app but are accessible via a **specific direct link**.
3. Answers on hidden events can be **automatically marked as presubscriptions**.
4. Presubscriptions are synced to NNX via **PubSub**:
   - `corporate-action` (on-prem) --> `ca-presubscription` (NNX)

### Creation

Created through the on-prem CA Admin interface by checking the "hidden event" checkbox.

### Known Limitations

- Hidden events are **separate from public events**, resulting in two compile files.
- **Anyone with the direct link** can access a hidden event (no additional auth gating).

### Related Epics

| Epic | Scope |
|------|-------|
| **EQF-1422** | MVP1 - Initial hidden events functionality |
| **EQF-1467** | MVP2 - Enhancements and presubscription sync |

---

## 9. Reserve Trading Power

System that reserves and releases trading power on accounts for corporate actions.

### Architecture

- Uses the **Pre-Trade API** for reservations.
- Input is provided via **CSV file uploaded to a Google Cloud Storage bucket**.
- Built on an **event sourcing architecture** for account reservations.

### Reserve Request Parameters

| Parameter | Description |
|-----------|-------------|
| `amount` | Amount to reserve |
| `validUntil` | Expiry date for the reservation |
| `reference` | Reference identifier |
| `reason` | Reason for the reservation |
| `force` | `true` = reserve regardless of current trading power; `false` = reject if reservation would cause negative trading power |

### Cross-Currency Support

Supports cross-currency reservations with **ongoing FX rate updates** to maintain accurate reservation amounts.

### Norwegian Shareholders Disclosure

A dedicated endpoint returns all currently reserved positions for disclosure obligations:

| Field | Description |
|-------|-------------|
| `AccountNumber` | The account with reserved positions |
| `ISIN` | The instrument ISIN |
| `shares` | Number of reserved shares |

This data is available via the **Pre-Trade Swagger API** endpoint.

---

## 10. Update Emission Settings for Blocked Accounts

The Corporate Action department periodically requests help to turn off emission settings for users who are missing LEI or have blocked accounts.

### Process

1. **Identify affected accounts** using the following SQL criteria:
   - Lock type `L`, `B`, or `F`
   - Juridical persons without a valid LEI

2. **Update emission settings:**

```sql
UPDATE SORC.EMISSION_SETTINGS
SET AUTO_EMISSION = 0
WHERE <affected_account_criteria>;
```

3. **Access:** Requires a **CM (Change Management) ticket** to get DBA access for executing the update.

### Trigger

This is a manual process initiated by the Corporate Action department when they identify accounts that should not participate in automatic emissions.

---

## 11. Translation Matrix

Translation matrix for new event types across all supported markets:

| Event Type | EN | SE | NO | FI | DK |
|-----------|----|----|----|----|-----|
| Unlisted offering | Unlisted offering | Onoterat erbjudande | Unotert tilbud | Listaamaton tarjous | Unoteret udbud |
| Venture fund offering | Venture fund offering | Riskkapitalfondserbjudande | Venturefondstilbud | Paaomararahastotarjous | Venturefondsudbud |
| Directed offering | Directed offering | Riktat erbjudande | Rettet tilbud | Suunnattu tarjous | Rettet udbud |
| Offering without priority | Offering without priority | Erbjudande utan foretradesratt | Tilbud uten fortrinnsrett | Tarjous ilman etuoikeutta | Udbud uden fortegningsret |

> **Note:** Translations should be verified with local market teams for accuracy. Special characters (a, o, a) should be properly encoded.

---

## 12. Testing

### Creating Test Data

Two approaches:

1. **Copy from production:** Extract a prospect from production `SORC.emission_prospekt` and insert into the test environment.
2. **Create manually:** Use the **Emission Admin** interface to create a new test prospect.

### Activating a Test Prospect

After creating or copying a prospect, activate it via the **"Visa Prospekt"** (Show Prospect) function in the admin interface.

### Testing Customer Responses

| Method | Description |
|--------|-------------|
| **Next** | Log in with test user credentials and submit a response |
| **Client Manager** | Use ClientManager credentials (available in the team documentation) to submit on behalf of a customer |

> **Note:** Test credentials for both Next and Client Manager are documented in the team's internal documentation. Do not store credentials in this file.
