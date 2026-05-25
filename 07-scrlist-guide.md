# BOA Canvas App — Use Case List Screen (scrList) Step-by-Step

Use this guide to build the `scrList` (Use Case List) screen end to end.
Each part is a small block of Insert + Property steps with a sanity check
at the end. When you finish Part 7 you have a fully filterable list that
opens to the detail screen.

This guide assumes you have already completed:

- `01-app-setup.md` — `App.OnStart` has set `theme`, `colUseCases`,
  `filterSearch`, `filterStatus`, `filterSBU`, `filterFY`, and the
  `cmpStatusPill` component exists. `App.Formulas` has the
  `filteredUseCases` named formula.
- `06-left-rail-buttons-guide.md` — `conLeftRail` exists on `scrHome`
  (built as Buttons + Icons, not as a gallery).
- The existing `02-build-guide.md` for `scrHome`'s `conHeader`
  (X: `If(sideCollapsed, 64, 220)`, Y: `0`, Height: `52`, Fill:
  `theme.Maroon`).
- `scrNew` and `scrDetail` exist as named screens. Both can be blank
  stubs for now — you only need their names so `Navigate(...)` doesn't
  error.

---

## Part 1 — Create the screen and base layout

### Step 1 — Add the screen

Top toolbar → **+ New screen** → **Blank**. Rename it `scrList` in the
tree. Make sure it sits below `scrHome` in the tree (drag if needed).

### Step 2 — Copy the rail and header from scrHome

In the tree on `scrHome`:

1. Click `conLeftRail`, **Ctrl+C**.
2. Click `scrList` in the tree, **Ctrl+V**. The rail appears in the same
   X/Y because its formulas reference `Parent` and `sideCollapsed`.
3. Repeat for `conHeader`: Ctrl+C on `scrHome` → click `scrList` →
   Ctrl+V.

### Step 3 — Create the page container

On `scrList`, Insert → **Container** (the classic Container, not a
horizontal/vertical responsive container).

| Property | Value |
|----------|-------|
| Name | `conPage` |
| X | `If(sideCollapsed, 64, 220)` |
| Y | `52` |
| Width | `Parent.Width - If(sideCollapsed, 64, 220)` |
| Height | `Parent.Height - 52` |
| Fill | `theme.Bg` |
| BorderThickness | `0` |

This is the canvas for the three blocks: title row, filter card, gallery
card. Every other control on this screen lives **inside** `conPage`.

### Step 4 — Sanity check

Press **F5**. You should see:

- [ ] Left rail visible. The "View/Edit Use Cases" row tints maroon
      (because the active screen name `scrList` matches the formula
      from guide 06 Step 5).
