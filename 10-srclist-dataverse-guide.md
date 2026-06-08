# BOA Canvas App — Migrate srcList to Dataverse (Step-by-Step)

This guide converts the **Use Case List screen (`srcList`)** from the
in-memory `colUseCases` sample collection (v1) to the **live Dataverse
tables** (v2). When you finish, the same list — same layout, filters,
pill, footer — is reading and filtering real **Projects** rows, with SBU
resolved through the **Business Hierarchy** lookup and Realized Value
rolled up from the **Value** child table.

It assumes you have already:

- Built `srcList` against collections using
  [`07-scrlist-guide.md`](07-scrlist-guide.md). This guide edits that
  screen in place; it does **not** rebuild it.
- Created and populated the three Dataverse tables exactly as documented
  in [`09-dataverse-schema.md`](09-dataverse-schema.md): **Business
  Hierarchy**, **Projects**, **Value**.

> **Scope:** this guide covers `srcList` only. `srcDetail`, `srcNew`,
> value rollups on the detail screen, and Submit/Patch flows reference
> the old collection field names too and will need the same rename pass
> later — flagged at the end. Do `srcList` first; it's the lowest-risk
> screen to learn the Dataverse field references on.

---

## Part 0 — Confirm YOUR actual Dataverse names (do this first)

Power Apps references Dataverse tables and columns by their **display
names**, but two things vary per environment and you must confirm them
before writing any formula:

1. **Table name as it appears in Studio.** When you add a Dataverse
   table as a data source, Studio shows it by display name (`Projects`,
   `Business Hierarchy`, `Value`). The table named **`Value`** is a
   problem: `Value` is also a built-in Power Fx function. Reference it
   quoted as `'Value'` everywhere, and if Studio renamed it on import
   (e.g. `Values`), use whatever name shows in **Data** in the tree.
2. **The lookup column name** on Projects that points at Business
   Hierarchy. The schema calls the stored key `Business Hierarchy Key`,
   but the *lookup column* may be named `Business Hierarchy` (the
   relationship). You navigate the relationship with a dot.

Fill this table in from your solution (Power Apps → your table → Columns)
and keep it next to you — every step below uses the **App field** name on
the left and you substitute the **Dataverse display name** on the right:

