# BOA Canvas App — Use Case List Screen (srcList) Step-by-Step

Use this guide to build the `srcList` (Use Case List) screen end to end.
Each part is a small block of Insert + Property steps with a sanity check
at the end. When you finish Part 7 you have a fully filterable list that
opens to the detail screen.

This guide assumes you have already completed:

- `01-app-setup.md` — `App.OnStart` has set `gblTheme`, `colUseCases`,
  `filterSearch`, `filterStatus`, `filterSBU`, `filterFY`,
  `filterOwner`. `App.Formulas` has the `filteredUseCases` named
  formula (including the Owner clause). The `cmpStatusPill` component
  is _not_ required by this screen — Step 28 inlines the pill as a
  Circle + Label because Power Apps blocks custom components inside
  containers nested in a gallery template.
- `06-left-rail-buttons-guide.md` — `conLeftRail` exists on `srcHome`
  (built as Buttons + Icons, not as a gallery).
- The existing `02-build-guide.md` for `srcHome`'s `conHeader`
  (X: `If(sideCollapsed, 64, 220)`, Y: `0`, Height: `52`, Fill:
  `gblTheme.Maroon`).
- `srcNew` and `srcDetail` exist as named screens. Both can be blank
  stubs for now — you only need their names so `Navigate(...)` doesn't
  error.

---

## Part 1 — Create the screen and base layout

### Step 1 — Add the screen

Top toolbar → **+ New screen** → **Blank**. Rename it `srcList` in the
tree. Make sure it sits below `srcHome` in the tree (drag if needed).

### Step 2 — Copy the rail and header from srcHome

In the tree on `srcHome`:

1. Click `conLeftRail`, **Ctrl+C**.
2. Click `srcList` in the tree, **Ctrl+V**. The rail appears in the same
   X/Y because its formulas reference `Parent` and `sideCollapsed`.
3. Repeat for `conHeader`: Ctrl+C on `srcHome` → click `srcList` →
   Ctrl+V.

### Step 3 — Create the page container

On `srcList`, Insert → **Container** (the classic Container, not a
horizontal/vertical responsive container).

| Property | Value |
|----------|-------|
| Name | `conPage` |
| X | `If(sideCollapsed, 64, 220)` |
| Y | `52` |
| Width | `Parent.Width - If(sideCollapsed, 64, 220)` |
| Height | `Parent.Height - 52` |
| Fill | `gblTheme.Bg` |
| BorderThickness | `0` |

This is the canvas for the three blocks: title row, filter card, gallery
card. Every other control on this screen lives **inside** `conPage`.

### Step 4 — Sanity check

Press **F5**. You should see:

- [ ] Left rail visible. The "View/Edit Use Cases" row tints maroon
      (because the active screen name `srcList` matches the formula
      from guide 06 Step 5).
