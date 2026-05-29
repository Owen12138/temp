# BOA Canvas App — Power Fx Formula Reference

Every formula in one place, organized by purpose. Copy/paste into Studio
as you go.

---

## A. Filters & list query

```powerfx
// Filtered gallery items (Named formula in App.Formulas)
filteredUseCases =
    Filter(
        colUseCases,
        (filterSearch = "" ||
         filterSearch in UCID ||
         filterSearch in Name ||
         filterSearch in Owner) &&
        (filterStatus = "All Statuses" || Status = filterStatus) &&
        (filterSBU    = "All SBUs"    || SBU    = filterSBU) &&
        (filterFY     = "All FYs"     || FY     = filterFY)
    );
```

```powerfx
// Reset filters
Set(filterSearch, "");
Reset(txtSearch);
Set(filterStatus, "All Statuses");
Set(filterSBU, "All SBUs");
Set(filterFY, "All FYs")
```

```powerfx
// "Showing N of M" footer label
// (gallery uses Items = filteredUseCases; all matching rows are loaded
// at once — there is no paging/offset, so this is just current vs total)
"Showing " & Text(CountRows(filteredUseCases)) &
" of "     & Text(CountRows(colUseCases))
```

```powerfx
// Filter-status label (right side of footer)
// Used to say "Scroll to load more", which was wrong: scrolling doesn't
// load anything because the gallery already has every filteredUseCases
// row. Now reports filter state instead — hidden rows are filtered, not
// pending.
If(CountRows(filteredUseCases) >= CountRows(colUseCases),
   "All " & Text(CountRows(colUseCases)) & " use cases shown",
   Text(CountRows(colUseCases) - CountRows(filteredUseCases)) & " hidden by filters"
)
```

---

## B. Status pill (color mapping)

Components cannot access global collections, so use `Switch` instead of
`LookUp(colStatus, ...)` inside `cmpStatusPill`.

```powerfx
// pillDot Fill  (paste into the Circle's Fill property inside the component)
Switch(cmpStatusPill.InputStatus,
    "Rationale",       RGBA(110,110,110,1),
    "DataPrep",        RGBA(110,110,110,1),
    "Development",     RGBA(31,111,178,1),
    "Testing",         RGBA(197,139,26,1),
    "Deployment",      RGBA(45,125,63,1),
    "Monitoring",      RGBA(74,124,140,1),
    "Decommissioning", RGBA(176,176,176,1),
    RGBA(110,110,110,1)
)
```

```powerfx
// pillLabel Text  (only DataPrep needs remapping; all others display as-is)
Switch(cmpStatusPill.InputStatus,
    "DataPrep", "Data Prep",
    cmpStatusPill.InputStatus
)
```

For inline pills outside the component (e.g. Value table signoff):

```powerfx
// Color
Switch(ThisItem.Signoff,
    "Signed off", gblTheme.Ok,
    "Pending",    gblTheme.Warn,
    "Not started", gblTheme.Ink4,
    gblTheme.Ink4
)

// Label
ThisItem.Signoff
```

---

## C. Realized value rollups

```powerfx
// In App.Formulas
ucValueRows = Filter(colValueEntries, UCID = selectedUC.UCID);

realizedYTD =
    Sum(
        Filter(ucValueRows, Signoff = "Signed off"),
        Amount
    );

estimatedTotal =
    Sum(ucValueRows, Amount);
```

Format for display:

```powerfx
// "$X.XM" or "—"
If(realizedYTD = 0, "—",
   "$" & Text(realizedYTD/1000000, "0.0") & "M")
```

---

## D. "Last updated" relative time

```powerfx
// Used in the gallery row
Switch(true,
    DateDiff(ThisItem.LastUpdated, Today()) = 0,  "Today",
    DateDiff(ThisItem.LastUpdated, Today()) < 7,  Text(DateDiff(ThisItem.LastUpdated, Today())) & " days ago",
    DateDiff(ThisItem.LastUpdated, Today()) < 30, Text(RoundDown(DateDiff(ThisItem.LastUpdated, Today())/7, 0)) & " wks ago",
    Text(RoundDown(DateDiff(ThisItem.LastUpdated, Today())/30, 0)) & " mo ago"
)
```

---

## E. Status stepper (Use Case Info section)

