# BOA — Migrate srcDetail to Dataverse (Edit Forms)

Convert the **Use Case Detail screen (`srcDetail`)** from the sample
collections to the live Dataverse tables, using **Edit forms**. When done,
every section reads and writes real **Projects** / **Value** rows.

Prerequisites:
- `srcList` already migrated ([`10-srclist-dataverse-guide.md`](10-srclist-dataverse-guide.md))
  so `selectedUC` is a **Projects** record and `App.OnStart` seeds
  `Set(selectedUC, Defaults(Projects))`.
- Status stepper fix applied ([`13-status-stepper-dataverse-fix.md`](13-status-stepper-dataverse-fix.md)).
- Dataverse tables exist ([`09-dataverse-schema.md`](09-dataverse-schema.md)).

---

## The one idea that makes this easy

In the sample app, **Contacts, Value, Governance, and Tech were each
separate child collections** (`colValueEntries`, `colGovernance`,
`colTechReview`, plus people on the use-case row). In the **final schema**,
that's not how it's modelled:

- **Only `Value` is a child table** (one Project → many Value rows).
- **Everything else is a column on the `Projects` row** — people are plain
  name/email text fields, governance is AIRAP/CIRA fields, funds and
  updates are fields too.

So the whole screen reduces to two patterns:

| Section | Pattern |
|---|---|
| Info, **Contacts, Funds, Gov, Tech, Updates** | An **Edit form bound to `Projects`, `Item = selectedUC`**, showing that section's columns. Each has its own Save. |
| **Value** | A **gallery + an Edit form bound to the `Value` table**, filtered by the Project lookup. |

And because these are *real Dataverse columns*, the form **auto-generates
the right control per type** — choice columns get a Choices-bound combo
box, lookups get a lookup picker, dates get a date picker. The hand-built
dropdowns and code/label `LookUp`s from the collection build **go away.**

---

## Part 0 — Fix the named formulas (App.Formulas)

The child-collection formulas don't apply anymore. Click `App` → **Formulas**:

**Replace** the value rollup to query the Dataverse `Values` table by the
Project lookup:

```powerfx
ucValueRows = Filter(Values, Project.'Use Case ID' = selectedUC.'Use Case ID');
```

**Delete** `ucGovRows`, `ucTechRows`, `govDoneCount`, `govTotalCount`,
`govProgressPct` (and any `ucTech*`) — governance/tech are now Projects
fields, not child rows. (If you still want a governance "progress" badge,
see the optional note in Part 4.)

> The table is **`Values`** (plural). Because that's a distinct name (not
> the `Value()` function), you can reference it unquoted. Use whatever name
> shows under **Data** in the tree if yours differs.

---

## Part 1 — Info section: rebind `frmInfo` to Projects

This is the centerpiece. Select `frmInfo` (in `conSectionInfo`) and change:

| Property | Old | New |
|----------|-----|-----|
| DataSource | `colUseCases` | `Projects` |
| Item | `selectedUC` | `selectedUC` *(unchanged — now a Projects record)* |
| DefaultMode | `FormMode.Edit` | `FormMode.Edit` |

Then **Edit fields** → remove the old collection fields and add the
Dataverse columns. Map per the table below. The crucial difference from
the collection build: **for choice and lookup columns you keep the form's
auto-generated control** — don't hand-build dropdowns.

| Old card | Dataverse column | Card type the form generates | Card `Update` |
|---|---|---|---|
| `Name` | `Use Case Name` | Text input | *(default)* |
| `ProblemStatement` | `Project Problem Statement` | Text (set MultiLine) | *(default)* |
| `AISolution` | `Description of AI Solution` | Text (set MultiLine) | *(default)* |
| `Type` | `Type of Use Case` | **Choice combo box** (auto) | `DataCardValue.Selected` *(default)* |
| `Status` | `Project Status` | **Choice combo box** (auto) | `DataCardValue.Selected` *(default)* |
| `SBU` + `LOB` | `Business Hierarchy` (lookup) | **Lookup combo box** (auto) | `DataCardValue.Selected` *(default)* — or cascade, see below |
| `TargetDate` | `Estimated Completion Time` | Date picker | `DataCardValue.SelectedDate` *(default)* |
| `RefreshFreq` | `Output Refresh Frequency` | **Choice combo box** (auto) | `DataCardValue.Selected` *(default)* |