- [ ] Maroon header strip across the top, to the right of the rail.
- [ ] Below the header, the area is the light gray `gblTheme.Bg`
      (#F5F5F5).

If the page is white, double-check `conPage.Fill = gblTheme.Bg`. If the
rail or header are missing, redo Step 2 on the `srcList` tree.

---

## Part 2 — Title row

A horizontal block with the page title, a count chip, a subtitle, and
the right-aligned Export + New buttons.

### Step 5 — Title row container

Inside `conPage`, Insert → **Container**.

| Property | Value |
|----------|-------|
| Name | `conTitleRow` |
| X | `24` |
| Y | `24` |
| Width | `Parent.Width - 48` |
| Height | `80` |
| Fill | `RGBA(0,0,0,0)` |
| BorderThickness | `0` |

### Step 6 — Page title

Inside `conTitleRow`, Insert → **Label**.

| Property | Value |
|----------|-------|
| Name | `lblPageTitle` |
| Text | `"View/Edit Use Cases"` |
| X | `0` |
| Y | `0` |
| Width | `380` |
| Height | `30` |
| Size | `22` |
| FontWeight | `FontWeight.Semibold` |
| Color | `gblTheme.Ink` |
| Font | `gblTheme.FontFamily` |

### Step 7 — Count chip

Inside `conTitleRow`, Insert → **Label**.

| Property | Value |
|----------|-------|
| Name | `lblPageCount` |
| Text | `Text(CountRows(colUseCases)) & " use cases"` |
| X | `lblPageTitle.X + lblPageTitle.Width + 8` |
| Y | `8` |
| Width | `140` |
| Height | `20` |
| Size | `14` |
| Color | `gblTheme.Ink3` |
| Font | `gblTheme.FontFamily` |

### Step 8 — Subtitle

Inside `conTitleRow`, Insert → **Label**.

| Property | Value |
|----------|-------|
| Name | `lblPageSub` |
| Text | `"All AI and analytics use cases across CIBC. Click a row to view or edit."` |
| X | `0` |
| Y | `lblPageTitle.Y + lblPageTitle.Height + 4` |
| Width | `700` |
| Height | `18` |
| Size | `13` |
| Color | `gblTheme.Ink3` |
| Font | `gblTheme.FontFamily` |

### Step 9 — Export button (secondary style)

Inside `conTitleRow`, Insert → **Button**.

| Property | Value |
|----------|-------|
| Name | `btnExport` |
| Text | `"Export to Excel"` |
| X | `Parent.Width - 280` |
| Y | `(Parent.Height - 36) / 2` |
| Width | `140` |
| Height | `36` |
| Size | `13` |
| FontWeight | `FontWeight.Semibold` |
| Fill | `gblTheme.Surface` |
| HoverFill | `RGBA(245, 230, 233, 1)` |
| PressedFill | `RGBA(235, 220, 225, 1)` |
| Color | `gblTheme.Maroon` |
| HoverColor | `gblTheme.Maroon` |
| BorderColor | `gblTheme.Maroon` |
| BorderThickness | `1` |
| RadiusTopLeft / TopRight / BottomLeft / BottomRight | `4` |
| OnSelect | `Notify("Export queued. You'll get a Teams ping when the file is ready.", NotificationType.Success)` |

### Step 10 — "+ New Use Case" button (primary style)

Inside `conTitleRow`, Insert → **Button**.

| Property | Value |
|----------|-------|
| Name | `btnPageNew` |
| Text | `"+ New Use Case"` |
| X | `btnExport.X + btnExport.Width + 12` |
| Y | `btnExport.Y` |
| Width | `128` |
| Height | `36` |
| Size | `13` |
| FontWeight | `FontWeight.Semibold` |
| Fill | `gblTheme.Maroon` |
| HoverFill | `gblTheme.MaroonDeep` |
| PressedFill | `gblTheme.MaroonDeep` |
| Color | `White` |
| HoverColor | `White` |
| BorderThickness | `0` |
| RadiusTopLeft / TopRight / BottomLeft / BottomRight | `4` |
| OnSelect | `Navigate(srcNew, ScreenTransition.None)` |

### Step 11 — Sanity check

Preview. You should see:

- [ ] Big "View/Edit Use Cases" title, "10 use cases" count chip to its
      right.
- [ ] Subtitle below in lighter gray.
- [ ] Two buttons on the right: Export (white, maroon border) and
      "+ New Use Case" (solid maroon).
- [ ] Hover on Export tints faintly pink; hover on New darkens to
      `gblTheme.MaroonDeep`.
- [ ] Clicking "+ New Use Case" navigates to `srcNew` (use the rail to
      return).

---

## Part 3 — Filter card

A white card containing search + four dropdowns + a Reset button, laid
out in six columns. All controls in this part are inside the **same
container** (`conFilterCard`), so X values are local to that container.

### Step 12 — Filter card container

Inside `conPage`, Insert → **Container**.

| Property | Value |
|----------|-------|
| Name | `conFilterCard` |
| X | `24` |
| Y | `conTitleRow.Y + conTitleRow.Height + 16` |
| Width | `Parent.Width - 48` |
| Height | `92` |
| Fill | `gblTheme.Surface` |
| BorderThickness | `1` |
| BorderColor | `gblTheme.Border` |
| RadiusTopLeft / TopRight / BottomLeft / BottomRight | `4` |

### Step 13 — Column layout (responsive)

We need the filter row to reflow when the rail toggles, because
`conFilterCard.Width` changes from ~1254 (rail collapsed) to ~1098
(rail expanded). Hard-coding X values would push the last column off
the right edge in the expanded state.

The pattern: every input gets the **same** Width formula, and every
input after the first sets its X by referencing the previous control.
Caption labels mirror their input below.

**Column width formula** (used by every input and every caption):

```powerfx
(Parent.Width - 14 * 7) / 6
```

That's 6 columns sharing the card width, minus 7 gutters of 14 px (one
on each edge + five between columns).

**Column X chain:**

| Column | Contents | X formula |
|--------|----------|-----------|
| 1 | Search | `14` |
| 2 | Status | `txtSearch.X + txtSearch.Width + 14` |
| 3 | SBU    | `ddStatus.X + ddStatus.Width + 14` |
| 4 | FY     | `ddSBU.X + ddSBU.Width + 14` |
| 5 | Owner  | `ddFY.X + ddFY.Width + 14` |
| 6 | Reset  | `ddOwner.X + ddOwner.Width + 14` |

All caption labels: Y=`10`, Height=`14`. All inputs and the Reset
button: Y=`30`, Height=`40`. Build the controls in the order below
(Step 14 → 19) so each X reference resolves to a control that already
exists.

### Step 14 — Search input + caption

Inside `conFilterCard`, Insert → **Label** (caption first):