The stepper is driven entirely by `colStatus` (built in App.OnStart),
which carries `Order` 1–7 for Rationale … Decommissioning. The gallery
`Items` is just `colStatus`, `TemplateSize` is `Self.Width /
CountRows(colStatus)`, and the active-step index comes from:

```powerfx
// Hidden helper label lblCurrentIdx.Text on the section
LookUp(colStatus, Code = selectedUC.Status, Order)
```

Track geometry (so the line threads through the circle centers):

```powerfx
// recStepTrack — inactive line, first-center to last-center
X:     galStepper.X + galStepper.Width / (CountRows(colStatus) * 2)
Width: galStepper.Width * (CountRows(colStatus) - 1) / CountRows(colStatus)

// recStepTrackFill — maroon fill up to the current step
Width: recStepTrack.Width * (Value(lblCurrentIdx.Text) - 1) / (CountRows(colStatus) - 1)
```

Within the stepper gallery template:

```powerfx
// Circle fill — done OR current
If(ThisItem.Order <= lblCurrentIdx.Text, gblTheme.Maroon, White)

// Circle border
If(ThisItem.Order <= lblCurrentIdx.Text, gblTheme.Maroon, gblTheme.BorderStrong)

// Number/text color
If(ThisItem.Order <= lblCurrentIdx.Text, White, gblTheme.Ink3)

// Label color
If(ThisItem.Order <  lblCurrentIdx.Text, gblTheme.Ink2,
   ThisItem.Order = lblCurrentIdx.Text, gblTheme.Maroon,
   gblTheme.Ink3)

// Connector line fill (between steps)
If(ThisItem.Order < lblCurrentIdx.Text, gblTheme.Maroon, gblTheme.BorderStrong)
```

---

## F. Section navigation (detail page)

```powerfx
// Section nav gallery Items
Table(
    {Key:"Info",     Label:"Use Case Info",     Num: Blank()},
    {Key:"Contacts", Label:"Contacts",          Num: Blank()},
    {Key:"Value",    Label:"Value",             Num: CountRows(ucValueRows)},
    {Key:"Funds",    Label:"Funds",             Num: Blank()},
    {Key:"Gov",      Label:"Governance",        Num: Text(govDoneCount) & "/" & Text(govTotalCount)},
    {Key:"Tech",     Label:"Technical Review",  Num: Blank()},
    {Key:"Updates",  Label:"Monthly Update",    Num: Blank()}
)

// Row OnSelect
Set(currentSection, ThisItem.Key)

// Section visibility (on each conSection*)
currentSection = "Info"   // etc.

// Active row accent
If(ThisItem.Key = currentSection, gblTheme.Maroon, RGBA(0,0,0,0))

// Active row text color
If(ThisItem.Key = currentSection, gblTheme.Maroon, gblTheme.Ink2)

// Active row weight
If(ThisItem.Key = currentSection, FontWeight.Bold, FontWeight.Normal)
```

---

## G. Field write-back (Patch back into the row)

Every form input on the Detail page follows this pattern. Example for the
Use Case Name TextInput:

```powerfx
// OnChange
Patch(colUseCases, selectedUC, { Name: Self.Text });
Set(selectedUC, LookUp(colUseCases, UCID = selectedUC.UCID))
```

For Dropdowns:

```powerfx
Patch(colUseCases, selectedUC, { SBU: Self.Selected.Value });
Set(selectedUC, LookUp(colUseCases, UCID = selectedUC.UCID))
```

For DatePickers:

```powerfx
Patch(colUseCases, selectedUC, { TargetDate: Self.SelectedDate });
Set(selectedUC, LookUp(colUseCases, UCID = selectedUC.UCID))
```

---

## H. Governance / Tech Review — checklist toggle

```powerfx
// Action button OnSelect (inside the row template)
Patch(colGovernance, ThisItem, { Done: !ThisItem.Done })

// Button text
If(ThisItem.Done, "Completed", "Mark complete")

// Checkbox fill
If(ThisItem.Done, gblTheme.Ok, White)

// Checkbox border
If(ThisItem.Done, gblTheme.Ok, gblTheme.BorderStrong)

// "✓" label visibility
ThisItem.Done

// Progress label
Text(govDoneCount) & " of " & Text(govTotalCount) & " complete"

// Progress bar fill width
80 * govProgressPct
```

