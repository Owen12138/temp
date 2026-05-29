# BOA Canvas App — Build Guide

How to use this document:
1. Build screens in order. Each screen has: a tree of controls, key properties
   set on each, and any per-control formulas.
2. Property values in `` `backticks` `` are Power Fx — paste them into the
   formula bar. Plain values (numbers, "strings", colors) are typed directly
   into the property panel.
3. All colors come from the `gblTheme` variable defined in App.OnStart
   (see `01-app-setup.md`).
4. Naming: every control gets a meaningful name. Power Apps barks at duplicate
   names across components, not across screens — but consistent naming makes
   the code scan-able.

---

## Reusable Component: cmpStatusPill

Create a component (Tree view → Components → New component):

> **Tip:** If you delete a component and immediately try to create a new one
> with the same name, Power Apps will say the name is already taken. Fix:
> save the app (Ctrl+S), close it, and reopen it. The deleted component is
> fully cleared and the name is free again.

- Name: `cmpStatusPill`
- Width: `120`, Height: `24`
- Custom property `InputStatus` (Input, Data type: Text)

Controls inside:

> **Note:** Components are sandboxed — they cannot access global collections
> like `colStatus` or variables like `gblTheme`. Use inline `Switch` expressions
> instead of `LookUp`.

| Control   | Name        | Properties |
|-----------|-------------|------------|
| Circle    | `pillDot`   | X: `0`, Y: `8`, Width: `8`, Height: `8`, BorderThickness: `0`, Fill: see formula below |
| Label     | `pillLabel` | X: `14`, Y: `0`, Width: `Parent.Width - 14`, Height: `24`, Text: see formula below, Color: `RGBA(74,74,74,1)`, Size: `12`, VerticalAlign: `VerticalAlign.Middle` |

`pillDot` Fill:
```powerfx
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

`pillLabel` Text:
```powerfx
Switch(cmpStatusPill.InputStatus,
    "DataPrep", "Data Prep",
    cmpStatusPill.InputStatus
)
```
(All other codes display as-is; only `DataPrep` needs remapping to "Data Prep".)

---

## Screen 1 · srcHome (Landing page)

Screen properties:
- Name: `srcHome`
- Fill: `gblTheme.Bg`

### Control tree

```
srcHome
├── conLeftRail            (Container — full-height side rail, sticky on every screen)
│   └── ... (see "Shared left rail" below)
├── conHeader              (Container — maroon header bar across the top)
│   ├── lblCIBC            (label "CIBC" + red dot)
│   ├── recDivider         (1px rectangle)
│   ├── lblAppName         ("Enterprise AI Hub")
│   ├── lblTitle           ("Business Opportunity Assessment", centered)
│   └── conUser            (right-aligned: avatar circle + "Welcome, Owen")
└── conLandingActions      (Container — centered, 3 large buttons)
    ├── btnNew             ("New Intake" — primary maroon)
    ├── btnList            ("View/Edit Use Cases" — outline)
    └── btnTracker         ("Use Case Tracker" — disabled-style)
