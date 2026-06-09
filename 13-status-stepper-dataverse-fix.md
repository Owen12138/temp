# BOA — Fix the Status Stepper After Switching to Dataverse

The status stepper (the track + 7 circles that show where a use case sits
in its lifecycle, on `srcDetail`) stops highlighting once `selectedUC`
comes from **Dataverse** instead of the `colUseCases` sample collection.
This guide fixes it. It's standalone — every formula you need is here.

## Why it broke

The whole stepper is driven by one hidden label, `lblCurrentStatusIdx`,
whose `Text` resolves the **active step number (Order 1–7)**. The track
fill width and every circle's highlight read `Value(lblCurrentStatusIdx.Text)`.

Its original formula was:

```powerfx
LookUp(colStatus, Code = selectedUC.Status, Order)
```

Three things broke the moment `selectedUC` became a Dataverse **Projects**
record:

1. **Field renamed.** There's no `selectedUC.Status` anymore — the column
   is `selectedUC.'Project Status'`.
2. **It's an option set, not text.** `Project Status` is a Dataverse
   **choice**, so comparing it to the text `Code` throws "incompatible
   types" (and the column has no `.Value` — coerce with `Text(...)`).
3. **Labels don't match the codes.** `colStatus.Code` is `Rationale`,
   `DataPrep`, `Development`, `Testing`, `Deployment`, `Monitoring`,
   `Decommissioning`. The Dataverse choice labels are `Rationale for AI
   Solutions`, `Data Preparation`, `Development`, `Test and Validation`,
   `Deployment`, `Monitoring and Review`, `Decommissioning`. Only three
   match — so even after fixing 1 & 2, the lookup would miss most rows.

When the lookup misses, `lblCurrentStatusIdx.Text` is blank →
`Value("")` is blank → the fill width and every `ThisItem.Order <= …`
comparison fail → **no fill, no highlighted circles.**

## Confirm the diagnosis (optional, 20 seconds)

Temporarily set a label's `Text` to:

```powerfx
"status=[" & Text(selectedUC.'Project Status') & "] idx=[" & lblCurrentStatusIdx.Text & "]"
```

You'll see the real status label (e.g. `Development`) but an **empty
`idx`** — confirming the lookup isn't resolving. Delete the probe after.

---

## Step 1 — Add `DVStatus` to `colStatus` (App.OnStart)

`colStatus` needs a field that holds the **exact Dataverse choice label**
so we can match against it. Click `App` → **OnStart** and replace the
`ClearCollect(colStatus, …)` block with (this matches
[`01-app-setup.md` §3](01-app-setup.md), already updated there):

```powerfx
ClearCollect(colStatus,
    { Order: 1, Code: "Rationale",      Label: "Rationale",       DVStatus: "Rationale for AI Solutions", Color: gblTheme.Ink3 },
    { Order: 2, Code: "DataPrep",       Label: "Data Prep",       DVStatus: "Data Preparation",           Color: gblTheme.Ink3 },
    { Order: 3, Code: "Development",    Label: "Development",     DVStatus: "Development",                 Color: gblTheme.Info },
    { Order: 4, Code: "Testing",        Label: "Testing",         DVStatus: "Test and Validation",        Color: gblTheme.Warn },
    { Order: 5, Code: "Deployment",     Label: "Deployment",      DVStatus: "Deployment",                 Color: gblTheme.Ok },
    { Order: 6, Code: "Monitoring",     Label: "Monitoring",      DVStatus: "Monitoring and Review",      Color: gblTheme.Monitor },
    { Order: 7, Code: "Decommissioning",Label: "Decommissioning", DVStatus: "Decommissioning",            Color: gblTheme.Decom }
);
```

