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
| `Status` | `Project Status` | **Choice** | `ThisItem.'Project Status'` (no `.Value` — see note) |
| `FY` | `Project Completion Fiscal Year` | Text | `ThisItem.'Project Completion Fiscal Year'` |
| `RealizedValue` | *(none on Projects)* — rolled up from `'Value'` | — | see [Part 2C](#part-2c--decide-how-realized-value-is-computed) |
| `EstimatedValue` | `Estimated Monetary Benefit` | Currency | `ThisItem.'Estimated Monetary Benefit'` |
| `LastUpdated` | `Modified On` (system column) | DateTime | `ThisItem.'Modified On'` |
| `Type` | `Type of Use Case` | Choice | `ThisItem.'Type of Use Case'` (no `.Value`) |

> **Choice (option set) columns are an `OptionSetValue`, not text, and
> have NO `.Value` property.** This trips everyone up. Three rules:
>
> - **Display:** reference the column **directly** — `ThisItem.'Project
>   Status'` shows "Development". `ThisItem.'Project Status'.Value`
>   **errors** (`.Value` exists only on `Choices(...)` rows and on a
>   dropdown's `.Selected.Value`, never on the column).
> - **Compare against text** (a dropdown selection, a label): wrap the
>   column in **`Text(...)`** to coerce it to its label, then compare
>   text-to-text — `Text('Project Status') = ddStatus.Selected.Value`.
>   This is what the inline gallery filter uses (Step 7b) because the
>   dropdown holds label *strings*. It is **not delegable** (a yellow
>   warning), which is fine under ~500 rows.
> - **Compare delegably** (large tables): compare `OptionSetValue` to
>   `OptionSetValue` — either an option-set member
>   (`'Project Status' = 'Project Status (Projects)'.'Development'`) or a
>   choice picked from a combo box bound to `Choices(...)`
>   (`'Project Status' = comboStatus.Selected`). **Never** compare the
>   column to a `Choices(...)` *record* (e.g. `= LookUp(Choices(...), …)`)
>   — that throws "incompatible types: OptionSetValue and Record".
>
> This guide uses the `Text(...)` form (simplest, fine for this data
> size); the delegable combo-box alternative is noted in Step 3.
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

## Part 2 — App.OnStart and inline filtering

We're using **inline filtering**: the gallery's `Items` property holds the
whole `Filter(...)` expression and reads the filter controls (`txtSearch`,
`ddStatus`, …) **directly**. There is **no `filteredUseCases` named
formula** and **no `filterSearch`/`filterStatus`/… variables**. The
gallery recomputes automatically whenever a control changes, which keeps
all the filter logic in one place and removes the `OnChange`/`Set`
plumbing (and a class of state-drift bugs that comes with it).

So `App.OnStart` only needs to seed the typed-blank `selectedUC` and UI
state (`currentSection`, `sideCollapsed`) — **not** filter state.

### Part 2A — Remove the old filter plumbing

If you built `srcList` from [`07-scrlist-guide.md`](07-scrlist-guide.md),
strip out the collection-era filter wiring (the inline `Filter` in Step
7b replaces all of it):

- **`App.Formulas`:** delete the entire
  `filteredUseCases = Filter(colUseCases, …)` named formula.
- **`App.OnStart`:** delete the filter-state seeds —
  `Set(filterSearch, "")`, `Set(filterStatus, "All Statuses")`,
  `Set(filterSBU, "All SBUs")`, `Set(filterFY, "All FYs")`,
  `Set(filterOwner, "")`. **Keep** `Set(currentSection, …)` and
  `Set(sideCollapsed, …)`.
- The dropdowns' `OnChange` handlers (`Set(filterX, …)`) are removed in
  Part 3; the Reset button is simplified in Step 7c.

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

## Part 3 — Set up the filter controls

Open `srcList`. The captions and layout don't change. For each control
you set its `Items` (where noted) and its `Default`, and — because we
filter inline — you **delete the `OnChange` handler** (the old
`Set(filterX, …)`). The gallery in Step 7b reads each control directly.

### Step 3 — Status dropdown (`ddStatus`)

The options must be the **Dataverse Project Status labels** (not the old
`colStatus` codes):

| Property | Value |
|---|---|
| Items | `["All Statuses","Rationale for AI Solutions","Data Preparation","Development","Test and Validation","Deployment","Monitoring and Review","Decommissioning"]` |
| Default | `"All Statuses"` |
| OnChange | *(delete it — leave blank)* |

The gallery filter (Step 7b) reads `ddStatus.Selected.Value` directly.

> These seven labels must match your Project Status choice **exactly**
> (see [`09-dataverse-schema.md` §5](09-dataverse-schema.md#5-choice-option-sets)).
> A mismatch means that status filters to zero rows — copy them verbatim,
> watching for "Test and Validation" vs. "Testing".
>
> **Delegable alternative.** The inline filter coerces the choice with
> `Text('Project Status')`, which is **not delegable**. If you outgrow
> ~500 projects, bind this dropdown to the choice instead —
> `ddStatus.Items = Choices(Projects.'Project Status')`,
> `AllowEmptySelection = true` — and compare `OptionSetValue` to
> `OptionSetValue` in Step 7b:
> `(IsBlank(ddStatus.Selected) || 'Project Status' = ddStatus.Selected)`.
> Compare to `ddStatus.Selected` (the choice), **never** to a
> `Choices(...)` record — that throws "incompatible types". Trade-off:
> you lose the explicit "All Statuses" row (clearing the dropdown = all).

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

Either way, set `Default` = `"All SBUs"` and **delete the `OnChange`**.
The gallery reads `ddSBU.Selected.Value` directly.

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

then set `ddFY.Items = colFYOptions`. Set `Default` = `"All FYs"` and
**delete the `OnChange`**. The gallery reads `ddFY.Selected.Value`.

### Step 6 — Owner dropdown (`ddOwner`)

For consistency with the other dropdowns, give Owner an explicit
`"All Owners"` row. You can't prepend it to `Distinct(...)` inline, so
build the list in `App.OnStart` (same Pattern A as SBU/FY):

```powerfx
ClearCollect(colOwnerOptions, "All Owners");
Collect(colOwnerOptions, Distinct(Projects, 'AI Solution Owner Name').Value);
```

Then set:

| Property | Value |
|---|---|
| Items | `colOwnerOptions` |
| Default | `"All Owners"` |
| OnChange | *(delete it — leave blank)* |

Leave `AllowEmptySelection` off. The gallery reads `ddOwner.Selected.Value`
and treats `"All Owners"` as "no owner filter" (Step 7b).

### Step 7 — Search box (`txtSearch`)

**Delete the `OnChange`** (`Set(filterSearch, …)`) — the gallery reads
`txtSearch.Text` directly. Optionally update `HintText` to
`"Use Case ID, name, owner…"`.

---

## Part 4 — Inline filter, then the gallery rows

### Step 7b — Bind the gallery to the inline filter

Select `galUseCases` itself (the gallery node, **not** a control inside
the template) and set its `Items` to the full filter:

```powerfx
Filter(
    Projects,
    (IsBlank(txtSearch.Text)
        || StartsWith('Use Case ID',   txtSearch.Text)
        || StartsWith('Use Case Name', txtSearch.Text)
        || StartsWith('AI Solution Owner Name', txtSearch.Text))
    && (ddStatus.Selected.Value = "All Statuses" || Text('Project Status') = ddStatus.Selected.Value)
    && (ddSBU.Selected.Value    = "All SBUs"    || 'Business Hierarchy'.'Strategic Business Unit' = ddSBU.Selected.Value)
    && (ddFY.Selected.Value     = "All FYs"     || 'Project Completion Fiscal Year' = ddFY.Selected.Value)
    && (ddOwner.Selected.Value  = "All Owners"  || 'AI Solution Owner Name' = ddOwner.Selected.Value)
)
```

How it works:

- **References controls directly** (`txtSearch.Text`,
  `ddStatus.Selected.Value`, …) so the gallery recomputes live — no
  variables, no `OnChange`.
- **Search uses `StartsWith`, not `in`.** `in` does **not** delegate to
  Dataverse (silently caps at 500 rows). `StartsWith` delegates from the
  start of the field. For contains-anywhere search, use the data source
  `Search(Projects, txtSearch.Text, "cr123_usecasename", …)` instead.
- **Status is a choice**, so `Text('Project Status')` coerces it to its
  label for the text comparison. This single clause is **not delegable**
  (a yellow warning) — fine under ~500 rows; see Step 3's delegable
  alternative otherwise.
- **SBU navigates the lookup**:
  `'Business Hierarchy'.'Strategic Business Unit'`.
- The `"All …"` sentinels (and `IsBlank` for the search box) short-circuit
  each clause when no filter is set.

### Step 7c — Reset button (`btnReset`)

With no variables, Reset just clears the controls back to their defaults:

```powerfx
Reset(txtSearch); Reset(ddStatus); Reset(ddSBU); Reset(ddFY); Reset(ddOwner)
```

Now build the row controls. Enter the template (chevron next to
`galUseCases`).

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

This is a per-row expression (delegation doesn't apply inside a gallery
row), so the simplest correct form is to coerce the choice with `Text(...)`
and switch on the label string:

```powerfx
Switch(Text(ThisItem.'Project Status'),
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

> The `Text(...)` wrap is what makes this work — `Switch` on the bare
> choice column would compare `OptionSetValue` to the string literals and
> error. The match strings must equal your choice labels exactly. (Same
> colors as before; see the status mapping in
> [`09-dataverse-schema.md` §4](09-dataverse-schema.md#4-mapping-to-app-collections).)

### Step 12 — Status pill text (`lblStatusText.Text`)

The choice label is already human-readable, so the old `DataPrep` →
"Data Prep" Switch is no longer needed. Reference the column **directly**
— it coerces to its label in a text context (this is the one place the
bare reference Just Works; only comparisons need the choice form):

```powerfx
ThisItem.'Project Status'
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

With inline filtering there's no `filteredUseCases` to count — reference
the gallery's own rows via `galUseCases.AllItems`:

```powerfx
"Showing " & Text(CountRows(galUseCases.AllItems)) & " of " & Text(CountRows(Projects))
```

### Step 16 — Footer scroll-status label (`lblScrollStatus`)

```powerfx
If(CountRows(galUseCases.AllItems) >= CountRows(Projects),
   "All " & Text(CountRows(Projects)) & " use cases shown",
   Text(CountRows(Projects) - CountRows(galUseCases.AllItems)) & " hidden by filters")
```

> `galUseCases.AllItems` is the gallery's currently-loaded (filtered) set.
> Under ~500 rows the gallery loads everything, so the count is exact. On
> very large tables it reflects only loaded rows — switch to the delegable
> filter design (Step 3 note) and a cached total if that becomes an issue.

---

## Part 6 — Delegation pass

Before you call it done, hunt blue-underlined warnings:

1. In the tree, anything with a **blue dotted underline** or a warning
   triangle = a formula that won't run server-side and silently caps at
   the delegation limit (default 500 rows).
2. **Raise the limit as a stopgap:** Settings → General → *Data row
   limit for non-delegable queries* → set to 2000 (max). This is a
   band-aid, not a fix — it just delays the cliff.
3. **Expected, acceptable warnings** in this design: the Status
   `Text('Project Status')` clause in the gallery `Items`, `Distinct` for
   the SBU/FY lists, and `CountRows` over the whole table. All are fine
   under ~500 rows. `Filter`, `Sort`, `StartsWith`, and `=` themselves
   delegate.
4. **The real fixes** already applied: `StartsWith` (not `in`) for search,
   and a rollup column for Realized Value. If you need the Status filter
   to delegate too, use the choice-native dropdown from the Step 3 note.

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
- [ ] `galUseCases.Items` shows a delegation warning **only** on the
      Status `Text(...)` clause (expected). No *other* unexpected
      underlines, and no red errors.

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
| Gallery is empty after rebind | `galUseCases.Items` still references `colUseCases`/`filteredUseCases`, or a clause is malformed | Repaste the inline `Filter` from Step 7b. The Status clause is `(ddStatus.Selected.Value = "All Statuses" \|\| Text('Project Status') = ddStatus.Selected.Value)`. |
| Filters don't react when I type / pick | A dropdown still has an old `OnChange` `Set(filterX, …)`, or the gallery references a variable instead of the control | Clear every dropdown/search `OnChange` (Part 3); the gallery `Items` must reference `txtSearch.Text` / `dd*.Selected.Value` directly (Step 7b). |
| `'...'.Value` / `'Project Status'.Value` errors with "name isn't valid" | A choice column has no `.Value` property — that's only on `Choices()` rows and `Dropdown.Selected.Value` | Drop `.Value`. Display: `ThisItem.'Project Status'`. Compare: choice-to-choice (Part 2A / Step 11). |
| "Incompatible types: OptionSetValue and Text" | Comparing the choice column directly to a string (`'Project Status' = "Development"` or `= ddStatus.Selected.Value`) | Wrap the column: `Text('Project Status') = ddStatus.Selected.Value`. |
| "Incompatible types: OptionSetValue and Record" | Comparing the column to a `Choices(...)` / `LookUp(Choices(...))` **record** | Don't compare to a record. Use `Text('Project Status') = <string>`, or in the delegable design compare to `ddStatus.Selected` (a choice) — never `.Selected.Value`. |
| SBU column blank for every row | Lookup column not navigated, or the related Business Hierarchy row isn't set on the project | Use `ThisItem.'Business Hierarchy'.'Strategic Business Unit'`. If still blank, the project's lookup is empty in Dataverse. |
| `'Business Hierarchy'` not recognized | The lookup column has a different display name | Type `ThisItem.` in the formula bar and pick the actual lookup column from the suggestion list; use that quoted name. |
| Status pill always gray | Switch still keys off old codes (`"DataPrep"`), or the column isn't wrapped in `Text()` so no branch matches | Use Step 11's `Switch(Text(ThisItem.'Project Status'), "<label>", …)` with the exact Dataverse labels. |
| Status filter returns zero rows | A label in `ddStatus.Items` doesn't match the Dataverse choice exactly | Copy labels verbatim from the choice definition (§5 of the schema doc). Watch for "Test and Validation" vs "Testing". |
| Realized Value always "—" | Rollup column hasn't recalculated yet, or Option B relationship name is wrong | Trigger the rollup (or wait for its schedule). For Option B, confirm the relationship name after `ThisItem.` (often plural, e.g. `Values`). |
| Blue underline on `galUseCases.Items` | A delegation warning. The Status `Text(...)` clause warns **by design**; the `in` operator would also warn | The Status warning is expected (Part 6). For search use `StartsWith` (Step 7b), not `in`. Raise the row limit to 2000 as a stopgap. |
| `Value(...)` / `'Value'` errors in a formula | Table named `Value` collides with the `Value()` function | Always quote the table as `'Value'`; or rename the data source in **Data**. |
| `selectedUC.<field>` errors on srcDetail | OnStart seed still uses collection field names | Set `Set(selectedUC, Defaults(Projects))` in OnStart (Part 2B). Full srcDetail migration is separate. |
| "Last Updated" shows a huge number | `Modified On` is a DateTime, off by timezone, or record never saved | Expected for newly imported rows; `DateDiff` against `Today()` still works. Confirm you used `ThisItem.'Modified On'`. |
| Count chip stuck at 500 | `CountRows(Projects)` hit the delegation cap | For large tables, cache the count in `App.OnStart` or accept the cap (Part 5, Step 14). |