| Property | Value |
|----------|-------|
| Name | `lblCapSearch` |
| Text | `"Search"` |
| X | `14` |
| Y | `10` |
| Width | `(Parent.Width - 14 * 7) / 6` |
| Height | `14` |
| Size | `11` |
| FontWeight | `FontWeight.Semibold` |
| Color | `gblTheme.Ink3` |

Then Insert → **Text input**:

| Property | Value |
|----------|-------|
| Name | `txtSearch` |
| HintText | `"UCID, name, owner…"` |
| X | `14` |
| Y | `30` |
| Width | `(Parent.Width - 14 * 7) / 6` |
| Height | `40` |
| Size | `13` |
| BorderColor | `gblTheme.Border` |
| BorderThickness | `1` |
| Color | `gblTheme.Ink` |
| OnChange | `Set(filterSearch, Self.Text)` |

### Step 15 — Status dropdown + caption

Caption `lblCapStatus`: same style as `lblCapSearch`, but
X=`txtSearch.X + txtSearch.Width + 14`, Y=`10`,
Width=`txtSearch.Width`, Text=`"Status"`.

Dropdown:

| Property | Value |
|----------|-------|
| Name | `ddStatus` |
| X | `txtSearch.X + txtSearch.Width + 14` |
| Y | `30` |
| Width | `txtSearch.Width` |
| Height | `40` |
| Items | `["All Statuses","Rationale","Data Prep","Development","Testing","Deployment","Monitoring","Decommissioning"]` |
| Default | `filterStatus` |
| OnChange | `Set(filterStatus, Self.Selected.Value)` |
| BorderColor | `gblTheme.Border` |
| BorderThickness | `1` |
| Color | `gblTheme.Ink` |

### Step 16 — SBU dropdown + caption

Caption `lblCapSBU`: X=`ddStatus.X`, Y=`10`, Width=`txtSearch.Width`,
Text=`"SBU"`.

Dropdown:

| Property | Value |
|----------|-------|
| Name | `ddSBU` |
| X | `ddStatus.X + ddStatus.Width + 14` |
| Y | `30` |
| Width | `txtSearch.Width` |
| Height | `40` |
| Items | `["All SBUs","PBB","Capital Markets","Wealth","Commercial","Direct Banking"]` |
| Default | `filterSBU` |
| OnChange | `Set(filterSBU, Self.Selected.Value)` |
| BorderColor | `gblTheme.Border` |
| BorderThickness | `1` |

### Step 17 — FY dropdown + caption

Caption `lblCapFY`: X=`ddSBU.X`, Y=`10`, Width=`txtSearch.Width`,
Text=`"Fiscal year"`.

Dropdown:

| Property | Value |
|----------|-------|
| Name | `ddFY` |
| X | `ddSBU.X + ddSBU.Width + 14` |
| Y | `30` |
| Width | `txtSearch.Width` |
| Height | `40` |
| Items | `["F26","F25","F24"]` |
| Default | `filterFY` |
| OnChange | `Set(filterFY, Self.Selected.Value)` |
| BorderColor | `gblTheme.Border` |
| BorderThickness | `1` |

### Step 18 — Owner dropdown + caption

Caption `lblCapOwner`: X=`ddFY.X`, Y=`10`, Width=`txtSearch.Width`,
Text=`"Owner"`.

Dropdown:

| Property | Value |
|----------|-------|
| Name | `ddOwner` |
| X | `ddFY.X + ddFY.Width + 14` |
| Y | `30` |
| Width | `txtSearch.Width` |
| Height | `40` |
| Items | `Distinct(colUseCases, Owner)` |
| Default | `filterOwner` |
| AllowEmptySelection | `true` |
| OnChange | `Set(filterOwner, If(IsBlank(Self.Selected), "", Self.Selected.Value))` |
| BorderColor | `gblTheme.Border` |
| BorderThickness | `1` |

> Owner is wired into `filteredUseCases` via the `(filterOwner = "" ||
> Owner = filterOwner)` clause in `App.Formulas` (see `01-app-setup.md`
> section 3). `filterOwner` is initialized to `""` in App.OnStart, so
> the dropdown starts unselected and shows every row. Deselecting the
> dropdown (or hitting Reset) clears `filterOwner` back to `""`.

### Step 19 — Reset button

Inside `conFilterCard`, Insert → **Button**.