> The `DVStatus` strings must match your Project Status choice **exactly**
> (see [`09-dataverse-schema.md` §5](09-dataverse-schema.md#5-choice-option-sets)).
> Watch the renamed ones: **"Test and Validation"** (not "Testing"),
> **"Data Preparation"** (not "Data Prep"), **"Monitoring and Review"**
> (not "Monitoring"), **"Rationale for AI Solutions"** (not "Rationale").

**Re-run OnStart** so the new field takes effect: in the tree, `App` →
`…` → **Run OnStart** (or close and reopen the app).

---

## Step 2 — Fix the index lookup (`lblCurrentStatusIdx`)

Select `lblCurrentStatusIdx` (the hidden label on `srcDetail` inside
`conSectionInfo`). Change its **`Text`** to:

```powerfx
Coalesce(
    LookUp(colStatus, DVStatus = Text(selectedUC.'Project Status'), Order),
    0
)
```

What each piece does:

- **`Text(selectedUC.'Project Status')`** — coerces the choice to its
  label string (e.g. `"Development"`), which is the safe way to read a
  Dataverse choice (no `.Value`, no choice-vs-text comparison error).
- **`DVStatus = …`** — matches that label against the new field from
  Step 1, so the renamed statuses resolve correctly.
- **`Coalesce(…, 0)`** — if the status is blank or unmatched, the index
  falls back to `0` (stepper shows no progress) instead of blank, which
  would break the math downstream.

That's the core fix. The track, fill, circles, and labels all read
`Value(lblCurrentStatusIdx.Text)` and need **no change** — they work the
moment the index resolves.

---

## Step 3 — (Recommended) clamp the fill so index 0 can't go negative

The track-fill width (Step 30 of the srcDetail guide) is:

```powerfx
recStepTrack.Width * (Value(lblCurrentStatusIdx.Text) - 1) / (CountRows(colStatus) - 1)
```

At index `0` (blank/unmatched status) that's `(0-1)/6` → a **negative**
width. Guard it so it never goes below zero — set `recStepTrackFill.Width`
to:

```powerfx
recStepTrack.Width * Max(Value(lblCurrentStatusIdx.Text) - 1, 0) / (CountRows(colStatus) - 1)
```

(Only the `Max(… , 0)` is added.) Now a record with no/unknown status
shows an empty track instead of erroring.

---

## Step 4 — Sanity check

On `srcList`, click a use case whose status you know, to open `srcDetail`:

- [ ] The correct number of circles are filled maroon (up to and including
      the current status), the rest are outline.
- [ ] The fill line reaches the current circle.
- [ ] The active status label is bold.
- [ ] Open records at different statuses (e.g. one in "Development", one in
      "Monitoring and Review") — the stepper advances correctly for each.
- [ ] A record with an empty status shows an empty track, no error.

---

## What this does NOT cover

This fixes the stepper **display** for Dataverse. The rest of `srcDetail`
still references collection field names and needs the same migration pass
(separate work):

- The **Status edit form** (`ddStatus2` / the `Status` DataCard, Step 34
  of [`08-srcdetail-guide.md`](08-srcdetail-guide.md)) still reads/writes
  `colStatus.Code` against `selectedUC.Status`. On Dataverse the card's
  `Default` and `Update` must use the `Project Status` choice (bind to
  `Choices(Projects.'Project Status')` and write `.Selected`), and
  `selectedUC` is refreshed from `Projects`.
- Other detail fields (`selectedUC.Name`, `.UCID`, `.Owner`, value /
  governance galleries) need the same rename/rebind as in
  [`10-srclist-dataverse-guide.md`](10-srclist-dataverse-guide.md).

---

## Troubleshooting

| Symptom | Cause | Fix |
|---|---|---|
| Stepper still all-grey after the fix | OnStart not re-run, so `DVStatus` doesn't exist yet | App → Run OnStart (or reopen). Probe `First(colStatus).DVStatus` in a label — should show "Rationale for AI Solutions". |
| One specific status never highlights | Its `DVStatus` string doesn't match the real choice label | Compare to §5 of the schema doc; fix the string in Step 1. Common miss: "Testing" vs **"Test and Validation"**. |
| "Incompatible types" on `lblCurrentStatusIdx` | The column is compared without `Text(...)`, or `'Project Status'` isn't quoted/named right | Use `Text(selectedUC.'Project Status')`; confirm the column's display name via IntelliSense (`selectedUC.`). |
| `selectedUC.'Project Status'` shows blank | `selectedUC` isn't a Projects record (still the typed-blank, or set from the old collection) | Ensure `srcList`'s gallery `OnSelect` does `Set(selectedUC, ThisItem)` on the `Projects`-bound gallery, and the OnStart seed is `Set(selectedUC, Defaults(Projects))`. |
| Fill bar disappears / errors for a blank-status record | Index 0 makes the width negative | Apply Step 3's `Max(…, 0)` clamp. |