Optionally add `Output of Deliverable` (multi-select choice → the form
makes a **multi-select** combo box; `Update = DataCardValue.SelectedItems`).

### Why this is simpler than the collection version

- **Status no longer needs the code/label `LookUp`.** `Project Status` is a
  real choice, so the form's combo box shows the labels and writes the
  choice directly (`Update = DataCardValue.Selected`). Delete the old
  `ddStatus2` + `LookUp(colStatus, …)` wiring entirely.
- The stepper still advances on save because `frmInfo.OnSuccess`
  (below) refreshes `selectedUC`, and the stepper reads
  `Text(selectedUC.'Project Status')` per [guide 13](13-status-stepper-dataverse-fix.md).

### Multiline + widths

`Use Case Name`, `Project Problem Statement`, `Description of AI Solution`:
set the **card** Width to `frmInfo.Width` (full row). For the two long
ones, unlock the card → select the input → `Mode = TextMode.MultiLine`,
raise card Height (~120). The rest keep half-column width and flow into
two columns.

### SBU / LOB → Business Hierarchy lookup

**Default (simplest):** keep the auto lookup combo box on the `Business
Hierarchy` card. It lists the Business Hierarchy rows by their key
(`SBU/LOB`, e.g. `Capital Markets/Equities`); the user picks one and
`Update = DataCardValue.Selected` writes the lookup.