---

## I. Value table — Add / Edit modal

```powerfx
// "+ Add value entry" OnSelect
Set(editingValueRow,
    Defaults(colValueEntries));
Set(showValueModal, true)

// Edit row OnSelect
Set(editingValueRow, ThisItem);
Set(showValueModal, true)

// Modal "Save" OnSelect
If(
    IsBlank(LookUp(colValueEntries, Period = editingValueRow.Period &&
                                    Driver = editingValueRow.Driver &&
                                    UCID   = selectedUC.UCID)),
    // new row
    Collect(colValueEntries,
        Patch(editingValueRow, { UCID: selectedUC.UCID })
    ),
    // existing — patch by full record reference
    Patch(colValueEntries,
        LookUp(colValueEntries,
               UCID = selectedUC.UCID &&
               Period = editingValueRow.Period &&
               Driver = editingValueRow.Driver),
        {
            Amount: editingValueRow.Amount,
            Frequency: editingValueRow.Frequency,
            Signoff: editingValueRow.Signoff
        }
    )
);
Set(showValueModal, false)

// Modal "Cancel" OnSelect
Set(showValueModal, false)
```

---

## J. New Use Case flow

```powerfx
// Screen.OnVisible — fresh draft
Set(newUC, Defaults(colUseCases));
Set(newUC, Patch(newUC, {
    Status: "Rationale",
    FY: "F26",
    Owner: currentUser.FullName
}))
```

```powerfx
// btnCreate.OnSelect
Set(nextUCID,
    "UC-" &
    Text(
        Max(
            AddColumns(colUseCases, NumPart, Value(Mid(UCID, 4, 4))),
            NumPart
        ) + 1,
        "0000"
    )
);

Collect(colUseCases,
    Patch(newUC,
        {
            UCID: nextUCID,
            LastUpdated: Today(),
            RealizedValue: 0
        }
    )
);

Set(selectedUC, LookUp(colUseCases, UCID = nextUCID));
Set(currentSection, "Info");
Notify("Use case " & nextUCID & " created.", NotificationType.Success);
Navigate(srcDetail, ScreenTransition.None)
```

Validation gate before allowing Create (set `btnCreate.DisplayMode`):

```powerfx
If(
    !IsBlank(newUC.Name) &&
    !IsBlank(newUC.ProblemStatement) &&
    !IsBlank(newUC.Type) &&
    !IsBlank(newUC.Status) &&
    !IsBlank(newUC.SBU) &&
    !IsBlank(newUC.Owner),
    DisplayMode.Edit,
    DisplayMode.Disabled
)
```

---

## K. Submit Assessment (calls Power Automate when wired)

For v1 (no flow yet):

```powerfx
// btnSubmit.OnSelect
Patch(colUseCases, selectedUC, { LastUpdated: Today() });
Notify("Assessment submitted. (Power Automate flow will fire in production.)",
       NotificationType.Success)
```

For production with a flow named `BOA_SubmitAssessment`:

```powerfx
Set(submitResult,
    'BOA_SubmitAssessment'.Run(
        selectedUC.UCID,
        selectedUC.Name,
        selectedUC.Status,
        selectedUC.SBU,
        selectedUC.Owner,
        selectedUC.EstimatedValue,
        selectedUC.RealizedValue,
        LookUp(colUseCases, UCID = selectedUC.UCID, ProblemStatement),
        Toggle_NotifyTeams.Value
    )
);

If(submitResult.success = "true",
    Notify("Submitted. Teams notification sent.", NotificationType.Success),
    Notify("Submit failed: " & submitResult.error, NotificationType.Error)
)
```

The Power Automate flow itself (built later in make.powerautomate.com):

1. **Trigger**: Power Apps (V2) — inputs match the `.Run()` parameters above.
2. **Update row** in Dataverse `cibc_usecase` table (UCID match).
3. **Get a row** for the Excel master file (OneDrive/SharePoint).
4. **Add a row** with the updated values.
5. **Refresh Power BI dataset** (Power BI connector).
6. **Condition**: if NotifyTeams = true → **Post adaptive card** in the AAI Leadership Teams channel.
7. **Respond to Power Apps**: `{ success: "true", error: "" }`.

---

## L. People picker (Contacts section, future)

Drop a Combobox bound to `Office365Users`:

