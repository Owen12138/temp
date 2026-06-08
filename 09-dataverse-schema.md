# BOA — Dataverse Schema (Authoritative Final Design)

This is the **authoritative, final** data model for the BOA AI use-case
tracker once it moves from the standalone `colUseCases` sample data (v1)
onto Dataverse. There are **exactly three tables**:

1. **Business Hierarchy** — canonical SBU / LOB classification.
2. **Projects** — the parent table for AI use-case / intake records.
3. **Value** — the child table for recurring project value reporting.

Design principles baked into this schema:

- **No custom user table.** People are stored directly as plain-text
  name + email field pairs on the record. There is no lookup to a
  Contacts/Users table.
- **One lookup relationship per table edge.** Projects → Business
  Hierarchy (classification), and Value → Projects (parent).
- **Choice (option set) columns** carry the controlled vocabularies; the
  full option lists are in [§5](#5-choice-option-sets).

Type column legend: `Text` = single line of text, `Multiline` =
multiple lines of text, `Email` = single line w/ Email format, `URL` =
single line w/ URL format, `Choice` = option set, `MultiChoice` =
multi-select option set, `Lookup` = relationship column, `Currency`,
`Whole #`, `Date`, `DateTime`, `Yes/No` = boolean.
A trailing **†** marks a field whose Dataverse type I inferred and you
should confirm — see [§6](#6-assumptions--open-questions).

---

## 1. Table: Business Hierarchy

**Purpose:** Canonical business-classification table. Exactly **one row
per valid SBU + LOB combination**. Projects reference this table instead
of storing free-text SBU/LOB, so classification stays consistent.

| Field | Type | Notes |
|---|---|---|
| **Business Hierarchy Key** | Text | **Primary name column.** Formatted *exactly* as `SBU/LOB` (e.g. `Capital Markets/Equities`). This string is the practical matching key used by transformed / import-facing files. |
| Strategic Business Unit (SBU) | Text | The SBU half of the key. |
| Line of Business (LOB) | Text | The LOB half of the key. |

**Rules & behaviour:**

- **Business Hierarchy Key** is text, formatted exactly `SBU/LOB`. It is
  the canonical stored classification value.
- In the app the user **selects SBU and LOB**, and the app **resolves
  that pair to the matching Business Hierarchy row** (and its Key) before
  writing the Project's lookup.
- This table is the single source of truth for valid SBU/LOB pairs —
  adding a new business area means adding a row here.

---

## 2. Table: Projects

**Purpose:** Parent table for AI use-case / intake records. One row per
use case. This is the table the Detail screen (`srcDetail`) reads and
edits.

> **Relationship:** Projects has a **lookup to Business Hierarchy**
> (many Projects → one Business Hierarchy row). Business Hierarchy is the
> canonical stored classification; the Business Hierarchy Key is the
> practical matching key in import-facing files.

### 2.1 Identity & classification

| Field | Type | Notes |
|---|---|---|
| **Use Case Name** | Text | **Primary name column.** Sourced from the source system's field ID. |
| Use Case ID | Text | Business identifier. Also sourced from the source system's field ID. Good candidate for an **alternate key**. |
| Business Hierarchy Key | Lookup | → **Business Hierarchy**. The classification of this project. |
| Project Problem Statement | Multiline | |
| Description of AI Solution | Multiline | |
| Type of Use Case | Choice | AI / Non-AI — see [§5](#5-choice-option-sets). |
| Project Status | Choice | 7-stage lifecycle — see [§5](#5-choice-option-sets) and [§4 mapping](#4-mapping-to-app-collections). |

### 2.2 People (plain-text name + email pairs — no user table)

| Field | Type | Notes |
|---|---|---|
| AI Enablement Owner Name | Text | |
| AI Enablement Owner Email | Email | |
| AI Model Developer Name | Text | |
| AI Model Developer Email | Email | |
| AI Solution Owner Name | Text | |
| AI Solution Owner Email | Email | |
| Executive Sponsor Name | Text | |
| Executive Sponsor Email | Email | |
| Data Contact Name | Text | |
| Data Contact Email | Email | |
| Primary Business Contact Name | Text | |
| Primary Business Contact Email | Email | |
| Intake Submitter Name | Text | |
| Intake Submitter Email | Email | |
| Other Stakeholder Impacted | Multiline † | Free text of impacted stakeholders. |

### 2.3 Governance — AIRAP

| Field | Type | Notes |
|---|---|---|
| AIRAP ID | Text | |
| AIRAP Status | Choice | Not Started / In Process / Completed. |
| AIRAP Outcome Link | **URL** | **Must be a Dataverse URL field.** |

### 2.4 Governance — CIRA

| Field | Type | Notes |
|---|---|---|
| CIRA ID | Text | |
| CIRA Status | Choice | Not Started / In Process / Completed. |
| CIRA Report Link | **URL** | **Must be a Dataverse URL field.** |

### 2.5 Governance — model & review

| Field | Type | Notes |
|---|---|---|
| Algorithm Review Completed | Yes/No † | "Completed" reads as a boolean flag. |
| Code Review | Yes/No † | |
| Model Validation Approval | Yes/No † | |
| Model Validation Report ID | Text | |
| Model Monitoring | Yes/No † | |
| MLOps Intake | Yes/No † | |
| Enterprise AI Inventory | Yes/No † | In the enterprise AI inventory? |
| AI Solution Performance | Multiline † | |
| MAP ID | Text | |
| Triage Number | Text † | |
| Technology | Multiline † | |
| Prerequisite | Multiline | |
| Standard Folder Structure | URL † | Link to / flag for the standard folder structure. |

### 2.6 Intake & timeline

| Field | Type | Notes |
|---|---|---|
| Intake Start Time | DateTime † | |
| Intake Completion Time | DateTime † | |
| Estimated Completion Time | DateTime † | |
| Project Completion Fiscal Year | Text † | e.g. `FY2025`. |
| Project Completion Fiscal Quarter | Choice | Q1 / Q2 / Q3 / Q4. |
| Project Completion Month | Text † | |

### 2.7 Value / financials

| Field | Type | Notes |
|---|---|---|
| Estimated Monetary Benefit | Currency † | |
| Estimated Monetary Benefit Type | Choice | See [§5](#5-choice-option-sets). |
| Estimated Net Benefit Description | Multiline | |
| Funding | Text † | Funding source / status. Confirm if Choice. |
| Finance Partner Tracked | Yes/No † | Is a finance partner tracking this? |

### 2.8 Output & operations

| Field | Type | Notes |
|---|---|---|
| Output of Deliverable | **MultiChoice** | **Multi-select.** Options in [§5](#5-choice-option-sets). **Working Model Integration must be one of these options.** |
| Output Refresh Frequency | Choice | See [§5](#5-choice-option-sets). |
| Monthly Update | Multiline | **Remains a freeform text field** (not a choice). |
| Team Notification | Yes/No † | |

---

## 3. Table: Value

**Purpose:** Child table for **recurring project value reporting** —
one row per reporting period (e.g. per fiscal quarter/month) per project.

> **Relationship:** Value has a **many-to-one lookup to Projects**.
> **Project** is the parent lookup field (many Value rows → one Project).

| Field | Type | Notes |
|---|---|---|
| **Project** | Lookup | → **Projects**. Parent lookup field. |
| Use Case ID | Text † | Denormalized copy of the parent's Use Case ID for matching in import-facing files. |
| Fiscal Year | Text † | |
| Fiscal Quarter | Choice † | Likely reuses the Q1–Q4 option set. |
| Month | Text † | |
| Realized Value | Currency † | |
| Value Frequency | Choice † | Cadence of this value entry. Confirm option set. |
| Business Case Funded | Yes/No † | |
| Description of Investment | Multiline | |
| Investment Spend | Currency † | |
| Value Calculation | Multiline † | How the realized value was calculated. |
| Value Drivers | Multiline † | |
| Value Sign Off | Text † | Approver / sign-off. Confirm if name vs. Yes/No. |
| Value Status | Choice † | Confirm option set. |
| Notes | Multiline † | Freeform notes. *(See [§6](#6-assumptions--open-questions) — "notes recorded at" was ambiguous.)* |
| Recorded At | DateTime † | When this value row was recorded. |

---

## 4. Mapping to app collections

The standalone app (v1) drives the status stepper off the `colStatus`
collection in `App.OnStart` ([`01-app-setup.md`](01-app-setup.md)), which
stores a short **Code** and shows a friendly **Label**. The Dataverse
**Project Status** choice uses the authoritative labels below. When you
migrate, map each `colStatus.Code` to the Dataverse choice value:

| `colStatus` Code | `colStatus` Label | Dataverse **Project Status** choice |
|---|---|---|
| Rationale | Rationale | Rationale for AI Solutions |
| DataPrep | Data Prep | Data Preparation |
| Development | Development | Development |
| Testing | Testing | Test and Validation |
| Deployment | Deployment | Deployment |
| Monitoring | Monitoring | Monitoring and Review |
| Decommissioning | Decommissioning | Decommissioning |

Keep the stepper logic data-driven off `colStatus` (`Order` field +
`CountRows(colStatus)`); only the label text differs between the app
collection and the Dataverse choice.

---

## 5. Choice option sets

These are the controlled vocabularies for the Choice / MultiChoice
columns. Labels are authoritative as written.

**Type of Use Case** (Projects)
- AI
- Non-AI

**Project Status** (Projects) — 7-stage lifecycle, in order
1. Rationale for AI Solutions
2. Data Preparation
3. Development
4. Test and Validation
5. Deployment
6. Monitoring and Review
7. Decommissioning

**AIRAP Status** (Projects)
- Not Started
- In Process
- Completed

**CIRA Status** (Projects)
- Not Started
- In Process
- Completed

**Estimated Monetary Benefit Type** (Projects)
- Revenue Enhancement
- Cost Reduction
- Cost / Risk Avoidance
- Client / Employee Experience
- Capacity Creation

**Output of Deliverable** (Projects, **multi-select**)
- Data Set
- Report
- Dashboard
- Real-time Scoring
- Chatbot
- Working Model Integration  ← *Working Model Integration must be one of these options.*

**Output Refresh Frequency** (Projects)
- Daily
- Weekly
- Monthly
- Quarterly
- Yearly
- Ad Hoc
- One-time Handoff

**Project Completion Fiscal Quarter** (Projects)
- Q1
- Q2
- Q3
- Q4

---

## 6. Assumptions & open questions

These are transcription / design ambiguities I resolved with a best
guess (and flagged with **†** above). Confirm or correct each:

1. **Value table — "notes recorded at".** Dictated without a separating
   comma, so it's ambiguous. I split it into **two** fields — `Notes`
   (Multiline) and `Recorded At` (DateTime). If it was meant as a single
   field (e.g. `Notes Recorded At`), say so and I'll merge them.
2. **Field types marked †.** I inferred these from the field name:
   - *Booleans* (`Algorithm Review Completed`, `Code Review`,
     `Model Validation Approval`, `Model Monitoring`, `MLOps Intake`,
     `Enterprise AI Inventory`, `Finance Partner Tracked`,
     `Team Notification`, `Business Case Funded`) — confirm these are
     Yes/No and not Choice or text status fields.
   - *Dates* (`Intake Start Time`, `Intake Completion Time`,
     `Estimated Completion Time`, `Recorded At`) — confirm Date vs.
     DateTime.
   - *Currency* (`Estimated Monetary Benefit`, `Realized Value`,
     `Investment Spend`) — confirm currency vs. plain number, and units.
   - `Funding`, `Standard Folder Structure`, `Value Status`,
     `Value Sign Off`, `Value Frequency` — confirm whether these are
     Choice columns and, if so, their option sets (not provided).
3. **Value `Fiscal Quarter` / `Fiscal Year` / `Month`.** Assumed
   `Fiscal Quarter` reuses the Projects Q1–Q4 option set; `Fiscal Year`
   and `Month` left as text. Confirm.
4. **Value `Use Case ID`.** Documented as a denormalized text copy of the
   parent for import-side matching. If it should instead be derived
   purely through the `Project` lookup, drop the standalone column.
5. **Naming from dictation.** A few names had transcription noise
   ("AI solution on the email" → **AI Solution Owner Email**; "this is
   case ID CIRA ID" → **CIRA ID**). Confirm these readings.
```