**Recommended (cascading SBU → LOB, matches the schema's intent):** unlock
the card, replace the combo with two dropdowns:

```powerfx
// ddSBU2.Items
Distinct('Business Hierarchy', 'Strategic Business Unit')
// ddLOB2.Items  (LOBs valid for the chosen SBU)
Distinct(Filter('Business Hierarchy', 'Strategic Business Unit' = ddSBU2.Selected.Value), 'Line of Business')
// Business Hierarchy DataCard  Update  (resolve the pair to the lookup row)
LookUp('Business Hierarchy',
       'Strategic Business Unit' = ddSBU2.Selected.Value
       && 'Line of Business' = ddLOB2.Selected.Value)
```

Set each dropdown's `Default` from the current record:
`ddSBU2.Default = selectedUC.'Business Hierarchy'.'Strategic Business Unit'`,
`ddLOB2.Default = selectedUC.'Business Hierarchy'.'Line of Business'`.

### Save / Cancel / refresh

Keep the existing buttons and wiring — only the data source changed:

| Property | `btnSaveInfo` | `btnCancelInfo` |
|---|---|---|
| OnSelect | `SubmitForm(frmInfo)` | `ResetForm(frmInfo)` |
| DisplayMode | `If(frmInfo.Unsaved, DisplayMode.Edit, DisplayMode.Disabled)` | `DisplayMode.Edit` |

On `frmInfo`:
```powerfx
OnSuccess: Set(selectedUC, frmInfo.LastSubmit)
OnFailure: Notify("Couldn't save: " & frmInfo.Error, NotificationType.Error)
```

`SubmitForm` now writes a real Dataverse row; `OnSuccess` refreshes
`selectedUC` so the rail-head pill and stepper advance immediately.

---

## Part 2 — Section-form recipe (Contacts, Funds, Gov, Tech, Updates)

These are all just **Projects columns**, so each section becomes its own
Edit form. Repeat this recipe in each `conSection*`:

1. **Delete** the old collection-bound controls / checklist galleries
   (`galGovChecks`, `galTechChecks`, the person-card grid, etc.).
2. Insert → **Edit form** → set `DataSource = Projects`,
   `Item = selectedUC`, `DefaultMode = FormMode.Edit`, `Columns = 2`,
   Width `Parent.Width - 56`.
3. **Edit fields** → add that section's columns (lists below).
4. Keep `OnSuccess = Set(selectedUC, frm….LastSubmit)` on each form. For
   **saving**, use **one** mechanism — recommended: wire the action bar's
   **Save Draft** to `SubmitForm` the visible section's form via a `Switch`
   on `currentSection`, and **delete `conSubmitZone`** (see
   [`17` Step 10](17-srcdetail-info-tab-dataverse.md)). Don't add a Save
   button to every section *and* keep the action bar / submit zone.

Choice/URL/date columns auto-generate the correct control — leave them.
People fields are plain text (no user table).

| Section (form) | Projects columns to add |
|---|---|
| **Contacts** (`frmContacts`) | AI Enablement Owner Name/Email · AI Model Developer Name/Email · AI Solution Owner Name/Email · Executive Sponsor Name/Email · Data Contact Name/Email · Primary Business Contact Name/Email · Intake Submitter Name/Email |
| **Funds** (`frmFunds`) | Funding · Finance Partner Tracked · Estimated Monetary Benefit (currency) · Estimated Monetary Benefit Type (choice) · Estimated Net Benefit Description |
| **Gov** (`frmGov`) | AIRAP ID · AIRAP Status (choice) · AIRAP Outcome Link (**URL**) · CIRA ID · CIRA Status (choice) · CIRA Report Link (**URL**) · Algorithm Review Completed · Model Validation Approval · Enterprise AI Inventory · MAP ID · Triage Number |
| **Tech** (`frmTech`) | Code Review · Model Monitoring · MLOps Intake · Model Validation Report ID · Standard Folder Structure · AI Solution Performance · Prerequisite · Technology |
| **Updates** (`frmUpdates`) | Monthly Update (multiline text) |

> **Multiple forms over one record is fine.** Each form edits its own
> cards on the same `selectedUC`; each `SubmitForm` saves only that form's
> fields. Give each its own Save/Cancel and `OnSuccess = Set(selectedUC,
> frm….LastSubmit)` so the others see the refreshed record.

> **Optional governance progress badge.** The old `govDoneCount` came from
> child rows. To keep a "x of y done" badge, compute it from the Projects
> fields, e.g.:
> ```powerfx
> CountIf([ Text(selectedUC.'AIRAP Status'), Text(selectedUC.'CIRA Status') ], ThisRecord.Value = "Completed")
> ```
> (or a small inline table of the governance statuses you care about).

---

## Part 3 — Value section (the one real child table)

### Rollup tiles + gallery

- **Realized tile:** `Sum(ucValueRows, 'Realized Value')`.
- **Estimated tile:** `selectedUC.'Estimated Monetary Benefit'`.
- **Gallery** `galValueRows.Items = ucValueRows` (now the filtered `Value`
  rows from Part 0). Rebind each row label to the Value columns —
  `ThisItem.'Fiscal Quarter'`, `ThisItem.'Fiscal Year'`,
  `ThisItem.'Realized Value'`, `Text(ThisItem.'Value Status')`,
  `Text(ThisItem.'Value Frequency')`, etc. (Choice columns: wrap in
  `Text(...)` to display, same rule as everywhere.)

### Add/Edit with an Edit form (use form)

Replace the hand-built modal inputs + `Collect`/`Patch(colValueEntries)`
with an **Edit form bound to `Value`** inside the modal card:

1. In `conValueModalCard`, delete the manual inputs. Insert → **Edit
   form** `frmValue`, `DataSource = Values`, `Columns = 1`, Width
   `Parent.Width - 56`.
2. **Edit fields:** Project (lookup) · Fiscal Year · Fiscal Quarter
   (choice) · Month · Realized Value (currency) · Value Frequency · Value
   Status · Description of Investment · Investment Spend · Notes.
3. **The Project lookup card** ties the row to this use case. On a new
   row, set its `Update` to the parent record:
   `Update = selectedUC`  *(or hide the card and default it — see below)*.

Wire the buttons:

```powerfx
// "+ Add value entry"
NewForm(frmValue); Set(showValueModal, true)

// "Edit" on a row (in the gallery row template)
EditForm(frmValue); Set(editingValueRow, ThisItem); Set(showValueModal, true)
// and set  frmValue.Item = editingValueRow

// Modal Save
SubmitForm(frmValue)
// frmValue.OnSuccess:
Set(showValueModal, false)
// Modal Cancel
ResetForm(frmValue); Set(showValueModal, false)
```

For a **new** row, make sure the Project lookup gets `selectedUC`: either
keep the Project card visible (pre-selected to `selectedUC`) or unlock it,
hide it, and set its `Update = selectedUC`. `SubmitForm` then inserts the
Value row already linked to this project.

> This replaces the collection `Collect`/`Patch` with real Dataverse
> insert/update via `SubmitForm` — and `Sum(ucValueRows, …)` re-rolls the
> tiles automatically after a save.

---

## Part 4 — Status stepper

No work here beyond [guide 13](13-status-stepper-dataverse-fix.md): the
stepper reads `Text(selectedUC.'Project Status')` against the `DVStatus`
field, and `frmInfo.OnSuccess` refreshes `selectedUC` so it advances on
save.

---

## Part 5 — Sanity check

From `srcList`, open a use case:

- [ ] **Info** form shows real values; Status/Type/Refresh are choice
      combo boxes (labels, not codes); the lookup/SBU+LOB resolves.
- [ ] Editing a field enables **Save**; saving writes to Dataverse and the
      **stepper advances** (Status change reflected immediately).
- [ ] **Contacts / Funds / Gov / Tech / Updates** each load their Projects
      fields and save independently.
- [ ] **Value** section lists the project's real Value rows; tiles sum
      correctly; **+ Add** opens the form, saves a new row linked to this
      project; **Edit** updates a row.
- [ ] AIRAP/CIRA **link** fields render as clickable URLs.
- [ ] No "incompatible types" errors (choices displayed via `Text(...)`,
      compared choice-to-choice).

---

## Troubleshooting

| Symptom | Cause | Fix |
|---|---|---|
| `frmInfo` cards show old field names / errors | DataSource still `colUseCases` | Set `DataSource = Projects`; re-add fields via Edit fields. |
| Status card shows a text box, not a label combo | You kept the old unlocked `ddStatus2` | Reset the card to default (re-add the `Project Status` field) — the form builds a Choices combo automatically; delete the `LookUp(colStatus,…)` wiring. |
| Value gallery empty | `ucValueRows` still filters `colValueEntries`, or the lookup nav is wrong | Use `Filter(Values, Project.'Use Case ID' = selectedUC.'Use Case ID')` (Part 0). |
| New Value row not linked to the project | Project lookup card not set on insert | Set the Project card `Update = selectedUC` (or pre-select it) before `SubmitForm`. |
| "Incompatible types" on a Value/Gov choice display | Showing a choice column without `Text(...)` | Wrap displays in `Text(...)`; compare choice-to-choice, never to a string. |
| Stepper doesn't advance after Save | `OnSuccess` missing | `frmInfo.OnSuccess = Set(selectedUC, frmInfo.LastSubmit)`; see guide 13. |
| `Values` table not recognized | Wrong name, or data source not added | Add the table via Data → Add data; reference it by the exact name shown under **Data** (here, `Values`). |

---

## What's next

- **`srcNew`** (new intake): an Edit form in `FormMode.New` over `Projects`
  — `NewForm` + `SubmitForm`, resolving SBU/LOB to the Business Hierarchy
  lookup, same as Part 1. Then drop the sample `ClearCollect(colUseCases…)`
  and the other `col*` sample blocks from `App.OnStart`.