```

### conHeader

Container, horizontal layout, sticky to top.

- X: `If(sideCollapsed, 64, 220)` — note: if your Studio version doesn't
  recompute this on screen entry, set in `OnVisible` instead via Set().
  Easier alternative: don't shift the header at all; put the rail on top of
  it with a higher Z order. (The HTML mock has the header behind the rail.)
- Y: `0`, Width: `Parent.Width - Self.X`, Height: `52`
- Fill: `gblTheme.Maroon`
- DropShadow: None

Children:

| Control    | Name        | Key properties |
|------------|-------------|----------------|
| Label      | `lblCIBC`   | X: `24`, Y: `16`, Width: `52`, Height: `20`, Text: `"CIBC"`, Color: `White`, Size: `13`, FontWeight: `Bold` |
| Rectangle  | `recCIBCDot`| X: `lblCIBC.X + lblCIBC.Width - 14`, Y: `22`, Width: `9`, Height: `9`, Fill: `gblTheme.CIBCRed` |
| Rectangle  | `recDivider`| X: `84`, Y: `16`, Width: `1`, Height: `20`, Fill: `RGBA(255,255,255,0.25)` |
| Label      | `lblAppName`| X: `96`, Y: `16`, Width: `200`, Height: `20`, Text: `"Enterprise AI Hub"`, Color: `RGBA(255,255,255,0.8)`, Size: `12` |
| Label      | `lblTitle`  | X: `Parent.Width/2 - 200`, Y: `16`, Width: `400`, Height: `20`, Text: `"Business Opportunity Assessment"`, Color: `White`, Size: `14`, FontWeight: `Semibold`, Align: `Center` |
| Circle     | `circUserAv`| X: `Parent.Width - 180`, Y: `12`, Width: `28`, Height: `28`, Fill: `RGBA(255,255,255,0.18)` |
| Label      | `lblUserInit`| X: `circUserAv.X`, Y: `circUserAv.Y`, Width: `28`, Height: `28`, Text: `currentUser.Initials`, Color: `White`, Size: `11`, Align: `Center`, VerticalAlign: `Center`, FontWeight: `Semibold` |
| Label      | `lblWelcome`| X: `Parent.Width - 145`, Y: `16`, Width: `130`, Height: `20`, Text: `"Welcome, " & First(Split(currentUser.FullName, " ")).Value`, Color: `RGBA(255,255,255,0.85)`, Size: `12` |

### conLandingActions

Horizontal container, centered.

- X: `(Parent.Width - Self.Width) / 2`, Y: `(Parent.Height - 52) / 2 + 52 - Self.Height/2`
- Width: `720`, Height: `72`
- LayoutDirection: Horizontal
- LayoutJustifyContent: Center
- LayoutAlignItems: Stretch
- Gap: `18`

Children (each 220×72):

| Button         | Name         | Text                  | Fill                | Color           | BorderColor       | OnSelect |
|----------------|--------------|------------------------|----------------------|------------------|--------------------|----------|
| `btnNew`       | New Intake             | `gblTheme.Maroon`       | `White`          | `gblTheme.Maroon`    | `Navigate(srcNew, ScreenTransition.None)` |
| `btnList`      | View/Edit Use Cases    | `White`              | `gblTheme.Maroon`   | `gblTheme.BorderStrong` | `Navigate(srcList, ScreenTransition.None)` |
| `btnTracker`   | Use Case Tracker       | `White`              | `gblTheme.Ink2`     | `gblTheme.BorderStrong` | `Notify("Coming soon", NotificationType.Information)` |

Each button: Size 15, FontWeight Bold, BorderThickness 1, Radius 4.
For `btnNew` HoverFill: `gblTheme.MaroonDeep`. For `btnList` HoverFill: `gblTheme.MaroonLight`.

### Screen.OnVisible

```powerfx
Set(currentSection, "Info");
```

---

## Shared "left rail" (appears on all screens)

Easiest approach: make this a **component** so you don't duplicate it 4 times.
Create `cmpLeftRail` with two output properties: nothing — it manipulates the
global variables directly.

- Width: `If(sideCollapsed, 64, 220)`
- Height: `App.Height` — use `App.Height` not `Parent.Height` inside a
  component; `Parent` doesn't resolve until the component is placed on a screen.
- Fill: `RGBA(255,255,255,1)` (components don't expose `gblTheme` — hardcode white)

**Fake the right border with a Rectangle inside the component** — components
have no `BorderColor` property:

| Control   | Name            | X                  | Y | Width | Height          | Fill                    | BorderThickness |
|-----------|-----------------|--------------------|---|-------|-----------------|-------------------------|-----------------|
| Rectangle | `recRailBorder` | `Parent.Width - 1` | `0` | `1` | `Parent.Height` | `RGBA(225,225,225,1)` | `0`             |

When you place `cmpLeftRail` on a screen, set the instance's:
- X: `0`, Y: `52` (below the 52px header)
- Height: `Parent.Height - 52` (here `Parent` resolves correctly to the screen)

### Children

Top toggle row (Height 56, BorderBottom via a thin rect):

| Control | Name | Properties |
|---------|------|------------|
| Icon (Menu) | `icnRailToggle` | X: `12`, Y: `10`, Width: `36`, Height: `36`, Icon: `Icon.Menu`, Color: `gblTheme.Ink2`, OnSelect: `Set(sideCollapsed, !sideCollapsed)` |
| Label | `lblRailTitle` | X: `58`, Y: `20`, Width: `150`, Height: `16`, Text: `"BOA MENU"`, Color: `gblTheme.Ink2`, Size: `10`, FontWeight: `Bold`, Visible: `!sideCollapsed` |

Three nav items as a Gallery (vertical, no nav arrows, blank template):

- Items: `Table(
    {Key:"Home",  Label:"Home",                  Count: Blank()},
    {Key:"List",  Label:"View/Edit Use Cases",   Count: CountRows(colUseCases)},
    {Key:"New",   Label:"New Use Case",          Count: Blank()}
  )`
- TemplateSize: `40`
- ShowScrollbar: false

Inside the gallery template:

> **Component color rule:** components cannot access `gblTheme` or `Transparent`.
> Use hardcoded RGBA values throughout. `Transparent` → `RGBA(0,0,0,0)`.

> **Default controls in the gallery template:** when you insert a vertical
> gallery, Power Apps pre-populates the template with four controls. Map them
> as follows — rename each one and update its properties:
>
> - **First Rectangle** (tall, full height) → rename to `recRailAccent` — the left accent bar
> - **Second Rectangle** (thin, Height 1, at the bottom) → rename to `recRowSeparator` — the row divider line, just set Fill to `RGBA(225,225,225,1)` and leave it
> - **NextArrow2** (the icon) → rename to `icnRailItem`
> - **Title2** (the label) → rename to `lblRailItem`
>
> You do not need to insert any new controls — just rename and update what is already there.
> To tell the two rectangles apart: click each one and check its Height in the formula bar.
> The tall one (Height = `Parent.TemplateHeight`) is the accent. The short one (Height `1`) is the separator.

| Control | Default name | Rename to | Properties |
|---------|-------------|-----------|------------|
| Rectangle (left accent) | first Rectangle | `recRailAccent` | X: `0`, Y: `0`, Width: `3`, Height: `Parent.TemplateHeight`, Fill: see formula below |
| Rectangle (separator) | second Rectangle | `recRowSeparator` | Y: `Parent.TemplateHeight - 1`, Width: `Parent.TemplateWidth`, Height: `1`, Fill: `RGBA(225,225,225,1)` |
| Icon | `NextArrow2` | `icnRailItem` | X: `14`, Y: `(Parent.TemplateHeight - 16) / 2`, Width: `16`, Height: `16`, Icon: `Switch(ThisItem.Key, "Home", Icon.Home, "List", Icon.DetailList, "New", Icon.Add)`, Color: `RGBA(74,74,74,1)` |
| Label | `Title2` | `lblRailItem` | X: `40`, Y: `0`, Width: `Parent.TemplateWidth - 80`, Height: `Parent.TemplateHeight`, Text: `ThisItem.Label`, Color: `RGBA(74,74,74,1)`, Size: `12`, VerticalAlign: `VerticalAlign.Middle`, Visible: `!sideCollapsed` |

`recRailAccent` Fill — detects active screen without using `gblTheme`:
```powerfx
If(
    ThisItem.Key = LookUp(
        Table(
            {K:"srcHome",   V:"Home"},
            {K:"srcList",   V:"List"},
            {K:"srcDetail", V:"List"},
            {K:"srcNew",    V:"New"}
        ),
        K = App.ActiveScreen.Name,
        V
    ),
    RGBA(122,26,46,1),
    RGBA(0,0,0,0)
)
```

`lblRailItem` Color — same active-screen logic:
```powerfx
If(
    ThisItem.Key = LookUp(
        Table(
            {K:"srcHome",   V:"Home"},
            {K:"srcList",   V:"List"},
            {K:"srcDetail", V:"List"},
            {K:"srcNew",    V:"New"}
        ),
        K = App.ActiveScreen.Name,
        V
    ),
    RGBA(122,26,46,1),
    RGBA(74,74,74,1)
)
```

Gallery OnSelect:
```powerfx
Switch(ThisItem.Key,
    "Home", Navigate(srcHome, ScreenTransition.None),
    "List", Navigate(srcList, ScreenTransition.None),
    "New",  Navigate(srcNew,  ScreenTransition.None)
)
```

Put `cmpLeftRail` on every screen, anchored X=0 Y=0.

---

## Screen 2 · srcList (Use Case List)

> **Build this screen using `07-scrlist-guide.md`** — it's the step-by-step
> version of this section, with numbered Insert steps, full property tables
> for every control, and a sanity check after each part. The summary below
> is kept as a fast reference only; don't try to build from it alone.

### Control tree (right of the rail)

```
srcList
├── cmpLeftRail
├── conHeader                  (same as Home)
├── conPage                    (content container, padding 24)
│   ├── conTitleRow            (page title + Export + New)
│   │   ├── lblPageTitle       ("View/Edit Use Cases")
│   │   ├── lblPageCount       ("142 use cases")
│   │   ├── lblPageSub         ("All AI and analytics use cases...")
│   │   ├── btnExport          ("Export to Excel" — secondary)
│   │   └── btnPageNew         ("New Use Case" — primary, Navigate srcNew)
│   ├── conFilterCard          (white card, 6-col grid)
│   │   ├── txtSearch
│   │   ├── ddStatus
│   │   ├── ddSBU
│   │   ├── ddFY
│   │   ├── ddOwner
│   │   └── btnReset
│   ├── conGallery             (white card)
│   │   ├── conGalleryHeader   (maroon row with column labels)
│   │   ├── galUseCases        (gallery; row per use case)
│   │   └── conGalleryFooter   (pagination + range)
└── (nothing else)
```

### conTitleRow

Horizontal container. Width: `Parent.Width - 220 - 48` (rail offset + padding).

Children sized inline:
- `lblPageTitle`: Text `"View/Edit Use Cases"`, Size 22, FontWeight Semibold, Color `gblTheme.Ink`.
- `lblPageCount`: Text `Text(CountRows(colUseCases)) & " use cases"`, Size 14, Color `gblTheme.Ink3`.
- `lblPageSub`: Text `"All AI and analytics use cases across CIBC. Click a row to view or edit."`, Size 13, Color `gblTheme.Ink3`.
- `btnExport`: secondary style (Fill White, Color gblTheme.Maroon, BorderColor gblTheme.Maroon). OnSelect: `Notify("Export queued. You'll get a Teams ping when the file is ready.", NotificationType.Success)` (real export goes through a Power Automate flow — see deferred flow spec in `03-formulas-reference.md`).
- `btnPageNew`: primary style. OnSelect: `Navigate(srcNew, ScreenTransition.None)`.

### conFilterCard

White card, padding 14×16, BorderRadius 4, Border `gblTheme.Border`.

| Control | Name | Items / Default / OnChange |
|---------|------|----------------------------|
| TextInput | `txtSearch` | HintText: `"UCID, name, owner…"`, OnChange: `Set(filterSearch, Self.Text)` |
| Dropdown | `ddStatus` | Items: `["All Statuses", "Rationale", "Data Prep", "Development", "Testing", "Deployment", "Monitoring", "Decommissioning"]`, Default: `filterStatus`, OnChange: `Set(filterStatus, Self.Selected.Value)` |
| Dropdown | `ddSBU` | Items: `Concat(["All SBUs"], colSBU)` (or just `["All SBUs"; "PBB"; "Capital Markets"; "Wealth"; "Commercial"; "Direct Banking"]`), Default: `filterSBU`, OnChange: `Set(filterSBU, Self.Selected.Value)` |
| Dropdown | `ddFY`     | Items: `["All FYs", "F26", "F25", "F24"]`, Default: `filterFY`, OnChange: `Set(filterFY, Self.Selected.Value)` |
| Dropdown | `ddOwner`  | Items: `Distinct(colUseCases, Owner)`, Default: `filterOwner`, AllowEmptySelection: `true`, OnChange: `Set(filterOwner, If(IsBlank(Self.Selected), "", Self.Selected.Value))` |
| Button | `btnReset` | Text: `"Reset"`, OnSelect: `Set(filterSearch,""); Reset(txtSearch); Set(filterStatus,"All Statuses"); Set(filterSBU,"All SBUs"); Set(filterFY,"All FYs"); Set(filterOwner,""); Reset(ddOwner)` |

Label each filter with a small caption label (Size 11, Bold, Color `gblTheme.Ink3`) above the control.

### conGalleryHeader

A 1-row container styled like a table header. Fill `gblTheme.Maroon`, Color White, Height 36, Font Size 12, FontWeight Semibold.

Columns (these widths must match the gallery template below):

| Label   | Width |
|---------|-------|
| UCID    | 100   |
| Use Case Name | 320 |
| SBU     | 120   |
| AI Solution Owner | 160 |
| Status  | 150   |
| FY      | 60    |
| Realized Value | 130 |
| Last Updated | 110 |
| (action) | 80   |

### galUseCases

- Items: `filteredUseCases`
- TemplateSize: 52
- TemplateFill: `If(Mod(ThisItem.Position, 2) = 0, RGBA(251,251,251,1), White)`
  - There is no built-in `Position` — use `WithIndex` or compute via parent:
    Items: `AddColumns(filteredUseCases, RowIdx, Sequence(CountRows(filteredUseCases)).Value)` — actually simpler:
    Items: `ShowColumns(filteredUseCases, "UCID","Name","SBU","Owner","Status","FY","RealizedValue","LastUpdated")`
    and just rely on TemplateFill staying white. Power Apps galleries don't
    cleanly stripe rows without a row-index column. Accept solid white if
    you want to keep this simple.

**No pagination gating** — set `galUseCases.Items = filteredUseCases` directly.
Canvas galleries are already virtualised (they only render rows in the viewport),
so there is no performance reason to paginate. All rows load immediately;
the user scrolls inside the gallery container to see more, exactly matching
the HTML's infinite-scroll behaviour.

OnSelect (on the gallery, not template):
```powerfx
Set(selectedUC, ThisItem);
Set(currentSection, "Info");
Navigate(srcDetail, ScreenTransition.None)
```

Template controls (each row), positioned with X equal to running sum of widths above:

| Control | Name | Properties |
|---------|------|------------|
| Label | `lblUCID`     | Text: `ThisItem.UCID`, Color: `gblTheme.Ink3`, Size: 12, X: 14 |
| Label | `lblName`     | Text: `ThisItem.Name`, FontWeight: Semibold, Color: `gblTheme.Ink`, Size: 13 |
| Label | `lblSBU`      | Text: `ThisItem.SBU`, Color: `gblTheme.Ink2`, Size: 13 |
| Label | `lblOwner`    | Text: `ThisItem.Owner`, Color: `gblTheme.Ink2`, Size: 13 |
| Component | `cmpStatusPill` | InputStatus: `ThisItem.Status` |
| Label | `lblFY`       | Text: `ThisItem.FY`, Color: `gblTheme.Ink2`, Size: 13 |
| Label | `lblValue`    | Text: `If(ThisItem.RealizedValue = 0, "—", "$" & Text(ThisItem.RealizedValue / 1000000, "0.0") & "M")`, Color: `gblTheme.Ink`, Size: 13 |
| Label | `lblUpdated`  | Text: `Switch(true, DateDiff(ThisItem.LastUpdated, Today()) < 7, Text(DateDiff(ThisItem.LastUpdated, Today())) & " days ago", DateDiff(ThisItem.LastUpdated, Today()) < 30, Text(RoundDown(DateDiff(ThisItem.LastUpdated, Today())/7, 0)) & " wks ago", Text(RoundDown(DateDiff(ThisItem.LastUpdated, Today())/30, 0)) & " mo ago")`, Color: `gblTheme.Ink2`, Size: 12 |
| Button | `btnView`    | Text: `"View"`, Style: outline maroon, OnSelect: `Select(Parent)` (so it triggers the gallery row's OnSelect) |

Add a 1px bottom rect inside the template (`Fill: gblTheme.Border`, Height: 1, Y: TemplateHeight-1) for the row separator.

### conGalleryFooter

White-grey footer. Two children, laid out with `LayoutJustifyContent: SpaceBetween`:

- **Left label** (`lblRepoCount`): count of visible rows vs. total.
  ```powerfx
  "Showing " & Text(CountRows(filteredUseCases)) & " of " & Text(CountRows(colUseCases))
  ```
  Size 12, Color `gblTheme.Ink3`.

- **Right label** (`lblScrollStatus`): filter status indicator.
  ```powerfx
  If(CountRows(filteredUseCases) >= CountRows(colUseCases),
     "All " & Text(CountRows(colUseCases)) & " use cases shown",
     Text(CountRows(colUseCases) - CountRows(filteredUseCases)) & " hidden by filters"
  )
  ```
  Size 12, FontWeight Semibold, Color `gblTheme.Ink2`. Prepend a small 7×7
  maroon circle (Fill `gblTheme.Maroon`, BorderRadius 999) to the left of this
  label as a status accent. (The HTML mock implied "scroll to load more"
  here; the Power Apps gallery sets `Items = filteredUseCases` and
  already has every matching row, so the label reports filter state
  rather than load state.)

No page-index variable, no Next/Prev buttons. The gallery's own scroll handles everything.

---

## Screen 3 · srcDetail (Detail page with side rail)

> **Build this screen using [`08-srcdetail-guide.md`](08-srcdetail-guide.md).**
> That file replaces this section with a 59-step walkthrough — Insert
> instructions for every control, full property tables, and a sanity
> check after each part. The summary below is kept as a reference for
> the overall control tree and section-by-section spec.

### Control tree

```
srcDetail
├── cmpLeftRail
├── conHeader                    (maroon header)
├── conActionBar                 (white sticky row under header)
│   ├── btnBack                  ("← Back to List")
│   ├── lblCrumb                 ("Repository / UC-0142 · Credit Risk Churn Predictor")
│   ├── lblSaveNote              ("Draft saved ✓ 14:32")
│   ├── btnSaveDraft
│   └── btnSubmit
├── conDetailLayout              (two-column: side rail + content)
│   ├── conSectionRail
│   │   ├── conRailHead          (UCID, name, status pill)
│   │   └── galSectionNav        (gallery of section links)
│   └── conContent               (vertical, scrolling)
│       ├── conSectionInfo       (Use Case Info)
│       ├── conSectionContacts
│       ├── conSectionValue
│       ├── conSectionFunds
│       ├── conSectionGov
│       ├── conSectionTech
│       ├── conSectionUpdates
│       └── conSubmitZone        (bottom row with Save + Submit)
```

Only one `conSection*` is visible at a time. Each section's `Visible` is
`currentSection = "<sectionKey>"`. (You could show all at once and use
anchor scrolling, but matching the prototype's behavior is cleaner.)

### conActionBar

Fill White, Height 56, BorderBottom via rect.

| Control | Name | Properties |
|---------|------|------------|
| Button (ghost) | `btnBack` | Text: `"← Back to List"`, Color: `gblTheme.Ink2`, Fill: Transparent, HoverColor: `gblTheme.Maroon`, OnSelect: `Navigate(srcList, ScreenTransition.None)` |
| Label | `lblCrumb` | Text: `"Repository  /  " & selectedUC.UCID & " · " & selectedUC.Name`, Color: `gblTheme.Ink3`, Size: 12 |
| Label | `lblSaveNote` | Text: `"Draft saved ✓ " & Text(Now(), "[$-en-US]hh:mm")`, Color: `gblTheme.Ink3`, Size: 11. (Wire to an actual save timestamp variable later — for v1 it's decorative.) |
| Button | `btnSaveDraft` | Secondary style, Text: `"Save Draft"`, OnSelect: `Notify("Draft saved.", NotificationType.Information)` |
| Button | `btnSubmit` | Primary style, Text: `"Submit Assessment"`, OnSelect: see Submit flow below |

Submit OnSelect:
```powerfx
// In production this calls a Power Automate flow.
// For demo: update the in-memory record and notify.
Patch(colUseCases, selectedUC, { LastUpdated: Today() });
Notify("Assessment submitted. Power Automate flow triggered.", NotificationType.Success);
```

### conSectionRail (left side rail inside detail)

Width 220, Fill White, BorderRight 1px.

`conRailHead` shows:
- Small label: `selectedUC.UCID & " · " & selectedUC.FY` — Color `gblTheme.Ink3`, Size 11.
- Big label: `selectedUC.Name` — Color `gblTheme.Ink`, Size 15, FontWeight Semibold.
- A `cmpStatusPill` with InputStatus = `selectedUC.Status`.

`galSectionNav` (vertical blank gallery):

Items:
```powerfx
Table(
    {Key:"Info",      Label:"Use Case Info",   Num: Blank()},
    {Key:"Contacts",  Label:"Contacts",        Num: Blank()},
    {Key:"Value",     Label:"Value",           Num: CountRows(ucValueRows)},
    {Key:"Funds",     Label:"Funds",           Num: Blank()},
    {Key:"Gov",       Label:"Governance",      Num: Text(govDoneCount) & "/" & Text(govTotalCount)},
    {Key:"Tech",      Label:"Technical Review",Num: Blank()},
    {Key:"Updates",   Label:"Monthly Update",  Num: Blank()}
)
```

TemplateSize: 40. OnSelect: `Set(currentSection, ThisItem.Key)`.

In the template:
- Left accent rect (Width 3, Fill: `If(ThisItem.Key = currentSection, gblTheme.Maroon, Transparent)`)
- Label for `ThisItem.Label` (Color: `If(ThisItem.Key = currentSection, gblTheme.Maroon, gblTheme.Ink2)`, FontWeight: `If(ThisItem.Key = currentSection, Bold, Normal)`)
- Right badge for `Num` if not blank — small pill, Fill `gblTheme.Border` or `White` when active.

### conSectionInfo

Visible: `currentSection = "Info"`. Fill White, Padding 24×28, BorderRadius 4.

**Section header**: a label `"Use Case Info"` (Color `gblTheme.Maroon`, Size 16, Bold) with a bottom border rectangle 2px in `gblTheme.Maroon`.

**Status stepper**: container, Fill `RGBA(250,250,250,1)`, Padding 10×14.
For each of the 7 statuses ("Rationale", "Data Prep", "Development", "Testing", "Deployment", "Monitoring", "Decommissioning"), draw:
- A circle (Width 18, Height 18) — Fill `If(ThisItem.Order <= currentStatusIndex, gblTheme.Maroon, White)`, with Border `gblTheme.BorderStrong`.
- A label with the step name — Color logic same.
- A connecting track line behind the circles — its filled portion is `If(ThisItem.Order < currentStatusIndex, gblTheme.Maroon, gblTheme.BorderStrong)`.

Use a horizontal Gallery for elegance, driven straight off `colStatus`
(it already carries `Order`, `Code`, `Label` for all seven statuses):
```powerfx
colStatus
```

Inside each row, calculate `currentIdx` once: add a hidden label
`lblCurrentIdx` outside the gallery with Text =
`LookUp(colStatus, Code = selectedUC.Status, Order)`. See
`08-srcdetail-guide.md` Steps 29–32 for the exact geometry — the track
endpoints and circle `Y` are derived so the line threads through every
circle's center and the circles sit centered on the line.

**Form fields** — use a Power Apps **Edit form** (`frmInfo`) bound to
`DataSource: colUseCases`, `Item: selectedUC`, `DefaultMode:
FormMode.Edit`, `Columns: 2`. Add a **Save** button (`SubmitForm`) and
set `frmInfo.OnSuccess = Set(selectedUC, frmInfo.LastSubmit)` so the
rail-head pill and stepper refresh on save. The cards (full-width ones
span both columns):

| Field | Card / Control | Default | Notes |
|-------|----------------|---------|-------|
| Use Case Name (req, full) | TextInput | `Parent.Default` | Card Width = `frmInfo.Width` |
| Problem Statement (full) | TextInput (multi-line) | `Parent.Default` | |
| AI Solution Description (full) | TextInput (multi-line) | `Parent.Default` | |
| Type of Use Case | Dropdown | Items: `colUseCaseType`, Default: `Parent.Default` | unlock card |
| Current Status | Dropdown | Items: `colStatus.Label`, Default: `LookUp(colStatus, Code = Parent.Default, Label)` | card **Update**: `LookUp(colStatus, Label = ddStatus2.Selected.Value, Code)` |
| SBU (req) | Dropdown | Items: `colSBU`, Default: `Parent.Default` | unlock card |
| LOB / Sub-LOB | TextInput | `Parent.Default` | |
| Completion Date (target) | DatePicker | `Parent.Default` | auto-generated |
| Refresh Frequency | Dropdown | Items: `colRefreshFreq`, Default: `Parent.Default` | unlock card |

Because `colUseCases` is a plain collection, the form generates text
inputs for the choice fields — unlock those cards, swap in a Dropdown,
and point the card's **Update** at the dropdown's `Selected.Value`
(for Status, write the **code** back via the LookUp above). *Other
LOBs Impacted*, *Output Deliverable*, and *Prerequisite* aren't stored
in v1; add them as custom cards only if you want them visible.

See `08-srcdetail-guide.md` Steps 33–34 for the full card-by-card
walkthrough.

### conSectionContacts

Visible: `currentSection = "Contacts"`.

6 "person card" fields in a 2-column grid. Each card is a 36-tall horizontal container with:

- Avatar circle 22×22 (Fill `gblTheme.Maroon`, Color White, initials text centered)
- Name label
- Subtitle label (email or role) aligned right (Color `gblTheme.Ink3`, Size 11)

The "empty" state is a single 36-tall container with a dashed-border "+" placeholder and gray "Assign a developer" text. To trigger the People picker, place an `Office365Users.SearchUser({searchTerm: ...})` -driven combobox inside, hidden until clicked.

For v1, hardcode the displayed users in `selectedUC` extensions
(`OwnerEmail`, `SponsorName`, etc.) — we can swap to the People picker pattern later.

### conSectionValue

Visible: `currentSection = "Value"`.

**Top row — 3 summary tiles** (horizontal container, Gap 12):

| Tile | Label | Value |
|------|-------|-------|
| Estimated Benefit | `"Estimated Benefit"` | `"$" & Text(selectedUC.EstimatedValue/1000000, "0.0") & "M"` |
| Realized · F26 YTD | `"Realized · F26 YTD"` | `"$" & Text(realizedYTD/1000000, "0.0") & "M"` (Color: `gblTheme.Maroon`) |
| Investment Spend | `"Investment Spend"` | (hardcode `"$0.4M"` for v1 — add Investment column to UseCase later) |

Each tile: BorderRadius 3, Border `gblTheme.Border`, Fill `RGBA(250,250,250,1)`, padding 10×14, label Size 11 Bold uppercase, value Size 18 Bold.

**Value table** (Gallery + header row):

Header row container (Fill `RGBA(245,245,245,1)`, Height 32) with column labels: FY · Q | Driver | Value | Frequency | Signoff | (action).

`galValueRows`:
- Items: `ucValueRows`
- TemplateSize: 44
- Template controls per column (sized to match header), with:
  - Label for `ThisItem.Period`
  - Label for `ThisItem.Driver`
  - Label for `ThisItem.Amount` — Text: `If(ThisItem.Amount = 0, "— TBD", "$" & Text(ThisItem.Amount/1000000, "0.0") & "M")`, FontWeight Semibold.
  - Label for `ThisItem.Frequency`
  - `cmpStatusPill` with input — but our pill takes status codes, so either extend it or do a small inline pill: a circle + label using the Signoff field directly with colors:
    - "Signed off" → green
    - "Pending" → amber
    - "Not started" → grey
  - Edit button (right-aligned) — OnSelect opens a popup (see Add/Edit modal below).

**Add row** at the bottom: a container with dashed top-border, button text `"+ Add value entry"` Color `gblTheme.Maroon`. OnSelect:
```powerfx
Set(editingValueRow, Defaults(colValueEntries));
Set(showValueModal, true);
```

**Value edit modal** (overlay container):

A full-screen overlay rectangle (Fill `RGBA(0,0,0,0.4)`, Visible: `showValueModal`) with a centered white card containing form fields for Period, Driver, Amount, Frequency, Signoff. OnSelect of "Save":
```powerfx
If(IsBlank(editingValueRow.UCID),
    // new row
    Collect(colValueEntries,
        Patch(editingValueRow, { UCID: selectedUC.UCID })
    ),
    // update existing
    Patch(colValueEntries, editingValueRow, { /* fields */ })
);
Set(showValueModal, false);
```

### conSectionFunds

Visible: `currentSection = "Funds"`.

Simple 2-column grid:

| Field | Control | Items / Default |
|-------|---------|-----------------|
| Funding Available | Dropdown | `["Yes — Business Case Funded"; "Partial"; "No"]` |
| Tracked by Finance Partner | Dropdown | `["Yes"; "No"]` |
| Estimated Monetary Benefit | TextInput | `"$" & Text(selectedUC.EstimatedValue/1000000, "0.0") & "M"` |
| Estimated Benefit Type | Dropdown | `colBenefitType` |
| Actualized Monetary Benefit | TextInput | `"$" & Text(selectedUC.RealizedValue/1000000, "0.0") & "M"` |
| Estimated Other Benefit | TextInput | text |

### conSectionGov

Visible: `currentSection = "Gov"`.

Section head: `"Governance"` + a right-aligned progress label and a progress bar.

Progress bar = 2 stacked rectangles:
- Background: Width 80, Height 6, Fill `RGBA(238,238,238,1)`, BorderRadius 3.
- Fill: Width `80 * govProgressPct`, Height 6, Fill `gblTheme.Maroon`.

**Checklist gallery** `galGovernance`:
- Items: `ucGovRows`
- TemplateSize: 56
- Template:
  - Check box: a Rectangle Width/Height 18, BorderRadius 2, Fill `If(ThisItem.Done, gblTheme.Ok, White)`, BorderColor `If(ThisItem.Done, gblTheme.Ok, gblTheme.BorderStrong)`. Add a label "✓" centered, Color White, Visible: `ThisItem.Done`.
  - Name label (Size 13, Bold)
  - Meta label (Size 11, Color `gblTheme.Ink3`)
  - Action button right-aligned: Text `If(ThisItem.Done, "Completed", "Mark complete")`. OnSelect:
    ```powerfx
    Patch(colGovernance, ThisItem, { Done: !ThisItem.Done })
    ```
- TemplateFill: White. Add a 1px bottom border rect per row.

### conSectionTech

Visible: `currentSection = "Tech"`.

Same checklist pattern as Governance, bound to `ucTechRows`.

After the gallery, a `"Performance Metrics"` multi-line TextInput field.

### conSectionUpdates

Visible: `currentSection = "Updates"`.

Single multi-line TextInput labeled `"Update for " & Text(Now(), "mmmm yyyy")` — initial value: `"Model is production-ready in shadow mode. Awaiting MRM sign-off..."`.

Hint label below: `"This update appears in the PDF report and the monthly Teams digest."`

**Toggle** (built-in Toggle control): Default ON. Label: `"Notify Executive Sponsor and AAI leadership via MS Teams when submitted"`. Color via FillSelected `gblTheme.Maroon`.

### conSubmitZone

A bottom-of-content row with:
- Message label (Size 12, Color `gblTheme.Ink3`): `"Save Draft keeps your changes without notifying anyone. Submit Assessment publishes the latest values and triggers Power Automate flows (Excel refresh, Power BI, Teams digest)."`
- `btnSaveDraft2` (secondary)
- `btnSubmit2` (primary) — same OnSelect as `btnSubmit` in the action bar.

### Screen.OnVisible

```powerfx
// If user lands here without selecting a use case (e.g. via deep link), bounce.
If(IsBlank(selectedUC), Navigate(srcList, ScreenTransition.None));
Set(currentSection, "Info")
```

---

## Screen 4 · srcNew (Create New Use Case)

### Control tree

```
srcNew
├── cmpLeftRail
├── conHeader
├── conActionBarNew              (same pattern as detail's action bar)
│   ├── btnCancel                ("← Cancel")
│   ├── lblCrumb                 ("Repository / New Use Case")
│   ├── lblUCIDNote              ("UCID will be assigned on save")
│   ├── btnSaveDraft
│   └── btnCreate                ("Create Use Case", primary)
├── conNewForm                   (centered, max-width 920)
│   ├── lblSectionTitle          ("Create New Use Case")
│   ├── conInfoBanner            (maroon-tinted banner — "Step 1 of 2...")
│   ├── conFormGrid              (2-column grid of fields)
│   └── conSubmitZoneNew
```

### conInfoBanner

Fill `gblTheme.MaroonLight`, Border `RGBA(229,201,208,1)`, Padding 10×14, BorderRadius 3.

Two labels:
- `"Step 1 of 2."` (Bold, Color `gblTheme.Maroon`)
- `"Complete the basics below to register the use case. After saving, you'll be able to fill in Contacts, Value, Funds, Governance, and other sections from the detail page."`

### conFormGrid

Fields (2-column where not full):

| Field | Type | Items | Variable to Patch into newUC |
|-------|------|-------|------------------------------|
| Use Case Name (req, full) | TextInput | — | `Name` |
| Problem Statement (req, full) | TextInput (multi) | — | `ProblemStatement` |
| AI Solution Description (full) | TextInput (multi) | — | `AISolution` |
| Type of Use Case (req) | Dropdown | `colUseCaseType` | `Type` |
| Initial Status (req) | Dropdown | `["Rationale"; "DataPrep"; "Development"]` | `Status` |
| SBU (req) | Dropdown | `colSBU` | `SBU` |
| LOB / Sub-LOB | Dropdown | (filtered by SBU) | `LOB` |
| AI Solution Owner (req) | Person picker (or TextInput v1) | — | `Owner` |
| Fiscal Year | Dropdown | `["F26"; "F27"]` | `FY` |
| Target Completion Date | DatePicker | — | `TargetDate` |
| Refresh Frequency | Dropdown | `colRefreshFreq` | `RefreshFreq` |

Keep all values in a `newUC` record:
```powerfx
// In Screen.OnVisible
Set(newUC, Defaults(colUseCases))
```

And each OnChange:
```powerfx
Set(newUC, Patch(newUC, { /* field */: Self.Text or Self.Selected.Value }))
```

### btnCreate OnSelect

```powerfx
// Generate next UCID
Set(nextUCID,
    "UC-" &
    Text(
        Max(
            AddColumns(colUseCases, NumPart, Value(Mid(UCID, 4, 4)))
            , NumPart
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

### btnCancel and btnSaveDraft

- `btnCancel.OnSelect`: `Navigate(srcList, ScreenTransition.None)`
- `btnSaveDraft.OnSelect`: `Notify("Draft saved (in memory).", NotificationType.Information)` — wire to actual Dataverse `Patch` once schema exists.

---

## Final polish checklist

- [ ] All screens have the maroon header
- [ ] cmpLeftRail collapses/expands smoothly (will snap — that's expected in Canvas)
- [ ] Every button has Maroon/MaroonDeep/MaroonLight Hover states applied
- [ ] Tab order goes top-to-bottom on each section (set `TabIndex` on form inputs starting from 1)
- [ ] `srcHome` is the topmost screen in the Tree view (prevents startup formula errors on `selectedUC` and `gblTheme`)
- [ ] `selectedUC` is initialized as a typed record in `App.OnStart` (not `Blank()`) so field access resolves at startup
- [ ] `selectedUC` is never blank on Detail at runtime (Screen.OnVisible Navigate guard handles this)
- [ ] App.OnStart runs through to `Navigate(srcHome)` without errors (check the App checker pane)

---

## What this guide doesn't cover (intentional)

- **Person picker UX with Office365Users**: covered briefly under Contacts; full pattern is in `03-formulas-reference.md`.
- **Actual Excel export / Teams notification**: those are Power Automate flows triggered by the Submit button. Spec is also in the formulas file.
- **Power BI tile embed**: drop a Power BI tile control on a future "Tracker" screen once you have the workspace + dataset.
- **Dataverse migration**: when you're ready, run a search-and-replace from `colUseCases` → your Dataverse table name. All formulas are written so this is mostly mechanical.