```powerfx
// Items
Office365Users.SearchUserV2({searchTerm: cmbOwner.SearchText}).value

// DisplayFields
["DisplayName", "Mail"]

// SearchFields
["DisplayName", "Mail"]

// OnChange
Patch(colUseCases, selectedUC,
    { Owner: cmbOwner.Selected.DisplayName,
      OwnerEmail: cmbOwner.Selected.Mail,
      OwnerInitials:
        Left(cmbOwner.Selected.GivenName, 1) &
        Left(cmbOwner.Selected.Surname, 1) }
)
```

Avatar circle inside the person card:

```powerfx
// Initials label
selectedUC.OwnerInitials

// Fill (could also use the user's profile image via Office365Users.UserPhotoV2)
gblTheme.Maroon
```

---

## M. Theme application — copy-paste table

Use these for every control you create:

| Control type   | Property         | Default value                            |
|----------------|------------------|------------------------------------------|
| Button (primary)   | Fill             | `gblTheme.Maroon`                           |
|                | HoverFill        | `gblTheme.MaroonDeep`                       |
|                | Color            | `White`                                  |
|                | BorderColor      | `gblTheme.Maroon`                           |
| Button (secondary) | Fill             | `White`                                  |
|                | HoverFill        | `gblTheme.MaroonLight`                      |
|                | Color            | `gblTheme.Maroon`                           |
|                | BorderColor      | `gblTheme.Maroon`                           |
| TextInput      | BorderColor      | `gblTheme.BorderStrong`                     |
|                | FocusedBorderColor | `gblTheme.Maroon`                         |
|                | Color            | `gblTheme.Ink`                              |
|                | Fill             | `White`                                  |
| Dropdown       | BorderColor      | `gblTheme.BorderStrong`                     |
|                | Color            | `gblTheme.Ink`                              |
|                | Fill             | `White`                                  |
|                | SelectionColor   | `gblTheme.Maroon`                           |
| Label (field)  | Color            | `gblTheme.Ink2`                             |
|                | Size             | `13`                                     |
|                | Weight           | Semibold                                 |
| Label (hint)   | Color            | `gblTheme.Ink3`                             |
|                | Size             | `11`                                     |
| Container (card) | Fill           | `White`                                  |
|                | BorderColor      | `gblTheme.Border`                           |
|                | BorderThickness  | `1`                                      |
|                | RadiusTopLeft etc.| `4`                                     |

Apply once and you have visual consistency across all screens.

---

## N. Dataverse migration cheat sheet

When you flip from collections to Dataverse:

1. **Create the tables** (use the schema in the Dataverse solution doc).
2. **Add the data source** in Studio: Data → Add data → search your table name.
3. **Find/replace** across all formulas:
   - `colUseCases`     → `cibc_usecase` (or your prefix)
   - `colValueEntries` → `cibc_valueentry`
   - `colGovernance`   → `cibc_governancecheck`
   - `colTechReview`   → `cibc_technicalreview`
4. **Re-bind columns** with their Dataverse logical names (Power Apps will
   prompt for each one — `Name` → `cibc_name`, etc.). Names match
   case-insensitively in formulas, so most things just work.
5. **Replace** `Collect(...)` with `Patch(<table>, Defaults(<table>), {...})`
   to insert.
6. **Replace** in-memory aggregations with delegable Dataverse queries —
   `Sum`, `Filter` and `Search` all delegate against Dataverse cleanly.
7. **Drop the sample data ClearCollect** calls from `App.OnStart`.

---

## O. Things you'll want to add later (not blocking v1)

- Infinite scroll is already the default (gallery shows all `filteredUseCases` rows; Canvas virtualises off-screen rows automatically). If you switch to a Dataverse source with > 500 rows, add `StartableInfiniteLoad` or use the `galUseCases.ScrollTo()` pattern.
- Bulk export to Excel: trigger a Power Automate flow with the filtered set.
- Real-time auto-save: timer + `Patch` every 30s using a dirty-flag variable.
- Optimistic locking: store `modifiedOn` and `If(...)` -check before Patch on Submit.
- "Recycle Bin" screen: filter Dataverse on `statecode = Inactive`.
- Power BI tile embed for the "Use Case Tracker" landing button.
- Office365Users.MyPhoto() for the user avatar in the header.