| Property | Value |
|----------|-------|
| Name | `btnReset` |
| Text | `"Reset"` |
| X | `ddOwner.X + ddOwner.Width + 14` |
| Y | `30` |
| Width | `txtSearch.Width` |
| Height | `40` |
| Size | `13` |
| FontWeight | `FontWeight.Semibold` |
| Fill | `gblTheme.Surface` |
| HoverFill | `RGBA(245, 230, 233, 1)` |
| Color | `gblTheme.Maroon` |
| HoverColor | `gblTheme.Maroon` |
| BorderColor | `gblTheme.Maroon` |
| BorderThickness | `1` |
| RadiusTopLeft / TopRight / BottomLeft / BottomRight | `4` |
| OnSelect | `Set(filterSearch, ""); Reset(txtSearch); Set(filterStatus, "All Statuses"); Set(filterSBU, "All SBUs"); Set(filterFY, "F26"); Set(filterOwner, ""); Reset(ddOwner)` |

### Step 20 — Sanity check

Preview:

- [ ] Six columns visible: search box, four dropdowns, Reset button.
- [ ] Caption labels sit above each, small and gray.
- [ ] **Toggle the rail (hamburger).** Columns should reflow — each
      one narrows when the rail expands, widens when it collapses.
      No column should clip past the right edge of the white card.
- [ ] Type in search and select Status / SBU / FY — no visible effect
      on a gallery yet (you build it in Part 5).
- [ ] Click Reset — search clears, dropdowns return to defaults.

> **If you already built Part 3 with the old hard-coded X/Width values
> (14, 228, 442, …, all Width=200):** select each control in the tree
> and update only its X and Width properties to the formulas above.
> Other properties stay as you set them. The Search box stays at
> X=`14`; the rest chain off the previous control.

If anything looks misaligned, double-check that each X matches the
column reference in Step 13.

---

## Part 4 — Gallery card and header row

### Step 21 — Gallery card container

Inside `conPage`, Insert → **Container**.

| Property | Value |
|----------|-------|
| Name | `conGallery` |
| X | `24` |
| Y | `conFilterCard.Y + conFilterCard.Height + 16` |
| Width | `Parent.Width - 48` |
| Height | `Parent.Height - Self.Y - 24` |
| Fill | `gblTheme.Surface` |
| BorderThickness | `1` |
| BorderColor | `gblTheme.Border` |
| RadiusTopLeft / TopRight / BottomLeft / BottomRight | `4` |

### Step 22 — Header strip

Inside `conGallery`, Insert → **Layout** → **Horizontal container**.

| Property | Value |
|----------|-------|
| Name | `conGalleryHeader` |
| X | `0` |
| Y | `0` |
| Width | `Parent.Width` |
| Height | `36` |
| Fill | `gblTheme.Maroon` |
| BorderThickness | `0` |
| LayoutDirection | `LayoutDirection.Horizontal` |
| LayoutGap | `0` |
| LayoutAlignItems | `LayoutAlignItems.Center` |
| LayoutJustifyContent | `LayoutJustifyContent.Start` |
| PaddingLeft | `14` |
| PaddingRight | `0` |
| PaddingTop | `0` |
| PaddingBottom | `0` |

The container hosts the 9 column labels in Step 23. `LayoutGap: 0`
means no automatic spacing between columns — each label's allocated
width comes purely from its `FillPortions`. `PaddingLeft: 14`
reproduces the 14-px leading gutter without needing a spacer.

### Step 23 — Column labels (FillPortions)

All nine header labels live **inside** `conGalleryHeader`. Because the
parent is a Horizontal Container, you don't set X, Y, Width, or
Height on the labels — the container distributes them left-to-right
and centers them vertically. The only layout property you set is
`FillPortions`: the column's share of the row width.

Power Apps divides the available container width proportionally
across all FillPortions values, so toggling the rail rescales every
column with no extra math. The integer is the column's pixel budget
at the design size (rail collapsed) — use the same numbers we did
before.

**Insert the labels in order 1 → 9.** Tree order = left-to-right
rendering. If you insert them in the wrong order, drag in the tree
to reorder.

| # | Name | Text | FillPortions |
|---|------|------|--------------|
| 1 | `lblColUCID` | `"UCID"` | `100` |
| 2 | `lblColName` | `"Use Case Name"` | `320` |
| 3 | `lblColSBU` | `"SBU"` | `120` |
| 4 | `lblColOwner` | `"AI Solution Owner"` | `160` |
| 5 | `lblColStatus` | `"Status"` | `150` |
| 6 | `lblColFY` | `"FY"` | `60` |
| 7 | `lblColValue` | `"Realized Value"` | `130` |
| 8 | `lblColUpdated` | `"Last Updated"` | `110` |
| 9 | `lblColAction` | `""` | `80` |

All nine share these properties:

| Property | Value |
|----------|-------|
| Size | `12` |
| FontWeight | `FontWeight.Semibold` |
| Color | `White` |
| Font | `gblTheme.FontFamily` |
| PaddingLeft | `0` |

> **Adding or removing a column later:** just insert a new label
> inside `conGalleryHeader` (or delete an existing one) and set its
> `FillPortions`. The other columns automatically rebalance — no
> formula elsewhere to update. The matching insert/delete must be
> done in the row template (Step 27) so the rows still line up under
> the headers.

