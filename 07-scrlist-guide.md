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

### Step 13 — Column X reference

You'll reuse these six X values. Each column is 200 wide with a 14-wide
gutter on either side.

| Column | Contents | X |
|--------|----------|----|
| 1 | Search | `14` |
| 2 | Status | `228` |
| 3 | SBU | `442` |
| 4 | FY | `656` |
| 5 | Owner | `870` |
| 6 | Reset | `1084` |

(All caption labels: Y=10, Height=14, Width=200. All inputs: Y=30,
Height=40, Width=200.)

### Step 14 — Search input + caption

Inside `conFilterCard`, Insert → **Label** (caption first):

| Property | Value |
|----------|-------|
| Name | `lblCapSearch` |
| Text | `"Search"` |
| X | `14` |
| Y | `10` |
| Width | `200` |
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
| Width | `200` |
| Height | `40` |
| Size | `13` |
| BorderColor | `theme.Border` |
| BorderThickness | `1` |
| Color | `theme.Ink` |
| OnChange | `Set(filterSearch, Self.Text)` |

### Step 15 — Status dropdown + caption

Caption `lblCapStatus`: same style as `lblCapSearch`, X=`228`, Y=`10`,
Text=`"Status"`.

Dropdown:

| Property | Value |
|----------|-------|
| Name | `ddStatus` |
| X | `228` |
| Y | `30` |
| Width | `200` |
| Height | `40` |
| Items | `["All Statuses","Rationale","Data Prep","Development","Testing","Deployment","Monitoring","Decommissioning"]` |
| Default | `filterStatus` |
| OnChange | `Set(filterStatus, Self.Selected.Value)` |
| BorderColor | `theme.Border` |
| BorderThickness | `1` |
| Color | `theme.Ink` |

### Step 16 — SBU dropdown + caption

Caption `lblCapSBU`: X=`442`, Y=`10`, Text=`"SBU"`.

Dropdown:

| Property | Value |
|----------|-------|
| Name | `ddSBU` |
| X | `442` |
| Y | `30` |
| Width | `200` |
| Height | `40` |
| Items | `["All SBUs","PBB","Capital Markets","Wealth","Commercial","Direct Banking"]` |
| Default | `filterSBU` |
| OnChange | `Set(filterSBU, Self.Selected.Value)` |
| BorderColor | `theme.Border` |
| BorderThickness | `1` |

### Step 17 — FY dropdown + caption

Caption `lblCapFY`: X=`656`, Y=`10`, Text=`"Fiscal year"`.

Dropdown:

| Property | Value |
|----------|-------|
| Name | `ddFY` |
| X | `656` |
| Y | `30` |
| Width | `200` |
| Height | `40` |
| Items | `["F26","F25","F24"]` |
| Default | `filterFY` |
| OnChange | `Set(filterFY, Self.Selected.Value)` |
| BorderColor | `theme.Border` |
| BorderThickness | `1` |

### Step 18 — Owner dropdown + caption

Caption `lblCapOwner`: X=`870`, Y=`10`, Text=`"Owner"`.

Dropdown:

| Property | Value |
|----------|-------|
| Name | `ddOwner` |
| X | `870` |
| Y | `30` |
| Width | `200` |
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
| X | `1084` |
| Y | `30` |
| Width | `120` |
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
- [ ] Type in search and select Status / SBU / FY — no visible effect
      on a gallery yet (you build it in Part 5).
- [ ] Click Reset — search clears, dropdowns return to defaults.

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

### Step 23 — Column labels (running-sum X)

All nine header labels live inside `recGalleryHeader`. Use these exact X
values — they're the running sum starting at 14. Same X / Width pattern
will repeat in Part 5 for the gallery template, so getting these right
matters.

| # | Name | Text | X | Width |
|---|------|------|----|-------|
| 1 | `lblColUCID` | `"UCID"` | `14` | `100` |
| 2 | `lblColName` | `"Use Case Name"` | `114` | `320` |
| 3 | `lblColSBU` | `"SBU"` | `434` | `120` |
| 4 | `lblColOwner` | `"AI Solution Owner"` | `554` | `160` |
| 5 | `lblColStatus` | `"Status"` | `714` | `150` |
| 6 | `lblColFY` | `"FY"` | `864` | `60` |
| 7 | `lblColValue` | `"Realized Value"` | `924` | `130` |
| 8 | `lblColUpdated` | `"Last Updated"` | `1054` | `110` |
| 9 | `lblColAction` | `""` | `1164` | `80` |

All nine share these properties:

| Property | Value |
|----------|-------|
| Y | `10` |
| Height | `16` |
| Size | `12` |
| FontWeight | `FontWeight.Semibold` |
| Color | `White` |
| Font | `theme.FontFamily` |

> **Width note:** the nine columns sum to 1244 px. With the rail
> collapsed the `conPage` width is ≈ 1318, so everything fits with room
> to spare. With the rail expanded `conPage` is ≈ 1098 wide and the
> last column will clip. Either keep the rail collapsed while using
> this screen, or shrink the columns proportionally and update Part 5's
> template controls to match the new X values.

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
`theme.FontFamily`. Height is 16 except where noted. **X and Width must
match the header table in Step 23 exactly** or the columns will drift.

| # | Control | Name | Text | X | Width | Size | Color | FontWeight |
|---|---------|------|------|----|-------|------|-------|-----------|
| 1 | Label | `lblUCID` | `ThisItem.UCID` | `14` | `100` | 12 | `theme.Ink3` | `FontWeight.Normal` |
| 2 | Label | `lblName` | `ThisItem.Name` | `114` | `320` | 13 | `theme.Ink` | `FontWeight.Semibold` |
| 3 | Label | `lblSBU` | `ThisItem.SBU` | `434` | `120` | 13 | `theme.Ink2` | `FontWeight.Normal` |
| 4 | Label | `lblOwner` | `ThisItem.Owner` | `554` | `160` | 13 | `theme.Ink2` | `FontWeight.Normal` |
| 5 | Label | `lblFY` | `ThisItem.FY` | `864` | `60` | 13 | `theme.Ink2` | `FontWeight.Normal` |
| 6 | Label | `lblValue` | (see below) | `924` | `130` | 13 | `theme.Ink` | `FontWeight.Normal` |
| 7 | Label | `lblUpdated` | (see below) | `1054` | `110` | 12 | `theme.Ink2` | `FontWeight.Normal` |

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
| X | `714` |
| Y | `(galUseCases.TemplateHeight - Self.Height) / 2` |
| Width | `120` |
| Height | `24` |
| InputStatus | `ThisItem.Status` |

If `cmpStatusPill` doesn't appear under Custom, you skipped building it
— return to `01-app-setup.md` section 4 and create it first.

### Step 29 — View button (column 9)

Still inside the template, Insert → **Button**.

| Property | Value |
|----------|-------|
| Name | `btnView` |
| Text | `"View"` |
| X | `1164` |
| Y | `(galUseCases.TemplateHeight - Self.Height) / 2` |
| Width | `64` |
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

If a column drifts a few pixels left or right, cross-check its X
against the table in Step 23.

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
| Last column clips off the right | Rail is expanded; column widths sum to > content width | Collapse the rail (hamburger), or shrink each column proportionally and update both Step 23 and Step 27. |
| Columns drift from the header labels | One template control has a wrong X | Cross-check every X in Step 27 against the X in Step 23 — they must match per column. |
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