- [ ] Maroon header strip across the top, to the right of the rail.
- [ ] Below the header, the area is the light gray `theme.Bg`
      (#F5F5F5).

If the page is white, double-check `conPage.Fill = theme.Bg`. If the
rail or header are missing, redo Step 2 on the `scrList` tree.

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
| Color | `theme.Ink` |
| Font | `theme.FontFamily` |

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
| Color | `theme.Ink3` |
| Font | `theme.FontFamily` |

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
| Color | `theme.Ink3` |
| Font | `theme.FontFamily` |

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
| Fill | `theme.Surface` |
| HoverFill | `RGBA(245, 230, 233, 1)` |
| PressedFill | `RGBA(235, 220, 225, 1)` |
| Color | `theme.Maroon` |
| HoverColor | `theme.Maroon` |
| BorderColor | `theme.Maroon` |
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
| Fill | `theme.Maroon` |
| HoverFill | `theme.MaroonDeep` |
| PressedFill | `theme.MaroonDeep` |
| Color | `White` |
| HoverColor | `White` |
| BorderThickness | `0` |
| RadiusTopLeft / TopRight / BottomLeft / BottomRight | `4` |
| OnSelect | `Navigate(scrNew, ScreenTransition.None)` |

### Step 11 — Sanity check

Preview. You should see:

- [ ] Big "View/Edit Use Cases" title, "10 use cases" count chip to its
      right.
- [ ] Subtitle below in lighter gray.
- [ ] Two buttons on the right: Export (white, maroon border) and
      "+ New Use Case" (solid maroon).
- [ ] Hover on Export tints faintly pink; hover on New darkens to
      `theme.MaroonDeep`.
- [ ] Clicking "+ New Use Case" navigates to `scrNew` (use the rail to
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
| Fill | `theme.Surface` |
| BorderThickness | `1` |
| BorderColor | `theme.Border` |
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
| Color | `theme.Ink3` |

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
| BorderColor | `theme.Border` |
| BorderThickness | `1` |
| Color | `theme.Ink` |
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
| BorderColor | `theme.Border` |
| BorderThickness | `1` |
| Color | `theme.Ink` |

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
| BorderColor | `theme.Border` |
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
| BorderColor | `theme.Border` |
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
| AllowEmptySelection | `true` |
| BorderColor | `theme.Border` |
| BorderThickness | `1` |

> Owner isn't wired into `filteredUseCases` yet (App.Formulas in
> `01-app-setup.md` doesn't reference it). The dropdown still works
> visually. When you decide Owner should filter the list, add
> `OnChange: Set(filterOwner, Self.Selected.Value)` to `ddOwner`,
> initialize `filterOwner` in App.OnStart, and extend `filteredUseCases`
> with `(filterOwner = "" || Owner = filterOwner)`.

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
| Fill | `theme.Surface` |
| HoverFill | `RGBA(245, 230, 233, 1)` |
| Color | `theme.Maroon` |
| HoverColor | `theme.Maroon` |
| BorderColor | `theme.Maroon` |
| BorderThickness | `1` |
| RadiusTopLeft / TopRight / BottomLeft / BottomRight | `4` |
| OnSelect | `Set(filterSearch, ""); Reset(txtSearch); Set(filterStatus, "All Statuses"); Set(filterSBU, "All SBUs"); Set(filterFY, "F26"); Reset(ddOwner)` |

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
| Fill | `theme.Surface` |
| BorderThickness | `1` |
| BorderColor | `theme.Border` |
| RadiusTopLeft / TopRight / BottomLeft / BottomRight | `4` |

### Step 22 — Header strip

Inside `conGallery`, Insert → **Rectangle**.

| Property | Value |
|----------|-------|
| Name | `recGalleryHeader` |
| X | `0` |
| Y | `0` |
| Width | `Parent.Width` |
| Height | `36` |
| Fill | `theme.Maroon` |
| BorderThickness | `0` |

### Step 23 — Column labels (responsive)

All nine header labels live inside `recGalleryHeader`. To make the
gallery reflow when the rail toggles, each Width scales with the
current card width, and each X chains off the previous column.

**The scale ratio** (used by every Width below):

```powerfx
Parent.Width / 1244
```

1244 is the design width with the rail collapsed (the running sum of
14 + 100 + 320 + 120 + 160 + 150 + 60 + 130 + 110 + 80). At runtime
`Parent.Width` is whatever `recGalleryHeader.Width` resolves to (which
equals `conGallery.Width`), so all nine columns shrink proportionally
when the rail expands and grow back when it collapses.

Insert the labels in order (1 → 9), because each X references the
previous one.

| # | Name | Text | X | Width |
|---|------|------|---|-------|
| 1 | `lblColUCID` | `"UCID"` | `14 * Parent.Width / 1244` | `100 * Parent.Width / 1244` |
| 2 | `lblColName` | `"Use Case Name"` | `lblColUCID.X + lblColUCID.Width` | `320 * Parent.Width / 1244` |
| 3 | `lblColSBU` | `"SBU"` | `lblColName.X + lblColName.Width` | `120 * Parent.Width / 1244` |
| 4 | `lblColOwner` | `"AI Solution Owner"` | `lblColSBU.X + lblColSBU.Width` | `160 * Parent.Width / 1244` |
| 5 | `lblColStatus` | `"Status"` | `lblColOwner.X + lblColOwner.Width` | `150 * Parent.Width / 1244` |
| 6 | `lblColFY` | `"FY"` | `lblColStatus.X + lblColStatus.Width` | `60 * Parent.Width / 1244` |
| 7 | `lblColValue` | `"Realized Value"` | `lblColFY.X + lblColFY.Width` | `130 * Parent.Width / 1244` |
| 8 | `lblColUpdated` | `"Last Updated"` | `lblColValue.X + lblColValue.Width` | `110 * Parent.Width / 1244` |
| 9 | `lblColAction` | `""` | `lblColUpdated.X + lblColUpdated.Width` | `80 * Parent.Width / 1244` |

All nine share these properties:

| Property | Value |
|----------|-------|
| Y | `10` |
| Height | `16` |
| Size | `12` |
| FontWeight | `FontWeight.Semibold` |
| Color | `White` |
| Font | `theme.FontFamily` |

> **Why this works:** the nine bases (14, 100, 320, …, 80) sum to
> exactly 1244 px. Multiplying every X and Width by
> `Parent.Width / 1244` makes them sum to exactly `Parent.Width`, so
> the row always fills the card with no clipping in either rail state.

### Step 24 — Sanity check

Preview:

- [ ] White gallery card below the filter card, with a maroon top strip.
- [ ] Column header labels visible in white, left-aligned within their
      column widths.

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
Navigate(scrDetail, ScreenTransition.None)
```

The 40 reserved at the bottom of Height is for `conGalleryFooter` in
Part 6.

### Step 26 — Row divider

You need a 1px line at the bottom of each row.

In the tree, click the small chevron next to `galUseCases` to enter the
template (the template highlights with a dashed border). Insert →
**Rectangle**.

| Property | Value |
|----------|-------|
| Name | `recRowDivider` |
| X | `0` |
| Y | `galUseCases.TemplateHeight - 1` |
| Width | `Parent.Width` |
| Height | `1` |
| Fill | `theme.Border` |
| BorderThickness | `0` |

### Step 27 — Row labels

Still inside the template, insert these seven labels. Every label uses
Y=`(galUseCases.TemplateHeight - Self.Height) / 2` and Font=
`theme.FontFamily`. Height is 16 except where noted.

**X and Width reference the header column directly.** This guarantees
every row column lines up under its header — when the rail toggles
and the header columns reflow, the rows reflow with them. No
duplicated math.

| # | Control | Name | Text | X | Width | Size | Color | FontWeight |
|---|---------|------|------|---|-------|------|-------|-----------|
| 1 | Label | `lblUCID` | `ThisItem.UCID` | `lblColUCID.X` | `lblColUCID.Width` | 12 | `theme.Ink3` | `FontWeight.Normal` |
| 2 | Label | `lblName` | `ThisItem.Name` | `lblColName.X` | `lblColName.Width` | 13 | `theme.Ink` | `FontWeight.Semibold` |
| 3 | Label | `lblSBU` | `ThisItem.SBU` | `lblColSBU.X` | `lblColSBU.Width` | 13 | `theme.Ink2` | `FontWeight.Normal` |
| 4 | Label | `lblOwner` | `ThisItem.Owner` | `lblColOwner.X` | `lblColOwner.Width` | 13 | `theme.Ink2` | `FontWeight.Normal` |
| 5 | Label | `lblFY` | `ThisItem.FY` | `lblColFY.X` | `lblColFY.Width` | 13 | `theme.Ink2` | `FontWeight.Normal` |
| 6 | Label | `lblValue` | (see below) | `lblColValue.X` | `lblColValue.Width` | 13 | `theme.Ink` | `FontWeight.Normal` |
| 7 | Label | `lblUpdated` | (see below) | `lblColUpdated.X` | `lblColUpdated.Width` | 12 | `theme.Ink2` | `FontWeight.Normal` |

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

### Step 28 — Status pill (column 5)

Still inside the template, Insert → **Custom** → pick `cmpStatusPill`
(built in `01-app-setup.md` section 4).

| Property | Value |
|----------|-------|
| Name | `cmpStatusPill_1` |
| X | `lblColStatus.X` |
| Y | `(galUseCases.TemplateHeight - Self.Height) / 2` |
| Width | `lblColStatus.Width * 0.8` |
| Height | `24` |
| InputStatus | `ThisItem.Status` |

(The `* 0.8` leaves ~20% of the column as right-side breathing room,
matching the 120-in-150 ratio from the original spec. The pill still
sits at the left edge of the Status column.)

If `cmpStatusPill` doesn't appear under Custom, you skipped building it
— return to `01-app-setup.md` section 4 and create it first.

### Step 29 — View button (column 9)

Still inside the template, Insert → **Button**.

| Property | Value |
|----------|-------|
| Name | `btnView` |
| Text | `"View"` |
| X | `lblColAction.X` |
| Y | `(galUseCases.TemplateHeight - Self.Height) / 2` |
| Width | `lblColAction.Width * 0.8` |
| Height | `28` |
| Size | `12` |
| FontWeight | `FontWeight.Semibold` |
| Fill | `theme.Surface` |
| HoverFill | `RGBA(245, 230, 233, 1)` |
| Color | `theme.Maroon` |
| BorderColor | `theme.Maroon` |
| BorderThickness | `1` |
| RadiusTopLeft / TopRight / BottomLeft / BottomRight | `4` |
| OnSelect | `Select(Parent)` |

`Select(Parent)` programmatically fires the gallery row's OnSelect
(the `Navigate` from Step 25), so clicking View has the same effect as
clicking anywhere else on the row.

### Step 30 — Sanity check

Click anywhere outside the template to exit template-editing mode.
Preview:

- [ ] 10 rows visible (matching `colUseCases`).
- [ ] Each row shows UCID, name in bold, SBU, owner, a colored Status
      pill, FY, `$X.XM` value (or `"—"`), `"N days ago"`, and a View
      button.
- [ ] Clicking a row opens `scrDetail` with `selectedUC` populated
      (the detail screen may still be empty — that's OK).
- [ ] Typing `"Mortgage"` in Search filters to 1 row.
- [ ] Setting Status to `"Development"` filters to 2 rows.
- [ ] Reset → all 10 rows return.
- [ ] **Toggle the rail (hamburger).** Every column should reflow —
      narrower when the rail expands, wider when it collapses. The
      View button must never clip past the right edge of the gallery
      card. Row cells must stay aligned under their header labels in
      both states.

If a column drifts a few pixels left or right, the row label is
probably pointing at the wrong header column — cross-check Step 27
(`lblXXX.X` should equal `lblColXXX.X` for the same column number).

> **If you already built Parts 4/5 with the old fixed X/Width values
> (14, 114, 434, …, Width 100/320/120/…):** select each affected
> control in the tree and update only its X and Width to the formulas
> above. All other properties stay the same. The order to update is:
> Step 23 first (all 9 header labels), then Step 27 (7 row labels),
> then Step 28 (pill) and Step 29 (View button). Update the header
> first because the row controls reference it.

---

## Part 6 — Gallery footer

A small status bar showing visible / total counts, with a maroon
scroll-status dot on the right.

### Step 31 — Footer container

Inside `conGallery` (not inside the gallery template — back out first),
Insert → **Rectangle**.

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
| Fill | `theme.Border` |
| BorderThickness | `0` |

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
| Color | `theme.Ink3` |

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
| Fill | `theme.Maroon` |
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
| Color | `theme.Ink2` |

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

Press **F5** on `scrList` and walk through:

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
- [ ] Click any row → navigates to `scrDetail` with `selectedUC`
      populated.
- [ ] Click the rail's hamburger → rail collapses to 64px; content
      expands; gallery still works (last column may now extend past
      the visible area if you kept the wide widths — see Step 23
      note).

If all 9 boxes pass, `scrList` is done. Move on to `scrDetail` (Screen 3
in `02-build-guide.md`).

---

## Quick reference — things that go wrong

| Symptom | Cause | Fix |
|---------|-------|-----|
| Gallery shows no rows | `filteredUseCases` is empty, or filters too restrictive | Click `App` in the tree, check `Formulas` contains the `filteredUseCases =` block from `01-app-setup.md` section 3. Reset filters. |
| Gallery shows red squiggle on Items | `filteredUseCases` isn't defined as a named formula | Same as above — paste the block into `App.Formulas` (not `App.OnStart`). |
| Last column clips off the right when rail expands | Header / row controls were built with hard-coded X/Width instead of the `* Parent.Width / 1244` formulas | Apply the migration note at the bottom of Step 30 — update the 9 header labels (Step 23), then the 7 row labels (Step 27), then the pill (Step 28) and View button (Step 29). |
| Columns drift from the header labels | A row label points at the wrong header column | In Step 27, `lblXXX.X` must equal `lblColXXX.X` for the same column number (e.g. `lblSBU.X = lblColSBU.X`, not `lblColOwner.X`). |
| Header columns reflow but rows don't (or vice versa) | One side uses formulas, the other uses fixed values | Both Step 23 and Step 27 must use the formula approach. Mixing fixed and formula causes the rows to drift away from the header on rail toggle. |
| Status pill missing | `cmpStatusPill` not in tree | Build it per `01-app-setup.md` section 4 first. |
| Status pill shows the wrong color | `InputStatus` typed wrong, or doesn't match Code in `colStatus` | Status values must be one of: Rationale, DataPrep, Development, Testing, Deployment, Monitoring, Decommissioning. |
| Row click does nothing | OnSelect is on the template instead of the gallery | Click the gallery itself (not a child), put the formula from Step 25 there. |
| Reset doesn't clear search text | `Reset(txtSearch)` missing | Add `Reset(txtSearch);` to `btnReset.OnSelect` alongside `Set(filterSearch, "");`. |
| `Navigate(scrNew, ...)` errors | `scrNew` screen doesn't exist yet | Create a blank screen named exactly `scrNew`. You'll build it out in `02-build-guide.md` section 4. |
| `Navigate(scrDetail, ...)` errors | Same as above for `scrDetail` | Create a blank screen named exactly `scrDetail`. |
| Export button doesn't show a notification | `Notify(...)` only fires in app preview, not Studio edit mode | Press F5, then click. |
| Hover on rows does nothing visible | You're testing in Studio edit mode | Press F5 — hover effects only run in preview. |
| Filter card is too short / tall | Wrong Height on `conFilterCard` | Should be exactly `92`. |
| Gallery footer overlaps the last row | Gallery Height didn't reserve 40 for the footer | Set `galUseCases.Height = Parent.Height - 36 - 40`. |