| App field (`colUseCases`) | Dataverse column (Projects) — confirm exact display name | Type | Reference in a formula |
|---|---|---|---|
| `UCID` | `Use Case ID` | Text | `ThisItem.'Use Case ID'` |
| `Name` | `Use Case Name` (primary name) | Text | `ThisItem.'Use Case Name'` |
| `SBU` | via lookup → `Business Hierarchy`.`Strategic Business Unit` | Lookup | `ThisItem.'Business Hierarchy'.'Strategic Business Unit'` |
| `Owner` | `AI Solution Owner Name` | Text | `ThisItem.'AI Solution Owner Name'` |
| `Status` | `Project Status` | **Choice** | `ThisItem.'Project Status'.Value` |
| `FY` | `Project Completion Fiscal Year` | Text | `ThisItem.'Project Completion Fiscal Year'` |
| `RealizedValue` | *(none on Projects)* — rolled up from `'Value'` | — | see [Part 2C](#part-2c--decide-how-realized-value-is-computed) |
| `EstimatedValue` | `Estimated Monetary Benefit` | Currency | `ThisItem.'Estimated Monetary Benefit'` |
| `LastUpdated` | `Modified On` (system column) | DateTime | `ThisItem.'Modified On'` |
| `Type` | `Type of Use Case` | Choice | `ThisItem.'Type of Use Case'.Value` |

> **Choice columns return a record, not text.** `ThisItem.'Project
> Status'` is a choice *record*; its label is `.Value`. Comparisons and
> the color Switch must use `.Value`.
>
> **Lookup columns return the related row.** `ThisItem.'Business
> Hierarchy'` is the whole Business Hierarchy record; dot into it for
> `'Strategic Business Unit'`, `'Line of Business'`, or `'Business
> Hierarchy Key'`. If the related row is blank, the dotted reference is
> blank (no error).

---

## Part 1 — Add the data sources

### Step 1 — Connect the three tables

In Studio, left rail → **Data** → **+ Add data** → search and add, one at
a time:

- **Projects**
- **Business Hierarchy**
- **Value**

They now appear under **Data** in the tree. If `Value` imported under a
different name, note it — you'll use that name in Part 2C and Part 5.

### Step 2 — Sanity check the connection

Insert a temporary Label anywhere and set `Text` to
`CountRows(Projects)`. It should show your real row count (Studio may
show a delegation warning — expected, ignore for this probe). Delete the
label when satisfied.

---

## Part 2 — Rewrite App.OnStart and App.Formulas

The filter card writes to the same variables (`filterSearch`,
`filterStatus`, …) — those don't change. What changes is the
**`filteredUseCases` named formula** (it now queries Dataverse, must be
delegation-safe, and must compare against choice `.Value`), and the
**typed-blank `selectedUC`** seed.

### Part 2A — Replace the `filteredUseCases` named formula

Click `App` in the tree → **Formulas**. Replace the existing
`filteredUseCases = Filter(colUseCases, …)` block with:

```powerfx
filteredUseCases =
    Filter(
        Projects,
        (filterSearch = ""
            || StartsWith('Use Case ID',   filterSearch)
            || StartsWith('Use Case Name', filterSearch)
            || StartsWith('AI Solution Owner Name', filterSearch))
        && (filterStatus = "All Statuses" || 'Project Status'.Value = filterStatus)
        && (filterSBU    = "All SBUs"    || 'Business Hierarchy'.'Strategic Business Unit' = filterSBU)
        && (filterFY     = "All FYs"     || 'Project Completion Fiscal Year' = filterFY)
        && (filterOwner  = ""            || 'AI Solution Owner Name' = filterOwner)
    );
```

What changed and why:

- `colUseCases` → `Projects`.
- **Search uses `StartsWith`, not `in`.** The `in` operator does **not
  delegate** to Dataverse — on a large table it would silently cap at 500
  rows and miss matches. `StartsWith` delegates. (Trade-off: it matches
  from the start of the field, not mid-string. If you need
  contains-anywhere search, use `Search(Projects, filterSearch, "..."
  )` as the data source instead — `Search` delegates and matches
  substrings.)
- Status compares `'Project Status'.Value = filterStatus` (choice label).
- SBU navigates the lookup:
  `'Business Hierarchy'.'Strategic Business Unit' = filterSBU`.
- The "All …" sentinel short-circuits each clause exactly as before.

> If a column display name has **no spaces** (rare here) you can drop the
> quotes. When in doubt, keep the single quotes — they're always valid.

### Part 2B — Re-seed `selectedUC` as a Dataverse-shaped blank

In `App.OnStart`, the old typed-blank used collection field names
(`{ UCID: "", Name: "", … }`). With Dataverse, replace it so the blank
has the **Projects** shape (otherwise `Set(selectedUC, ThisItem)` from the
gallery yields a record whose fields don't match the seed and you get
type errors on `srcDetail`):

```powerfx
// OLD: Set(selectedUC, { UCID:"", Name:"", ... });
Set(selectedUC, Defaults(Projects));
```

`Defaults(Projects)` returns an empty, correctly-typed Projects record so
every `selectedUC.'<column>'` reference resolves before a row is picked.

### Part 2C — Decide how Realized Value is computed

Projects has no `RealizedValue`; it lives in the **Value** child table
(`Realized Value`, one row per reporting period). Pick one:

**Option A (recommended) — Dataverse rollup column.** In your solution,
add a **Rollup column** on Projects, e.g. display name `Realized Value
Total`, defined as `SUM` of related `'Value'`.`Realized Value`. Then the
list reads one delegable field: `ThisItem.'Realized Value Total'`.
*Trade-off:* rollups recalculate on a schedule (~hourly) / on demand, not
instantly — fine for a list view.

**Option B (no schema change) — sum the related rows in the app.** If
Projects exposes a one-to-many relationship to Value (it does, via the
`Project` lookup), reference the related set:
`Sum(ThisItem.Values, 'Realized Value')` — replace `Values` with the
relationship name shown when you type `ThisItem.` in the formula bar.
*Trade-off:* aggregating per visible row is **not delegable** and gets
slow on large tables; acceptable for a few hundred projects.

This guide uses **Option A** (`'Realized Value Total'`) in Step 9. If you
chose B, substitute the `Sum(...)` expression there.

---

## Part 3 — Rewire the filter dropdowns

Open `srcList`. The captions and layout don't change — only the `Items`
of each dropdown (and one search note).

### Step 3 — Status dropdown (`ddStatus`)

The options must be the **Dataverse Project Status labels** (not the old
`colStatus` codes). Set:

| Property | Value |
|---|---|
| Items | `["All Statuses","Rationale for AI Solutions","Data Preparation","Development","Test and Validation","Deployment","Monitoring and Review","Decommissioning"]` |

`Default` (`filterStatus`) and `OnChange`
(`Set(filterStatus, Self.Selected.Value)`) stay as they are.

> These seven labels must match your Project Status choice **exactly**
> (see [`09-dataverse-schema.md` §5](09-dataverse-schema.md#5-choice-option-sets)).
> A mismatch means that status filters to zero rows. Alternatively, drive
> Items dynamically: `["All Statuses"]` is hard to prepend to
> `Choices('Project Status')`, so the static array above is simplest and
> guarantees order.

### Step 4 — SBU dropdown (`ddSBU`)

SBUs now come from the Business Hierarchy table. You can't simply prepend
`"All SBUs"` to a `Distinct(...)` result inline — `&` is string
concatenation in Power Fx and does **not** combine tables, so
`["All SBUs"] & Distinct(...)` errors. Use one of these two patterns:

**Pattern A (recommended) — build the option list once in `App.OnStart`.**
Add to the bottom of `App.OnStart`:

```powerfx
ClearCollect(colSBUOptions, "All SBUs");
Collect(colSBUOptions, Distinct('Business Hierarchy', 'Strategic Business Unit').Value);
```

`ClearCollect` with a scalar makes a single `Value` column; `Collect` of
`Distinct(...).Value` (a single-column table of scalars) appends the rest
as `Value` rows. Then set the dropdown:

| Property | Value |
|---|---|
| Items | `colSBUOptions` |

**Pattern B (simplest) — keep a static array** if your SBU set is stable:

| Property | Value |
|---|---|
| Items | `["All SBUs","PBB","Capital Markets","Wealth","Commercial","Direct Banking"]` |

Either way, `Default` (`filterSBU`) and `OnChange`
(`Set(filterSBU, Self.Selected.Value)`) stay unchanged.

> `Distinct` on Dataverse may show a delegation warning. SBU lists are
> small (one row per valid SBU/LOB combo), so the result is well under the
> 500-row limit and complete in practice. Building it once in `OnStart`
> (Pattern A) also avoids re-querying on every screen render.

### Step 5 — FY dropdown (`ddFY`)

Same prepend constraint as SBU. Either keep the fixed array (simplest):

| Property | Value |
|---|---|
| Items | `["All FYs","F26","F25","F24"]` |

…or build it in `App.OnStart` from the data (Pattern A style):

```powerfx
ClearCollect(colFYOptions, "All FYs");
Collect(colFYOptions, Distinct(Projects, 'Project Completion Fiscal Year').Value);
```

then set `ddFY.Items = colFYOptions`. `Default` (`filterFY`) and
`OnChange` (`Set(filterFY, Self.Selected.Value)`) unchanged.

### Step 6 — Owner dropdown (`ddOwner`)

| Property | Value |
|---|---|
| Items | `Distinct(Projects, 'AI Solution Owner Name')` |

`Default`, `AllowEmptySelection`, and `OnChange` are unchanged from
[`07-scrlist-guide.md` Step 18](07-scrlist-guide.md). (`Distinct` returns
a `Value` column, so `Self.Selected.Value` still works.)

### Step 7 — Search box (`txtSearch`)

No control change needed — `OnChange` still does
`Set(filterSearch, Self.Text)`. The delegation-safe matching now happens
in the `filteredUseCases` formula (Part 2A). Optionally update `HintText`
to `"Use Case ID, name, owner…"`.

---

## Part 4 — Rewire the gallery rows

The gallery `Items` is already `filteredUseCases`, which now points at
Projects — so the gallery re-binds automatically. You only change the
**row controls' Text/Fill** to the Dataverse columns. Enter the template
(chevron next to `galUseCases`).

### Step 8 — Text columns

Update each row label's `Text` (names from [`07` Step 27](07-scrlist-guide.md)):

| Control | Old Text | New Text |
|---|---|---|
| `lblUCID` | `ThisItem.UCID` | `ThisItem.'Use Case ID'` |
| `lblName` | `ThisItem.Name` | `ThisItem.'Use Case Name'` |
| `lblSBU` | `ThisItem.SBU` | `ThisItem.'Business Hierarchy'.'Strategic Business Unit'` |
| `lblOwner` | `ThisItem.Owner` | `ThisItem.'AI Solution Owner Name'` |
| `lblFY` | `ThisItem.FY` | `ThisItem.'Project Completion Fiscal Year'` |

### Step 9 — Realized Value (`lblValue`)

Using **Option A** from Part 2C:

```powerfx
If(
    ThisItem.'Realized Value Total' = 0,
    "—",
    "$" & Text(ThisItem.'Realized Value Total' / 1000000, "0.0") & "M"
)
```

(If you chose Option B, replace `ThisItem.'Realized Value Total'` in both
places with `Sum(ThisItem.Values, 'Realized Value')`.)

### Step 10 — Last Updated (`lblUpdated`)

Point the date diff at the system `Modified On` column:

```powerfx
With({d: DateDiff(ThisItem.'Modified On', Today())},
    Switch(true,
        d < 7,  Text(d) & " days ago",
        d < 30, Text(RoundDown(d/7, 0)) & " wks ago",
                Text(RoundDown(d/30, 0)) & " mo ago"
    )
)
```

### Step 11 — Status pill color (`circStatusDot.Fill`)

The Switch must now key off the **choice label** (`.Value`) using the
Dataverse Project Status labels. Replace the formula with:

```powerfx
Switch(ThisItem.'Project Status'.Value,
    "Rationale for AI Solutions", RGBA(110,110,110,1),
    "Data Preparation",           RGBA(110,110,110,1),
    "Development",                RGBA(31,111,178,1),
    "Test and Validation",        RGBA(197,139,26,1),
    "Deployment",                 RGBA(45,125,63,1),
    "Monitoring and Review",      RGBA(74,124,140,1),
    "Decommissioning",            RGBA(176,176,176,1),
    RGBA(110,110,110,1)
)
```

(Same colors as before — only the match strings changed from codes to
the choice labels. See the mapping in
[`09-dataverse-schema.md` §4](09-dataverse-schema.md#4-mapping-to-app-collections).)

### Step 12 — Status pill text (`lblStatusText.Text`)

The choice label is already human-readable, so the old `DataPrep` →
"Data Prep" Switch is no longer needed. Replace with:

```powerfx
ThisItem.'Project Status'.Value
```

### Step 13 — Gallery OnSelect (unchanged, but verify)

`galUseCases.OnSelect` still works as written:

```powerfx
Set(selectedUC, ThisItem);
Set(currentSection, "Info");
Navigate(srcDetail, ScreenTransition.None)
```

`ThisItem` is now a Projects row, so `selectedUC` becomes a Dataverse
record. (The `srcDetail` controls that read `selectedUC.Name` etc. will
need the same rename pass — out of scope here, see Part 7.)

---

## Part 5 — Title chip and footer counts

These used `CountRows(colUseCases)`. On Dataverse, an unfiltered
`CountRows(Projects)` is **not delegable** and will count only the first
loaded page. Use the patterns below.

### Step 14 — Page count chip (`lblPageCount`)

```powerfx
Text(CountRows(Projects)) & " use cases"
```

For tables under ~500 rows this is exact. For larger tables, maintain a
count in `App.OnStart` with `Set(gblProjectCount, CountRows(Projects))`
and show `gblProjectCount` (refresh on screen `OnVisible`), or accept the
"500+" approximation.

### Step 15 — Footer count label (`lblFooterCount`)

```powerfx
"Showing " & Text(CountRows(filteredUseCases)) & " of " & Text(CountRows(Projects))
```

### Step 16 — Footer scroll-status label (`lblScrollStatus`)

```powerfx
If(CountRows(filteredUseCases) >= CountRows(Projects),
   "All " & Text(CountRows(Projects)) & " use cases shown",
   Text(CountRows(Projects) - CountRows(filteredUseCases)) & " hidden by filters")
```

---

## Part 6 — Delegation pass

Before you call it done, hunt blue-underlined warnings:

1. In the tree, anything with a **blue dotted underline** or a warning
   triangle = a formula that won't run server-side and silently caps at
   the delegation limit (default 500 rows).
2. **Raise the limit as a stopgap:** Settings → General → *Data row
   limit for non-delegable queries* → set to 2000 (max). This is a
   band-aid, not a fix — it just delays the cliff.
3. **The real fixes** are already in this guide: `StartsWith`/`Search`
   instead of `in` for search, choice `.Value` comparisons, and a rollup
   column for Realized Value. `Filter`, `Sort`, `StartsWith`, and `=`
   comparisons all delegate to Dataverse.
4. `Distinct` and `CountRows` over the whole table are the remaining
   non-delegable spots; they're acceptable here because the SBU list is
   tiny and the count tolerates approximation. Note them and move on.

---

## Part 7 — Sanity check

Press **F5** and walk through:

- [ ] Gallery shows your real Projects rows (not the 10 samples).
- [ ] UCID, name, SBU (resolved through the lookup), owner, status pill,
      FY, Realized Value, and "N days ago" all populate from live data.
- [ ] Status pill is the right color and the label reads the full
      Dataverse text (e.g. "Data Preparation").
- [ ] Search by Use Case ID / name / owner narrows the list.
- [ ] Status / SBU / FY / Owner dropdowns filter correctly; "All …"
      options show everything.
- [ ] Reset returns the full list.
- [ ] Footer reads "Showing X of Y" and flips to "N hidden by filters".
- [ ] Clicking a row navigates to `srcDetail` with `selectedUC`
      populated (detail fields may show blank until you migrate
      `srcDetail` — expected).
- [ ] No blue delegation underline on `galUseCases.Items`,
      `txtSearch`, or the filter dropdowns.

When these pass, `srcList` is fully on Dataverse. Then **drop the
`ClearCollect(colUseCases, …)` block** from `App.OnStart` (it's now dead
data) — but only after `srcDetail`/`srcNew` no longer reference
`colUseCases`.

---

## What's next (not in this guide)

- **`srcDetail`** reads `selectedUC.Name`, `.UCID`, `.Status`, etc. and
  the Edit form `frmInfo` is bound to `colUseCases`. Repeat this rename
  pass there: rebind the form to `Projects`, update each card, and swap
  the value/governance galleries to the `'Value'` table.
- **`srcNew`** insert flow: replace `Collect(colUseCases, …)` with
  `Patch(Projects, Defaults(Projects), {...})`, resolving the SBU/LOB
  pick to the Business Hierarchy lookup before submit.
- **Value child galleries / rollups** on the detail screen bind to
  `'Value'` filtered by the `Project` lookup.
- Keep the [`03-formulas-reference.md` §N cheat sheet](03-formulas-reference.md)
  open — it lists the table find/replace map for the remaining screens.

---

## Quick reference — things that go wrong

| Symptom | Cause | Fix |
|---|---|---|
| Gallery is empty after rebind | `filteredUseCases` still references `colUseCases`, or a filter clause compares a choice without `.Value` | Repaste Part 2A. Confirm `'Project Status'.Value = filterStatus`, not `'Project Status' = filterStatus`. |
| SBU column blank for every row | Lookup column not navigated, or the related Business Hierarchy row isn't set on the project | Use `ThisItem.'Business Hierarchy'.'Strategic Business Unit'`. If still blank, the project's lookup is empty in Dataverse. |
| `'Business Hierarchy'` not recognized | The lookup column has a different display name | Type `ThisItem.` in the formula bar and pick the actual lookup column from the suggestion list; use that quoted name. |
| Status pill always gray | Switch still keys off codes (`"DataPrep"`) or off the choice record instead of `.Value` | Use Step 11's label-based Switch on `ThisItem.'Project Status'.Value`. |
| Status filter returns zero rows | A label in `ddStatus.Items` doesn't match the Dataverse choice exactly | Copy labels verbatim from the choice definition (§5 of the schema doc). Watch for "Test and Validation" vs "Testing". |
| Realized Value always "—" | Rollup column hasn't recalculated yet, or Option B relationship name is wrong | Trigger the rollup (or wait for its schedule). For Option B, confirm the relationship name after `ThisItem.` (often plural, e.g. `Values`). |
| Blue underline on `galUseCases.Items` | `in` operator or non-delegable function in `filteredUseCases` | Use `StartsWith`/`Search` (Part 2A). Raise the row limit to 2000 as a stopgap. |
| `Value(...)` / `'Value'` errors in a formula | Table named `Value` collides with the `Value()` function | Always quote the table as `'Value'`; or rename the data source in **Data**. |
| `selectedUC.<field>` errors on srcDetail | OnStart seed still uses collection field names | Set `Set(selectedUC, Defaults(Projects))` in OnStart (Part 2B). Full srcDetail migration is separate. |
| "Last Updated" shows a huge number | `Modified On` is a DateTime, off by timezone, or record never saved | Expected for newly imported rows; `DateDiff` against `Today()` still works. Confirm you used `ThisItem.'Modified On'`. |
| Count chip stuck at 500 | `CountRows(Projects)` hit the delegation cap | For large tables, cache the count in `App.OnStart` or accept the cap (Part 5, Step 14). |