### Step 24 — Sanity check

Preview:

- [ ] White gallery card below the filter card, with a maroon top
      strip.
- [ ] Column header labels visible in white, left-aligned within
      their column widths.
- [ ] **Toggle the rail (hamburger).** Header columns should reflow
      — every column narrows proportionally when the rail expands,
      widens back when it collapses. No clipping on the right.

---

## Part 5 — Gallery rows

### Step 25 — Insert the gallery

Inside `conGallery`, Insert → **Gallery** → **Blank vertical**.

| Property | Value |
|----------|-------|
| Name | `galUseCases` |
| X | `0` |
| Y | `36` |
| Width | `Parent.Width` |
| Height | `Parent.Height - 36 - 40` |
| Items | `filteredUseCases` |
| TemplateSize | `52` |
| TemplatePadding | `0` |
| ShowScrollbar | `true` |
| BorderThickness | `0` |

**OnSelect** (on the gallery itself — NOT inside the template):

```powerfx
Set(selectedUC, ThisItem);
Set(currentSection, "Info");
Navigate(srcDetail, ScreenTransition.None)
```

The 40 reserved at the bottom of Height is for `conGalleryFooter` in
Part 6. The row template (built in Steps 26–29) will host its own
Horizontal Container that mirrors `conGalleryHeader`.

### Step 26 — Row template structure

In the tree, click the small chevron next to `galUseCases` to enter
the template (the template highlights with a dashed border). Inside
the template, you'll add two siblings:

1. **`conRow`** — Horizontal Container holding the 9 row columns
   (mirrors `conGalleryHeader`).
2. **`recRowDivider`** — 1px rectangle at the bottom of the row.

Insert → **Layout** → **Horizontal container**:

| Property | Value |
|----------|-------|
| Name | `conRow` |
| X | `0` |
| Y | `0` |
| Width | `Parent.TemplateWidth` |
| Height | `Parent.TemplateHeight - 1` |
| Fill | `RGBA(0,0,0,0)` |
| BorderThickness | `0` |
| LayoutDirection | `LayoutDirection.Horizontal` |
| LayoutGap | `0` |
| LayoutAlignItems | `LayoutAlignItems.Center` |
| LayoutJustifyContent | `LayoutJustifyContent.Start` |
| PaddingLeft | `14` |
| PaddingRight | `0` |
| PaddingTop | `0` |
| PaddingBottom | `0` |

The `- 1` on Height leaves room for the divider below. `Parent` here
is the gallery (`galUseCases`); `TemplateWidth` and `TemplateHeight`
are the gallery's row-size properties.

Then, still inside the template (NOT inside `conRow`), Insert →
**Rectangle** for the divider:

| Property | Value |
|----------|-------|
| Name | `recRowDivider` |
| X | `0` |
| Y | `Parent.TemplateHeight - 1` |
| Width | `Parent.TemplateWidth` |
| Height | `1` |
| Fill | `gblTheme.Border` |
| BorderThickness | `0` |

### Step 27 — Row columns

All 9 row controls live **inside `conRow`**. Like the header, you set
only `FillPortions` to control column width — no X, Y, Width, or
Height. **The FillPortions value for each column must match the
header in Step 23** so headers and rows line up.

Insert in order 1 → 9. Items 5 and 9 are plain Containers (not
labels) — they're empty wrappers that get filled in Steps 28 and 29.

| # | Control type | Name | Text | FillPortions | Size | Color | FontWeight |
|---|--------------|------|------|--------------|------|-------|-----------|
| 1 | Label | `lblUCID` | `ThisItem.UCID` | `100` | 12 | `gblTheme.Ink3` | `FontWeight.Normal` |
| 2 | Label | `lblName` | `ThisItem.Name` | `320` | 13 | `gblTheme.Ink` | `FontWeight.Semibold` |
| 3 | Label | `lblSBU` | `ThisItem.SBU` | `120` | 13 | `gblTheme.Ink2` | `FontWeight.Normal` |
| 4 | Label | `lblOwner` | `ThisItem.Owner` | `160` | 13 | `gblTheme.Ink2` | `FontWeight.Normal` |
| 5 | Container (classic) | `conStatusCol` | — | `150` | — | — | — |
| 6 | Label | `lblFY` | `ThisItem.FY` | `60` | 13 | `gblTheme.Ink2` | `FontWeight.Normal` |
| 7 | Label | `lblValue` | (see below) | `130` | 13 | `gblTheme.Ink` | `FontWeight.Normal` |
| 8 | Label | `lblUpdated` | (see below) | `110` | 12 | `gblTheme.Ink2` | `FontWeight.Normal` |
| 9 | Container (classic) | `conActionCol` | — | `80` | — | — | — |

All 7 labels share Font=`gblTheme.FontFamily` and PaddingLeft=`0`.

The two wrapper Containers (`conStatusCol` and `conActionCol`) need
just two properties: `FillPortions` (from the table) and
`Fill: RGBA(0,0,0,0)`. They exist so the pill and View button can
sit within a column slot without stretching to its full width — see
Steps 28 and 29.

`lblValue.Text`:

```powerfx
If(
    ThisItem.RealizedValue = 0,
    "—",
    "$" & Text(ThisItem.RealizedValue / 1000000, "0.0") & "M"
)
```

`lblUpdated.Text`:

```powerfx
With({d: DateDiff(ThisItem.LastUpdated, Today())},
    Switch(true,
        d < 7,  Text(d) & " days ago",
        d < 30, Text(RoundDown(d/7, 0)) & " wks ago",
                Text(RoundDown(d/30, 0)) & " mo ago"
    )
)
```

### Step 28 — Status pill (inlined inside `conStatusCol`)

> **Why not the `cmpStatusPill` component?** Power Apps explicitly
> blocks custom components from being placed as children of a
> Container that itself lives inside a gallery template. You'll see an
> error like *"does not support component as a child of container
> under gallery"* if you try. So for this screen we inline the pill
> as two primitives — a Circle for the colored dot and a Label for
> the status text. The `cmpStatusPill` component is still useful
> elsewhere (e.g. on the detail screen side rail, where it isn't
> inside a gallery).

Inside `conStatusCol`, insert two controls.

**Circle (the colored dot)** — Insert → **Icons** → **Circle**:

| Property | Value |
|----------|-------|
| Name | `circStatusDot` |
| X | `0` |
| Y | `(Parent.Height - Self.Height) / 2` |
| Width | `8` |
| Height | `8` |
| BorderThickness | `0` |
| Fill | (see formula below) |

`circStatusDot.Fill`:

```powerfx
Switch(ThisItem.Status,
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

(Same color map as the `cmpStatusPill` component, just inlined.)

**Label (the status text)** — Insert → **Label**:

| Property | Value |
|----------|-------|
| Name | `lblStatusText` |
| X | `circStatusDot.X + circStatusDot.Width + 6` |
| Y | `0` |
| Width | `Parent.Width - Self.X` |
| Height | `Parent.Height` |
| Text | (see formula below) |
| Color | `gblTheme.Ink2` |
| Size | `12` |
| FontWeight | `FontWeight.Normal` |
| Font | `gblTheme.FontFamily` |
| VerticalAlign | `VerticalAlign.Middle` |
| PaddingLeft | `0` |

`lblStatusText.Text`:

```powerfx
Switch(ThisItem.Status,
    "DataPrep", "Data Prep",
    ThisItem.Status
)
```

(Only `DataPrep` needs the space inserted; every other status code
is human-readable as-is.)

### Step 29 — View button (inside `conActionCol`)

Click `conActionCol` in the tree, then Insert → **Button**.

| Property | Value |
|----------|-------|
| Name | `btnView` |
| Text | `"View"` |
| X | `0` |
| Y | `(Parent.Height - Self.Height) / 2` |
| Width | `Parent.Width * 0.8` |
| Height | `28` |
| Size | `12` |
| FontWeight | `FontWeight.Semibold` |
| Fill | `gblTheme.Surface` |
| HoverFill | `RGBA(245, 230, 233, 1)` |
| Color | `gblTheme.Maroon` |
| BorderColor | `gblTheme.Maroon` |
| BorderThickness | `1` |
| RadiusTopLeft / TopRight / BottomLeft / BottomRight | `4` |
| OnSelect | `Select(Parent.Parent.Parent)` |

`Parent.Parent.Parent` walks: button → `conActionCol` → `conRow` →
`galUseCases`. Calling `Select()` on the gallery fires the row's
OnSelect (the `Navigate` from Step 25), so clicking View has the
same effect as clicking anywhere else on the row.

### Step 30 — Sanity check

Click anywhere outside the template to exit template-editing mode.
Preview:

- [ ] 10 rows visible (matching `colUseCases`).
- [ ] Each row shows UCID, name in bold, SBU, owner, a colored
      Status pill, FY, `$X.XM` value (or `"—"`), `"N days ago"`, and
      a View button.
- [ ] Clicking a row (anywhere — label, pill, or button) opens
      `srcDetail` with `selectedUC` populated.
- [ ] Typing `"Mortgage"` in Search filters to 1 row.
- [ ] Setting Status to `"Development"` filters to 2 rows.
- [ ] Reset → all 10 rows return.
- [ ] **Toggle the rail (hamburger).** Every column should reflow —
      narrower when the rail expands, wider when it collapses. Row
      cells stay aligned under their header labels in both states.

If a row column drifts away from its header, the FillPortions value
on a row control doesn't match the corresponding header value.
Compare Step 23 and Step 27 — column N's FillPortions must be the
same integer in both tables.

> **Adding a new column later:** insert a label in `conGalleryHeader`
> (Step 23 table), then insert a matching control in `conRow` (Step
> 27 table) with the same FillPortions. The other 9 columns rebalance
> automatically.
>
> **If you already built Parts 4/5 with the previous chained-formula
> approach (X = `lblColUCID.X + …`, Width = `… * Parent.Width / 1244`):**
> this is a larger migration than usual. The cleanest path is to
> delete `recGalleryHeader` and the gallery template contents, then
> rebuild from Step 22 with the Horizontal Container approach. Plan
> for ~15 minutes.

---

## Part 6 — Gallery footer

A small status bar showing visible / total counts, with a maroon
scroll-status dot on the right.

### Step 31 — Footer container

> **Why a Container and not a Rectangle?** A Rectangle is a primitive
> shape — Power Apps won't let you drop anything inside it. We need
> the footer to host four children (top-border rectangle, count label,
> scroll dot, scroll-status label), so it must be a Container. The
> `con` prefix in the name reflects that.

Inside `conGallery` (not inside the gallery template — back out first),
Insert → **Container** (the classic Container, not horizontal/vertical).

| Property | Value |
|----------|-------|
| Name | `conGalleryFooter` |
| X | `0` |
| Y | `Parent.Height - 40` |
| Width | `Parent.Width` |
| Height | `40` |
| Fill | `RGBA(250, 250, 250, 1)` |
| BorderThickness | `0` |

Add a 1px top border. Inside `conGalleryFooter`, Insert → **Rectangle**:

| Property | Value |
|----------|-------|
| Name | `recFooterTop` |
| X | `0` |
| Y | `0` |
| Width | `Parent.Width` |
| Height | `1` |
| Fill | `gblTheme.Border` |
| BorderThickness | `0` |

(The Rectangle works as a child here because its **parent** is the
Container `conGalleryFooter` — not another Rectangle.)

### Step 32 — Count label (left)

Inside `conGalleryFooter`, Insert → **Label**.

| Property | Value |
|----------|-------|
| Name | `lblFooterCount` |
| Text | `"Showing " & Text(CountRows(filteredUseCases)) & " of " & Text(CountRows(colUseCases))` |
| X | `14` |
| Y | `(Parent.Height - Self.Height) / 2` |
| Width | `300` |
| Height | `20` |
| Size | `12` |
| Color | `gblTheme.Ink3` |

### Step 33 — Scroll-status dot

Inside `conGalleryFooter`, Insert → **Icons** → **Circle** (if you can't
find Circle, insert a Rectangle and set RadiusTop/Bottom Left/Right to
`999`).

| Property | Value |
|----------|-------|
| Name | `icnScrollDot` |
| X | `Parent.Width - 220` |
| Y | `(Parent.Height - Self.Height) / 2` |
| Width | `7` |
| Height | `7` |
| Fill | `gblTheme.Maroon` |
| BorderThickness | `0` |

### Step 34 — Scroll-status label

Inside `conGalleryFooter`, Insert → **Label**.

| Property | Value |
|----------|-------|
| Name | `lblScrollStatus` |
| Text | `If(CountRows(filteredUseCases) >= CountRows(colUseCases), "All use cases loaded", "Scroll to load more")` |
| X | `icnScrollDot.X + icnScrollDot.Width + 8` |
| Y | `(Parent.Height - Self.Height) / 2` |
| Width | `200` |
| Height | `20` |
| Size | `12` |
| FontWeight | `FontWeight.Semibold` |
| Color | `gblTheme.Ink2` |

### Step 35 — Sanity check

Preview:

- [ ] Footer strip pinned at the bottom of the gallery card with a 1px
      top border.
- [ ] Left: `"Showing 10 of 10"`.
- [ ] Right: maroon dot + `"All use cases loaded"` in semibold.
- [ ] Apply any filter; the count updates and the status flips to
      `"Scroll to load more"` if the filtered count is less than total.

---

## Part 7 — Full-screen sanity check

Press **F5** on `srcList` and walk through:

- [ ] Header (top), rail (left), and content area visible.
- [ ] Title row: `"View/Edit Use Cases"` + `"10 use cases"` + subtitle.
      Export button (white / maroon border) and "+ New Use Case" button
      (solid maroon) on the right.
- [ ] Filter card: six columns of search + dropdowns + Reset.
- [ ] Gallery card: maroon header strip, 10 rows underneath, footer at
      bottom.
- [ ] Type `"Mortgage"` in Search → 1 row.
- [ ] Set Status to `"Monitoring"` → 2 rows.
- [ ] Click Reset → 10 rows again.
- [ ] Click any row → navigates to `srcDetail` with `selectedUC`
      populated.
- [ ] Click the rail's hamburger → rail collapses to 64px; content
      expands; gallery still works (last column may now extend past
      the visible area if you kept the wide widths — see Step 23
      note).

If all 9 boxes pass, `srcList` is done. Move on to `srcDetail` (Screen 3
in `02-build-guide.md`).

---

## Quick reference — things that go wrong

| Symptom | Cause | Fix |
|---------|-------|-----|
| Gallery shows no rows | `filteredUseCases` is empty, or filters too restrictive | Click `App` in the tree, check `Formulas` contains the `filteredUseCases =` block from `01-app-setup.md` section 3. Reset filters. |
| Gallery shows red squiggle on Items | `filteredUseCases` isn't defined as a named formula | Same as above — paste the block into `App.Formulas` (not `App.OnStart`). |
| Columns don't reflow when rail toggles | A child of `conGalleryHeader` or `conRow` has an explicit `Width` (or `X`) instead of `FillPortions` | Open the control and clear `Width` (it should be auto, with `FillPortions` driving sizing). Same for `X` — Horizontal Container children shouldn't have X set. |
| Header columns don't line up with row columns | A row column's `FillPortions` doesn't match its header column | Compare Step 23 and Step 27 — column N's FillPortions must be the same integer in both tables (e.g. SBU is `120` in both). |
| Row columns appear in the wrong order | Tree order inside `conRow` doesn't match column 1→9 | In the tree, drag children of `conRow` so they appear top-to-bottom in the desired left-to-right order. Same for `conGalleryHeader`. |
| Status text overflows the Status column | `lblStatusText.Width` is fixed instead of `Parent.Width - Self.X` | Width should reference Parent so it scales with `conStatusCol` when the rail toggles. |
| View button stretches the whole Action column | Button was inserted directly inside `conRow` instead of inside `conActionCol` | Cut the button from `conRow`, paste it inside `conActionCol`. |
| View column overflows past the gallery card | `btnView.Width = Parent.Width` (the wrapper container) | Change to `Parent.Width * 0.8` (Step 29). Same `* 0.8` pattern for the pill in Step 28. |
| Custom component (`cmpStatusPill`) won't insert into `conStatusCol` — Studio errors with "does not support component as a child of container under gallery" | Power Apps blocks custom components inside containers nested in gallery templates | Use the inline Circle + Label pattern in Step 28 instead of the component. The component is still fine elsewhere (e.g. the detail screen rail). |
| Status dot is the wrong color (or always gray) | `ThisItem.Status` doesn't match any branch in the `circStatusDot.Fill` Switch | Status values must be one of: Rationale, DataPrep, Development, Testing, Deployment, Monitoring, Decommissioning. Anything else falls through to the gray default. |
| Status label shows "DataPrep" instead of "Data Prep" | The Switch in `lblStatusText.Text` is missing | Add `"DataPrep", "Data Prep",` as the first Switch branch (Step 28 formula). |
| Row click does nothing | OnSelect is on a template control instead of the gallery itself | Click `galUseCases` in the tree (not a child of it), put the formula from Step 25 in its `OnSelect`. |
| View button doesn't navigate | `btnView.OnSelect` is `Select(Parent)` (left over from the old non-container layout) | Update to `Select(Parent.Parent.Parent)` — button → `conActionCol` → `conRow` → `galUseCases` (Step 29). |
| Reset doesn't clear search text | `Reset(txtSearch)` missing | Add `Reset(txtSearch);` to `btnReset.OnSelect` alongside `Set(filterSearch, "");`. |
| `Navigate(srcNew, ...)` errors | `srcNew` screen doesn't exist yet | Create a blank screen named exactly `srcNew`. You'll build it out in `02-build-guide.md` section 4. |
| `Navigate(srcDetail, ...)` errors | Same as above for `srcDetail` | Create a blank screen named exactly `srcDetail`. |
| Export button doesn't show a notification | `Notify(...)` only fires in app preview, not Studio edit mode | Press F5, then click. |
| Hover on rows does nothing visible | You're testing in Studio edit mode | Press F5 — hover effects only run in preview. |
| Filter card is too short / tall | Wrong Height on `conFilterCard` | Should be exactly `92`. |
| Gallery footer overlaps the last row | Gallery Height didn't reserve 40 for the footer | Set `galUseCases.Height = Parent.Height - 36 - 40`. |
| Can't insert anything inside `conGalleryFooter` — Studio greys out / does nothing | It was inserted as a Rectangle, which can't have children | Delete it and re-insert as a classic **Container** (Step 31). Then add `recFooterTop`, `lblFooterCount`, `icnScrollDot`, `lblScrollStatus` inside. |
