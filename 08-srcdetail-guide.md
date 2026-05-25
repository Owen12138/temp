# BOA Canvas App — Use Case Detail Screen (srcDetail) Step-by-Step

Use this guide to build the `srcDetail` (Use Case Detail) screen end to
end. This is the biggest screen in the app: a sticky action bar across
the top, a secondary side rail inside the page for switching between
sections, and seven mutually-exclusive content panels (Info, Contacts,
Value, Funds, Governance, Tech Review, Monthly Update) plus a pinned
submit row at the bottom. Build it in this order; don't skip ahead.

This guide assumes you have already completed:

- `01-app-setup.md` — `App.OnStart` initializes `gblTheme`,
  `selectedUC` (as a typed record with all 16 fields), `currentSection`,
  `colUseCases`, `colValueEntries`, `colGovernance`, `colTechReview`,
  `colStatus`, `colSBU`, `colUseCaseType`, `colRefreshFreq`,
  `colBenefitType`. `App.Formulas` has `ucValueRows`, `ucGovRows`,
  `ucTechRows`, `govDoneCount`, `govTotalCount`, `govProgressPct`,
  `realizedYTD`. The `cmpStatusPill` component exists.
- `06-left-rail-buttons-guide.md` — `conLeftRail` exists on `srcHome`
  (built as Buttons + Icons).
- The `02-build-guide.md` block for `srcHome.conHeader`
  (X: `If(sideCollapsed, 64, 220)`, Y: `0`, Height: `52`, Fill:
  `gblTheme.Maroon`).
- `srcList` exists with a working row click (Step 25 in
  `07-scrlist-guide.md`) that sets `selectedUC` and navigates here.

The reusable `cmpStatusPill` **can** be used on this screen (in the
section rail head) because that rail is not inside a gallery. The pill
inside the Value table (Step 38) does have to be inlined as primitives
for the same gallery-container restriction as `srcList`.

---

## Layout overview (read this first)

```
srcDetail (1366×768)
├── conLeftRail               ← copied from srcHome (X = 0)
├── conHeader                 ← copied from srcHome (Y = 0, H = 52)
├── conActionBar              (Y = 52, H = 56, white, sticky)
│   ├── btnBack               "← Back to List"
│   ├── lblCrumb              "Repository / UC-… · …"
│   ├── lblSaveNote           "Draft saved ✓ 14:32"
│   ├── btnSaveDraft
│   └── btnSubmit
└── conDetailLayout           (Y = 108, fills remaining height)
    ├── conSectionRail        (X = 0, W = 220, fills height) — internal nav
    │   ├── conRailHead       (UCID + FY, name, status pill)
    │   └── galSectionNav     (vertical gallery of 7 section links)
    ├── conContent            (X = 220, fills remaining width)
    │   ├── conSectionInfo        Visible: currentSection = "Info"
    │   ├── conSectionContacts    Visible: currentSection = "Contacts"
    │   ├── conSectionValue       Visible: currentSection = "Value"
    │   ├── conSectionFunds       Visible: currentSection = "Funds"
    │   ├── conSectionGov         Visible: currentSection = "Gov"
    │   ├── conSectionTech        Visible: currentSection = "Tech"
    │   └── conSectionUpdates     Visible: currentSection = "Updates"
    └── conSubmitZone         (pinned to bottom of conDetailLayout, H = 60)
```

Only **one** `conSection*` is visible at a time. Switching is just a
variable: clicking a row in `galSectionNav` sets `currentSection`, and
each section's `Visible` formula compares against it. No tab control,
no animation, no DOM manipulation — Power Apps native.

A note on widths: with the rail collapsed (`sideCollapsed = true`,
default), the page area is `1366 - 64 = 1302`. With it expanded:
`1366 - 220 = 1146`. The internal `conSectionRail` is a fixed 220, so
`conContent` flexes between 1082 (collapsed) and 926 (expanded). Form
fields below use `(Parent.Width - 48) / 2` for two-column widths so
they reflow automatically.

---

## Part 1 — Create the screen and base scaffolding

### Step 1 — Add the screen

Top toolbar → **+ New screen** → **Blank**. Rename to `srcDetail`. In
the tree, position it below `srcList`.

> **Tip:** If you already created `srcDetail` as a blank stub when
> building `srcList` (Step 25), you can keep it — just delete any
> placeholder controls before continuing.

### Step 2 — Copy the rail and header from srcHome

In the tree on `srcHome`:

1. Click `conLeftRail`, **Ctrl+C**.
2. Click `srcDetail` in the tree, **Ctrl+V**. The rail appears in the
   same X/Y because its formulas reference `Parent` and `sideCollapsed`.
3. Repeat for `conHeader`: Ctrl+C on `srcHome` → click `srcDetail` →
   Ctrl+V.

### Step 3 — Screen.OnVisible

Click `srcDetail` in the tree (the screen itself). In the property
dropdown, choose **OnVisible**. Paste:

```powerfx
If(IsBlank(selectedUC) || selectedUC.UCID = "",
    Navigate(srcList, ScreenTransition.None)
);
Set(currentSection, "Info")
```

The first line bounces the user back to the list if they somehow land
on Detail without a row selected (e.g. by clicking the rail's "Detail"
link directly, or via a deep link in a future build). The second line
resets the section tab to Info every time the screen opens — so the
user always sees the main form first, even if they last left it on
Value or Governance.

### Step 4 — Sanity check

Press **F5**. You should see:

- [ ] Left rail visible. The "Detail" / use-case row tints maroon if
      guide 06 Step 5 includes `srcDetail` in its active-match formula
      (otherwise no row tints — that's fine).
- [ ] Maroon header strip across the top.
- [ ] Below the header, an empty light-gray area (will become the
      action bar and detail layout in Parts 2–3).
- [ ] If you started F5 directly on `srcDetail` (no selected use
      case), you bounce to `srcList`. To test Detail standalone, click
      a row in `srcList` first, then check.

---

## Part 2 — Action bar (sticky row under the header)

A white horizontal strip showing the back link, breadcrumb, save
timestamp, and the right-aligned Save Draft + Submit buttons.

### Step 5 — Action bar container

On `srcDetail`, Insert → **Container** (classic).

| Property | Value |
|----------|-------|
| Name | `conActionBar` |
| X | `If(sideCollapsed, 64, 220)` |
| Y | `52` |
| Width | `Parent.Width - If(sideCollapsed, 64, 220)` |
| Height | `56` |
| Fill | `gblTheme.Surface` |
| BorderThickness | `0` |

Add a 1px bottom border inside the container so the action bar reads
as a distinct strip. Inside `conActionBar`, Insert → **Rectangle**:

| Property | Value |
|----------|-------|
| Name | `recActionBarBorder` |
| X | `0` |
| Y | `Parent.Height - 1` |
| Width | `Parent.Width` |
| Height | `1` |
| Fill | `gblTheme.Border` |
| BorderThickness | `0` |

### Step 6 — Back button (ghost style)

Inside `conActionBar`, Insert → **Button**.

| Property | Value |
|----------|-------|
| Name | `btnBack` |
| Text | `"← Back to List"` |
| X | `14` |
| Y | `(Parent.Height - Self.Height) / 2` |
| Width | `130` |
| Height | `32` |
| Size | `13` |
| FontWeight | `FontWeight.Semibold` |
| Fill | `RGBA(0,0,0,0)` |
| HoverFill | `RGBA(0,0,0,0)` |
| PressedFill | `RGBA(0,0,0,0)` |
| Color | `gblTheme.Ink2` |
| HoverColor | `gblTheme.Maroon` |
| BorderThickness | `0` |
| OnSelect | `Navigate(srcList, ScreenTransition.None)` |

### Step 7 — Breadcrumb label

Inside `conActionBar`, Insert → **Label**.

| Property | Value |
|----------|-------|
| Name | `lblCrumb` |
| Text | `"Repository  /  " & selectedUC.UCID & "  ·  " & selectedUC.Name` |
| X | `btnBack.X + btnBack.Width + 8` |
| Y | `(Parent.Height - Self.Height) / 2` |
| Width | `Parent.Width - Self.X - 380` |
| Height | `20` |
| Size | `12` |
| Color | `gblTheme.Ink3` |
| Font | `gblTheme.FontFamily` |
| Overflow | `Overflow.Hidden` |

Width subtracts 380 so the breadcrumb truncates rather than colliding
with the right-side controls (lblSaveNote + the two buttons).

### Step 8 — Save timestamp label

Inside `conActionBar`, Insert → **Label**.

| Property | Value |
|----------|-------|
| Name | `lblSaveNote` |
| Text | `"Draft saved ✓ " & Text(Now(), "[$-en-US]hh:mm")` |
| X | `Parent.Width - 380` |
| Y | `(Parent.Height - Self.Height) / 2` |
| Width | `120` |
| Height | `18` |
| Size | `11` |
| Color | `gblTheme.Ink3` |
| Font | `gblTheme.FontFamily` |
| Align | `Align.Right` |

> **v1 caveat:** `Now()` re-evaluates whenever the screen re-paints
> (e.g. when you click something), so the timestamp updates roughly
> continuously. That's not the real "last saved at" — wire it to a
> `Set(lastSavedAt, Now())` inside `btnSaveDraft.OnSelect` later for
> a real timestamp.

### Step 9 — Save Draft button (secondary style)

Inside `conActionBar`, Insert → **Button**.

| Property | Value |
|----------|-------|
| Name | `btnSaveDraft` |
| Text | `"Save Draft"` |
| X | `Parent.Width - 252` |
| Y | `(Parent.Height - Self.Height) / 2` |
| Width | `110` |
| Height | `36` |
| Size | `13` |
| FontWeight | `FontWeight.Semibold` |
| Fill | `gblTheme.Surface` |
| HoverFill | `RGBA(245, 230, 233, 1)` |
| Color | `gblTheme.Maroon` |
| HoverColor | `gblTheme.Maroon` |
| BorderColor | `gblTheme.Maroon` |
| BorderThickness | `1` |
| RadiusTopLeft / TopRight / BottomLeft / BottomRight | `4` |
| OnSelect | `Notify("Draft saved.", NotificationType.Information)` |

### Step 10 — Submit button (primary style)

Inside `conActionBar`, Insert → **Button**.

| Property | Value |
|----------|-------|
| Name | `btnSubmit` |
| Text | `"Submit Assessment"` |
| X | `btnSaveDraft.X + btnSaveDraft.Width + 8` |
| Y | `(Parent.Height - Self.Height) / 2` |
| Width | `130` |
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
| OnSelect | (see formula below) |

```powerfx
Patch(colUseCases, selectedUC, { LastUpdated: Today() });
Notify("Assessment submitted. Power Automate flow triggered.", NotificationType.Success)
```

(In production, replace the `Notify` with a flow call —
`BOASubmitAssessment.Run(selectedUC.UCID)` — once the flow exists.
For v1 this is enough to demo the round-trip.)

### Step 11 — Sanity check

Click a row in `srcList` to land on `srcDetail`, then preview. You
should see:

- [ ] White strip under the maroon header, 56 px tall, with a 1 px
      gray line at the bottom.
- [ ] Left: `"← Back to List"` — clicking returns to `srcList`.
- [ ] Center-left: breadcrumb showing the row's UCID and name.
- [ ] Right: `"Draft saved ✓ HH:MM"`, then Save Draft (outlined
      maroon) and Submit Assessment (solid maroon).
- [ ] Hover on Back tints text maroon; hover on Save tints fill pink;
      hover on Submit darkens to `MaroonDeep`.
- [ ] Toggle the rail — the action bar slides with it; nothing
      overflows.

---

## Part 3 — Detail layout shell (two-column scaffold)

A wrapper container that hosts the inner section rail (left), the
section content (right), and a pinned submit zone (bottom).

### Step 12 — Detail layout container

On `srcDetail`, Insert → **Container** (classic).

| Property | Value |
|----------|-------|
| Name | `conDetailLayout` |
| X | `If(sideCollapsed, 64, 220)` |
| Y | `108` |
| Width | `Parent.Width - If(sideCollapsed, 64, 220)` |
| Height | `Parent.Height - 108` |
| Fill | `gblTheme.Bg` |
| BorderThickness | `0` |

Y is `52` (header) + `56` (action bar) = `108`. Height fills whatever
is left below.

### Step 13 — Section rail container (internal nav)

Inside `conDetailLayout`, Insert → **Container** (classic).

| Property | Value |
|----------|-------|
| Name | `conSectionRail` |
| X | `0` |
| Y | `0` |
| Width | `220` |
| Height | `Parent.Height` |
| Fill | `gblTheme.Surface` |
| BorderThickness | `0` |

Add a 1px right border so the rail visually separates from the
content. Inside `conSectionRail`, Insert → **Rectangle**:

| Property | Value |
|----------|-------|
| Name | `recSectionRailBorder` |
| X | `Parent.Width - 1` |
| Y | `0` |
| Width | `1` |
| Height | `Parent.Height` |
| Fill | `gblTheme.Border` |
| BorderThickness | `0` |

### Step 14 — Content container

Inside `conDetailLayout`, Insert → **Container** (classic).

| Property | Value |
|----------|-------|
| Name | `conContent` |
| X | `220` |
| Y | `0` |
| Width | `Parent.Width - 220` |
| Height | `Parent.Height - 60` |
| Fill | `RGBA(0,0,0,0)` |
| BorderThickness | `0` |

Height reserves 60 at the bottom for `conSubmitZone` (Step 50). Fill
is transparent so the layout's `gblTheme.Bg` shows through and gives
the section cards a backdrop.

### Step 15 — Submit zone container (pinned bottom)

Inside `conDetailLayout`, Insert → **Container** (classic).

| Property | Value |
|----------|-------|
| Name | `conSubmitZone` |
| X | `220` |
| Y | `Parent.Height - 60` |
| Width | `Parent.Width - 220` |
| Height | `60` |
| Fill | `gblTheme.Surface` |
| BorderThickness | `0` |

We'll add the top border, message label, and two buttons in Step 50.
Leave it empty for now.

### Step 16 — Sanity check

Preview:

- [ ] Below the action bar, the detail area is light gray (the bg).
- [ ] A 220-px white strip runs down the left side of the detail area
      with a 1px right border.
- [ ] A 60-px white strip pins to the bottom-right (currently empty —
      Step 50 fills it in).
- [ ] Toggle the rail. The 220 internal section rail stays a fixed
      220 wide; only the right-side content area resizes. No overflow.

---

## Part 4 — Section rail head (UCID, name, status pill)

The top block of the internal rail. Shows which use case you're
viewing and its current status.

### Step 17 — Rail head container

Inside `conSectionRail`, Insert → **Container** (classic).

| Property | Value |
|----------|-------|
| Name | `conRailHead` |
| X | `0` |
| Y | `0` |
| Width | `Parent.Width` |
| Height | `100` |
| Fill | `RGBA(0,0,0,0)` |
| BorderThickness | `0` |
| PaddingLeft | `16` |
| PaddingRight | `16` |
| PaddingTop | `14` |

Add a 1px bottom border. Inside `conRailHead`, Insert → **Rectangle**:

| Property | Value |
|----------|-------|
| Name | `recRailHeadBorder` |
| X | `0` |
| Y | `Parent.Height - 1` |
| Width | `Parent.Width` |
| Height | `1` |
| Fill | `gblTheme.Border` |
| BorderThickness | `0` |

### Step 18 — UCID + FY caption

Inside `conRailHead`, Insert → **Label**.

| Property | Value |
|----------|-------|
| Name | `lblRailUCID` |
| Text | `selectedUC.UCID & "  ·  " & selectedUC.FY` |
| X | `16` |
| Y | `14` |
| Width | `Parent.Width - 32` |
| Height | `16` |
| Size | `11` |
| Color | `gblTheme.Ink3` |
| Font | `gblTheme.FontFamily` |

### Step 19 — Use case name

Inside `conRailHead`, Insert → **Label**.

| Property | Value |
|----------|-------|
| Name | `lblRailName` |
| Text | `selectedUC.Name` |
| X | `16` |
| Y | `lblRailUCID.Y + lblRailUCID.Height + 2` |
| Width | `Parent.Width - 32` |
| Height | `38` |
| Size | `15` |
| FontWeight | `FontWeight.Semibold` |
| Color | `gblTheme.Ink` |
| Font | `gblTheme.FontFamily` |
| Overflow | `Overflow.Hidden` |

Height of 38 fits a 2-line name; long names truncate at line 2.

### Step 20 — Status pill (using cmpStatusPill component)

Inside `conRailHead`, Insert → **Custom** → **cmpStatusPill**.

> **Why the component here but not on srcList?** This rail is **not**
> inside a gallery. Power Apps only blocks components when they're
> children of a Container nested in a gallery template. Here we're
> three levels deep (screen → conDetailLayout → conSectionRail →
> conRailHead) but no gallery in the chain, so the component is fine.

| Property | Value |
|----------|-------|
| Name | `pillRailStatus` |
| X | `16` |
| Y | `lblRailName.Y + lblRailName.Height + 6` |
| Width | `120` |
| Height | `20` |
| InputStatus | `selectedUC.Status` |

### Step 21 — Sanity check

Preview from `srcList` → click any row:

- [ ] Top of the internal rail shows `UC-XXXX · F26` in small gray.
- [ ] Below it, the use case name in semibold dark text (wraps to 2
      lines if long; truncates beyond).
- [ ] Below the name, a colored status pill matching the row's
      `Status` field (e.g. blue dot + "Development" for UC-0142).
- [ ] Click a different row in `srcList`, return to Detail — the
      UCID, name, and pill all update. (They're driven by
      `selectedUC` so they refresh automatically.)

---

## Part 5 — Section nav gallery

A vertical gallery of 7 section links. The active one tints maroon
with a left accent stripe.

### Step 22 — Insert the gallery

Inside `conSectionRail`, Insert → **Gallery** → **Blank vertical**.

| Property | Value |
|----------|-------|
| Name | `galSectionNav` |
| X | `0` |
| Y | `conRailHead.Y + conRailHead.Height` |
| Width | `Parent.Width` |
| Height | `Parent.Height - Self.Y` |
| TemplateSize | `40` |
| TemplatePadding | `0` |
| ShowScrollbar | `false` |
| BorderThickness | `0` |
| Items | (see formula below) |
| OnSelect | `Set(currentSection, ThisItem.Key)` |

```powerfx
Table(
    {Key: "Info",     Label: "Use Case Info",    Num: Blank()},
    {Key: "Contacts", Label: "Contacts",         Num: Blank()},
    {Key: "Value",    Label: "Value",            Num: CountRows(ucValueRows)},
    {Key: "Funds",    Label: "Funds",            Num: Blank()},
    {Key: "Gov",      Label: "Governance",       Num: govDoneCount},
    {Key: "Tech",     Label: "Technical Review", Num: Blank()},
    {Key: "Updates",  Label: "Monthly Update",   Num: Blank()}
)
```

> **A note on `Num`:** the spec shows `Num` as the count of rows for
> the Value and Gov sections so each link can display a badge (e.g.
> "Value · 3"). We surface only `Num` (an integer) in the gallery,
> and the badge is hidden when `Num` is blank. If you want the
> "X/Y" form for Governance, change to
> `Num: Text(govDoneCount) & "/" & Text(govTotalCount)` and update
> the badge label type in Step 26.

### Step 23 — Row template structure

Click the small chevron next to `galSectionNav` to enter the template
(dashed border). The template hosts three children:

1. `recNavAccent` — 3 px left stripe, maroon when active.
2. `lblNavLabel` — section name.
3. `lblNavBadge` — small count badge on the right (hidden when blank).

Inside the template, Insert → **Rectangle**:

| Property | Value |
|----------|-------|
| Name | `recNavAccent` |
| X | `0` |
| Y | `0` |
| Width | `3` |
| Height | `Parent.TemplateHeight` |
| Fill | `If(ThisItem.Key = currentSection, gblTheme.Maroon, RGBA(0,0,0,0))` |
| BorderThickness | `0` |

### Step 24 — Row label

Still inside the template, Insert → **Label**:

| Property | Value |
|----------|-------|
| Name | `lblNavLabel` |
| Text | `ThisItem.Label` |
| X | `16` |
| Y | `0` |
| Width | `Parent.TemplateWidth - 60` |
| Height | `Parent.TemplateHeight` |
| Size | `13` |
| Font | `gblTheme.FontFamily` |
| Color | `If(ThisItem.Key = currentSection, gblTheme.Maroon, gblTheme.Ink2)` |
| FontWeight | `If(ThisItem.Key = currentSection, FontWeight.Bold, FontWeight.Normal)` |
| VerticalAlign | `VerticalAlign.Middle` |
| PaddingLeft | `0` |

### Step 25 — Row badge (count chip)

Still inside the template, Insert → **Label**:

| Property | Value |
|----------|-------|
| Name | `lblNavBadge` |
| Text | `If(IsBlank(ThisItem.Num), "", Text(ThisItem.Num))` |
| Visible | `!IsBlank(ThisItem.Num)` |
| X | `Parent.TemplateWidth - 40` |
| Y | `(Parent.TemplateHeight - Self.Height) / 2` |
| Width | `28` |
| Height | `20` |
| Size | `10` |
| FontWeight | `FontWeight.Semibold` |
| Color | `If(ThisItem.Key = currentSection, White, gblTheme.Ink3)` |
| Fill | `If(ThisItem.Key = currentSection, gblTheme.Maroon, gblTheme.Border)` |
| Align | `Align.Center` |
| VerticalAlign | `VerticalAlign.Middle` |
| RadiusTopLeft / TopRight / BottomLeft / BottomRight | `10` |
| PaddingLeft | `0` |
| PaddingRight | `0` |

### Step 26 — Sanity check

Click outside the template to exit edit mode. Preview from `srcList`
→ click any row:

- [ ] Seven section links visible in the internal rail: Use Case
      Info, Contacts, Value, Funds, Governance, Technical Review,
      Monthly Update.
- [ ] "Use Case Info" is highlighted: 3px maroon accent on the left,
      label is bold maroon.
- [ ] Value and Governance show small count badges on the right
      (e.g. "3" and "3" for UC-0142).
- [ ] Click "Governance" — the accent and label move to that row;
      Info loses its accent.
- [ ] Click back to "Use Case Info" — accent returns. The content
      area on the right is still empty at this stage; you'll add
      panels in Parts 6–10.

---

## Part 6 — Section: Use Case Info (form panel)

The biggest section. Header, horizontal status stepper, then a
two-column form of 12 fields.

### Step 27 — Section container

Inside `conContent`, Insert → **Container** (classic).

| Property | Value |
|----------|-------|
| Name | `conSectionInfo` |
| X | `24` |
| Y | `24` |
| Width | `Parent.Width - 48` |
| Height | `Parent.Height - 48` |
| Fill | `gblTheme.Surface` |
| BorderThickness | `1` |
| BorderColor | `gblTheme.Border` |
| RadiusTopLeft / TopRight / BottomLeft / BottomRight | `4` |
| Visible | `currentSection = "Info"` |
| PaddingLeft | `28` |
| PaddingRight | `28` |
| PaddingTop | `24` |
| PaddingBottom | `24` |

### Step 28 — Section title + underline

Inside `conSectionInfo`, Insert → **Label**:

| Property | Value |
|----------|-------|
| Name | `lblInfoTitle` |
| Text | `"Use Case Info"` |
| X | `28` |
| Y | `24` |
| Width | `300` |
| Height | `24` |
| Size | `16` |
| FontWeight | `FontWeight.Bold` |
| Color | `gblTheme.Maroon` |
| Font | `gblTheme.FontFamily` |

Inside `conSectionInfo`, Insert → **Rectangle**:

| Property | Value |
|----------|-------|
| Name | `recInfoTitleBorder` |
| X | `28` |
| Y | `lblInfoTitle.Y + lblInfoTitle.Height + 4` |
| Width | `Parent.Width - 56` |
| Height | `2` |
| Fill | `gblTheme.Maroon` |
| BorderThickness | `0` |

### Step 29 — Status stepper — current-status helper

Power Fx inside the gallery template can't easily know the row index
of the current status, so we calculate it once and store it in a
hidden label. Inside `conSectionInfo`, Insert → **Label**:

| Property | Value |
|----------|-------|
| Name | `lblCurrentStatusIdx` |
| Text | `LookUp(Table({C:"Rationale",I:1},{C:"DataPrep",I:2},{C:"Development",I:3},{C:"Testing",I:4},{C:"Deployment",I:5},{C:"Monitoring",I:6}), C = selectedUC.Status, I)` |
| X | `0` |
| Y | `0` |
| Width | `1` |
| Height | `1` |
| Visible | `false` |

Setting Visible=false hides it but the Text formula still evaluates
and is reachable as `lblCurrentStatusIdx.Text` from sibling controls.

### Step 30 — Status stepper — track line

Inside `conSectionInfo`, Insert → **Rectangle** (this is the inactive
track behind all 6 circles):

| Property | Value |
|----------|-------|
| Name | `recStepTrack` |
| X | `60` |
| Y | `recInfoTitleBorder.Y + recInfoTitleBorder.Height + 36` |
| Width | `Parent.Width - 56 - 80` |
| Height | `2` |
| Fill | `gblTheme.BorderStrong` |
| BorderThickness | `0` |

The `-80` reserves 40 px of padding on each end so the line doesn't
poke past the first and last circles.

Then Insert → **Rectangle** for the filled portion:

| Property | Value |
|----------|-------|
| Name | `recStepTrackFill` |
| X | `recStepTrack.X` |
| Y | `recStepTrack.Y` |
| Width | `recStepTrack.Width * (Value(lblCurrentStatusIdx.Text) - 1) / 5` |
| Height | `2` |
| Fill | `gblTheme.Maroon` |
| BorderThickness | `0` |

`(currentIdx - 1) / 5` is the fraction of the track to fill (0 at
Rationale, 1.0 at Monitoring).

### Step 31 — Status stepper — horizontal gallery

Inside `conSectionInfo`, Insert → **Gallery** → **Blank horizontal**.

| Property | Value |
|----------|-------|
| Name | `galStepper` |
| X | `28` |
| Y | `recStepTrack.Y - 24` |
| Width | `Parent.Width - 56` |
| Height | `64` |
| TemplateSize | `(Self.Width) / 6` |
| TemplatePadding | `0` |
| ShowScrollbar | `false` |
| BorderThickness | `0` |
| Items | (see formula below) |

```powerfx
Table(
    {Idx: 1, Code: "Rationale",   Label: "Rationale"},
    {Idx: 2, Code: "DataPrep",    Label: "Data Prep"},
    {Idx: 3, Code: "Development", Label: "Development"},
    {Idx: 4, Code: "Testing",     Label: "Testing"},
    {Idx: 5, Code: "Deployment",  Label: "Deployment"},
    {Idx: 6, Code: "Monitoring",  Label: "Monitoring"}
)
```

`TemplateSize` for a horizontal gallery is the **width** of each row,
not the height. Dividing the gallery width by 6 gives equal cells.

> **Tree order matters:** drag `galStepper` so it sits **above**
> `recStepTrack` in the tree (later in the tree = on top in Power
> Apps). That way the circle backgrounds you add in Step 32 cover
> the line where the circle sits, so the line appears to thread
> through the circles rather than over them. Easier alternative: it
> still looks fine if the line crosses the circles — the circles are
> filled, so the line gets covered visually inside the dot.

### Step 32 — Status stepper — row template

Enter the template (chevron next to `galStepper`). Inside, Insert →
**Icons** → **Circle**:

| Property | Value |
|----------|-------|
| Name | `circStep` |
| X | `(Parent.TemplateWidth - Self.Width) / 2` |
| Y | `20` |
| Width | `18` |
| Height | `18` |
| BorderThickness | `2` |
| BorderColor | `If(ThisItem.Idx <= Value(lblCurrentStatusIdx.Text), gblTheme.Maroon, gblTheme.BorderStrong)` |
| Fill | `If(ThisItem.Idx <= Value(lblCurrentStatusIdx.Text), gblTheme.Maroon, gblTheme.Surface)` |

Still inside the template, Insert → **Label**:

| Property | Value |
|----------|-------|
| Name | `lblStep` |
| Text | `ThisItem.Label` |
| X | `0` |
| Y | `circStep.Y + circStep.Height + 4` |
| Width | `Parent.TemplateWidth` |
| Height | `18` |
| Size | `11` |
| Align | `Align.Center` |
| Color | `If(ThisItem.Idx <= Value(lblCurrentStatusIdx.Text), gblTheme.Maroon, gblTheme.Ink3)` |
| FontWeight | `If(ThisItem.Idx = Value(lblCurrentStatusIdx.Text), FontWeight.Bold, FontWeight.Normal)` |
| Font | `gblTheme.FontFamily` |

Click outside to exit the template.

### Step 33 — Form fields — first 6 (two-column grid)

We have 12 fields total, arranged in 6 rows × 2 columns. Each cell
uses a label + input pair. To keep things compact, we define column
math once:

- **Column width:** `(Parent.Width - 56 - 24) / 2` — total width
  minus 28 px padding × 2 minus 24 px gap between columns, divided
  by 2.
- **Left col X:** `28`
- **Right col X:** `28 + ((Parent.Width - 56 - 24) / 2) + 24`
- **Row Y:** `lblFormStart.Y + (rowIndex - 1) * 76` (each row 76 tall
  for caption + input + gap).

Add a hidden anchor label so all field Y values can reference it
consistently. Inside `conSectionInfo`, Insert → **Label**:

| Property | Value |
|----------|-------|
| Name | `lblFormStart` |
| Text | `""` |
| X | `0` |
| Y | `galStepper.Y + galStepper.Height + 24` |
| Width | `1` |
| Height | `1` |
| Visible | `false` |

Now build the form. For each field, you'll insert **two** controls:
a caption Label and the input. The caption sits at Y = row Y, the
input at Y = row Y + 20. Below is the table; build top to bottom,
left first then right.

**Row 1 — Use Case Name (full width, required):**

Caption `lblCapName` (full width):

| Property | Value |
|----------|-------|
| Text | `"⬤ Use Case Name *"` (or `"Use Case Name *"`) |
| X | `28` |
| Y | `lblFormStart.Y` |
| Width | `Parent.Width - 56` |
| Height | `16` |
| Size | `12` |
| FontWeight | `FontWeight.Semibold` |
| Color | `gblTheme.Ink2` |

Input `txtName` (Text input, full width):

| Property | Value |
|----------|-------|
| X | `28` |
| Y | `lblCapName.Y + lblCapName.Height + 4` |
| Width | `Parent.Width - 56` |
| Height | `36` |
| Default | `selectedUC.Name` |
| BorderColor | `gblTheme.Border` |
| BorderThickness | `1` |
| Color | `gblTheme.Ink` |
| Size | `13` |
| OnChange | `Patch(colUseCases, selectedUC, { Name: Self.Text }); Set(selectedUC, LookUp(colUseCases, UCID = selectedUC.UCID))` |

**Row 2 — Problem Statement (full, multi-line):**

Caption `lblCapProblem`: Y=`lblFormStart.Y + 64`, same style,
Text=`"Problem Statement"`.

Input `txtProblem` (Text input, mode Multiline):

| Property | Value |
|----------|-------|
| X | `28` |
| Y | `lblCapProblem.Y + 20` |
| Width | `Parent.Width - 56` |
| Height | `72` |
| Mode | `TextMode.MultiLine` |
| Default | `selectedUC.ProblemStatement` |
| BorderColor / Color / Size | (same as txtName) |
| OnChange | `Patch(colUseCases, selectedUC, { ProblemStatement: Self.Text }); Set(selectedUC, LookUp(colUseCases, UCID = selectedUC.UCID))` |

**Row 3 — AI Solution Description (full, multi-line):**

Caption `lblCapSolution`: Y=`lblCapProblem.Y + 100`, Text=`"AI
Solution Description"`.

Input `txtSolution`: same as `txtProblem` but Default
`selectedUC.AISolution` and OnChange writes to `AISolution`.

### Step 34 — Form fields — remaining 9 (two-column)

Below the three full-width fields, we have nine half-width fields in
a 5-row × 2-col grid (last row only fills the left cell). For each:
caption Label, then Dropdown/DatePicker/TextInput input. Same OnChange
pattern. Build them in this order:

**Helper formulas you'll reuse:**

- Left col X: `28`
- Right col X: `28 + ((Parent.Width - 56 - 24) / 2) + 24`
- Half col Width: `(Parent.Width - 56 - 24) / 2`
- Row N Y (N starts at 1 for the first row after Solution):
  `lblCapSolution.Y + 100 + (N - 1) * 76`

| # | Position | Caption | Input control | Items / Default | OnChange (field name in Patch) |
|---|----------|---------|---------------|-----------------|--------------------------------|
| 1 | Row 1, Left | `"Type of Use Case"` | Dropdown `ddType` | Items: `colUseCaseType`, Default: `selectedUC.Type` | `Type` ← `Self.Selected.Value` |
| 2 | Row 1, Right | `"Current Status *"` | Dropdown `ddStatus2` | Items: `colStatus.Label`, Default: `LookUp(colStatus, Code = selectedUC.Status, Label)` | `Status` ← `LookUp(colStatus, Label = Self.Selected.Value, Code)` |
| 3 | Row 2, Left | `"SBU *"` | Dropdown `ddSBU2` | Items: `colSBU`, Default: `selectedUC.SBU` | `SBU` ← `Self.Selected.Value` |
| 4 | Row 2, Right | `"LOB / Sub-LOB"` | TextInput `txtLOB` | Default: `selectedUC.LOB` | `LOB` ← `Self.Text` |
| 5 | Row 3, Left | `"Other LOBs Impacted"` | TextInput `txtOtherLOB` | Default: `""` | _(not stored in v1)_ |
| 6 | Row 3, Right | `"Completion Date (target)"` | DatePicker `dpTarget` | Default: `selectedUC.TargetDate` | `TargetDate` ← `Self.SelectedDate` |
| 7 | Row 4, Left | `"Output Deliverable"` | Dropdown `ddOutput` | Items: `["Real-time scoring", "Dataset / Extract", "Report", "Dashboard"]`, Default: `Blank()` | _(not stored in v1)_ |
| 8 | Row 4, Right | `"Refresh Frequency"` | Dropdown `ddRefresh` | Items: `colRefreshFreq`, Default: `selectedUC.RefreshFreq` | `RefreshFreq` ← `Self.Selected.Value` |
| 9 | Row 5, Left | `"Prerequisite for Other Initiatives"` | Dropdown `ddPrereq` | Items: `["Yes", "No"]`, Default: `Blank()` | _(not stored in v1)_ |

**Caption shared properties (apply to every `lblCap…` above):**

| Property | Value |
|----------|-------|
| Height | `16` |
| Size | `12` |
| FontWeight | `FontWeight.Semibold` |
| Color | `gblTheme.Ink2` |
| Font | `gblTheme.FontFamily` |

For required-field captions (rows marked `*`), wrap the label text in
the maroon-asterisk pattern or just include the `*` in the text as
shown — adjust to taste.

**Input shared properties** (apply to every Dropdown / TextInput /
DatePicker):

| Property | Value |
|----------|-------|
| Height | `36` |
| Width | `(Parent.Width - 56 - 24) / 2` (or the full-width formula if applicable) |
| BorderColor | `gblTheme.Border` |
| BorderThickness | `1` |
| Color | `gblTheme.Ink` |
| Size | `13` |

For each input with a stored field in the OnChange table, the OnChange
formula template is:

```powerfx
Patch(colUseCases, selectedUC, { <FieldName>: <ValueExpression> });
Set(selectedUC, LookUp(colUseCases, UCID = selectedUC.UCID))
```

(The `Set(selectedUC, …)` line keeps `selectedUC` fresh so other
sections — like the rail head pill and the status stepper — react
immediately to changes.)

### Step 35 — Sanity check

From `srcList`, click UC-0142. On Detail with section = Info:

- [ ] Section card visible: white, rounded corners, gray border.
- [ ] "Use Case Info" title in maroon, with a 2px maroon underline.
- [ ] Status stepper: 6 circles with labels. Circles 1–3 are filled
      maroon (Development is step 3); circles 4–6 are hollow gray.
      The line behind connects them with maroon up to step 3.
- [ ] Form below: Name, Problem Statement, AI Solution Description
      (full width), then a 2-column grid of Type, Status, SBU,
      LOB, Other LOBs, Completion Date, Output, Refresh, Prerequisite.
- [ ] Editing the Name field and tabbing out updates the rail head's
      "Name" label and the breadcrumb in the action bar.
- [ ] Changing the Status dropdown to "Testing" causes the stepper to
      advance: circle 4 fills maroon, the maroon track extends.
- [ ] Toggle the rail. Both columns widen/narrow proportionally; no
      input overflows its column.

If the stepper circles don't move when you change Status, check
`lblCurrentStatusIdx.Text` actually returns a number — its formula
must reference `selectedUC.Status` (the **code**, not the label), and
the dropdown's OnChange must write the **code** back (via
`LookUp(colStatus, Label = Self.Selected.Value, Code)`).

---

## Part 7 — Section: Contacts

For v1, six static person cards in a 2-column grid. The People-picker
swap-in is documented but left for v2.

### Step 36 — Section container

Inside `conContent`, Insert → **Container** (classic), same shape as
`conSectionInfo` (Step 27):

| Property | Value |
|----------|-------|
| Name | `conSectionContacts` |
| X / Y / Width / Height / Fill / Border / Radius / Padding | (copy from `conSectionInfo`) |
| Visible | `currentSection = "Contacts"` |

### Step 37 — Title + underline

Same pattern as Step 28 but with name `lblContactsTitle`,
Text=`"Contacts"`, and a sibling `recContactsTitleBorder`.

### Step 38 — Person card grid (6 cards in 2 cols × 3 rows)

We'll build one card and clone the rest. Each card is a 48-tall
container with avatar + name + role.

Inside `conSectionContacts`, Insert → **Container**:

| Property | Value |
|----------|-------|
| Name | `conPersonOwner` |
| X | `28` |
| Y | `recContactsTitleBorder.Y + recContactsTitleBorder.Height + 24` |
| Width | `(Parent.Width - 56 - 24) / 2` |
| Height | `48` |
| Fill | `RGBA(250,250,250,1)` |
| BorderThickness | `1` |
| BorderColor | `gblTheme.Border` |
| RadiusTopLeft / TopRight / BottomLeft / BottomRight | `4` |
| PaddingLeft | `12` |

Inside `conPersonOwner`, add three children:

1. **Avatar circle.** Insert → **Icons** → **Circle**:
   `circAvatarOwner`, X=`12`, Y=`(Parent.Height-24)/2`, Width=`24`,
   Height=`24`, Fill=`gblTheme.Maroon`, BorderThickness=`0`.

2. **Avatar initials label.** Insert → **Label**:
   `lblAvatarOwner`, X=`circAvatarOwner.X`, Y=`circAvatarOwner.Y`,
   Width=`circAvatarOwner.Width`, Height=`circAvatarOwner.Height`,
   Text=`selectedUC.OwnerInitials`, Color=`White`, Size=`10`,
   FontWeight=`FontWeight.Bold`, Align=`Align.Center`,
   VerticalAlign=`VerticalAlign.Middle`, PaddingLeft=`0`.

3. **Name + role labels.** Insert → **Label**:
   `lblPersonName`, X=`44`, Y=`6`, Width=`Parent.Width - 56`,
   Height=`16`, Text=`selectedUC.Owner`, Color=`gblTheme.Ink`,
   Size=`13`, FontWeight=`FontWeight.Semibold`.

   Insert → **Label**: `lblPersonRole`, X=`44`,
   Y=`lblPersonName.Y + lblPersonName.Height + 2`,
   Width=`Parent.Width - 56`, Height=`14`,
   Text=`"AI Solution Owner"`, Color=`gblTheme.Ink3`, Size=`11`.

Now duplicate the card 5 times (right-click `conPersonOwner` →
Duplicate). For each duplicate, change the wrapper container's
X/Y to place it in the grid, change the Name (e.g.
`conPersonSponsor`, `conPersonAAILead`, `conPersonTechLead`,
`conPersonDataPartner`, `conPersonFinancePartner`), and edit the
inner `lblPersonName.Text` and `lblPersonRole.Text` to match the
role. For v1, hardcode reasonable strings or use literal `""` if
empty:

| Card | X | Y | Role text | Name text |
|------|---|---|-----------|-----------|
| Owner (1) | `28` | `…+24` | `"AI Solution Owner"` | `selectedUC.Owner` |
| Sponsor (2) | right col | row 1 | `"Executive Sponsor"` | `"Megan Stone, MD"` |
| AAI Lead (3) | left col | row 2 | `"AAI Lead"` | `"Devon Brar"` |
| Tech Lead (4) | right col | row 2 | `"Tech Lead"` | `"Hugh Park"` |
| Data Partner (5) | left col | row 3 | `"Data Partner"` | `"Anika Sharma"` |
| Finance Partner (6) | right col | row 3 | `"Finance Partner"` | `"Tom Whitley"` |

Right column X = `28 + ((Parent.Width - 56 - 24) / 2) + 24`.
Row 2 Y = row 1 Y + `48 + 12`. Row 3 Y = row 2 Y + `48 + 12`.

### Step 39 — Sanity check

Click "Contacts" in the section rail. You should see:

- [ ] Section card titled "Contacts" with maroon underline.
- [ ] Six person tiles in a 2×3 grid. Each tile has a maroon
      avatar circle on the left, a bold name, a gray role
      caption.
- [ ] The first tile shows the use case's actual owner and
      initials (e.g. "Jasmine Lee" / "JL" for UC-0142).
- [ ] Toggle the rail — tiles reflow with the column width.

---

## Part 8 — Section: Value

Three summary tiles at the top, then a Value table (header row +
gallery), then an "Add value entry" button and an edit modal.

### Step 40 — Section container + title

Same pattern as Steps 27–28: Container `conSectionValue` with
`Visible: currentSection = "Value"`, title `"Value"` and underline.

### Step 41 — Three summary tiles

Three identical tiles in a row. Build one and duplicate. Inside
`conSectionValue`, Insert → **Container**:

| Property | Value |
|----------|-------|
| Name | `conTileEstimated` |
| X | `28` |
| Y | `recValueTitleBorder.Y + recValueTitleBorder.Height + 24` |
| Width | `(Parent.Width - 56 - 24) / 3` |
| Height | `72` |
| Fill | `RGBA(250,250,250,1)` |
| BorderThickness | `1` |
| BorderColor | `gblTheme.Border` |
| RadiusTopLeft / TopRight / BottomLeft / BottomRight | `3` |
| PaddingLeft | `14` |
| PaddingTop | `10` |

Inside the tile, two labels:

`lblTileCapEstimated`: X=`14`, Y=`10`, Width=`Parent.Width-28`,
Height=`14`, Text=`"ESTIMATED BENEFIT"`, Size=`10`,
FontWeight=`FontWeight.Bold`, Color=`gblTheme.Ink3`.

`lblTileValEstimated`: X=`14`, Y=`28`, Width=`Parent.Width-28`,
Height=`30`, Text=`"$" & Text(selectedUC.EstimatedValue/1000000, "0.0") & "M"`,
Size=`18`, FontWeight=`FontWeight.Bold`, Color=`gblTheme.Ink`.

Duplicate for the other two tiles:

| Tile | Position | Caption | Value | Value color |
|------|----------|---------|-------|-------------|
| Estimated (1) | X=`28` | `"ESTIMATED BENEFIT"` | `selectedUC.EstimatedValue` | `gblTheme.Ink` |
| Realized YTD (2) | X = tile1.X + tile1.Width + 12 | `"REALIZED · F26 YTD"` | `realizedYTD` | `gblTheme.Maroon` |
| Investment (3) | X = tile2.X + tile2.Width + 12 | `"INVESTMENT SPEND"` | `"$0.4M"` (hardcoded) | `gblTheme.Ink` |

Value formulas:

- Tile 1: `"$" & Text(selectedUC.EstimatedValue/1000000, "0.0") & "M"`
- Tile 2: `"$" & Text(realizedYTD/1000000, "0.0") & "M"`
- Tile 3: `"$0.4M"`

### Step 42 — Value table header row

Below the tiles, build a header row using a Horizontal Container —
same pattern as `srcList` Step 22. Inside `conSectionValue`,
Insert → **Layout** → **Horizontal container**.

| Property | Value |
|----------|-------|
| Name | `conValueHeader` |
| X | `28` |
| Y | `conTileEstimated.Y + conTileEstimated.Height + 24` |
| Width | `Parent.Width - 56` |
| Height | `32` |
| Fill | `RGBA(245,245,245,1)` |
| BorderThickness | `0` |
| LayoutDirection | `LayoutDirection.Horizontal` |
| LayoutGap | `0` |
| LayoutAlignItems | `LayoutAlignItems.Center` |
| LayoutJustifyContent | `LayoutJustifyContent.Start` |
| PaddingLeft | `14` |

Insert 6 Labels inside in this order, with these properties:

| # | Name | Text | FillPortions |
|---|------|------|--------------|
| 1 | `lblValColPeriod`    | `"FY · Q"`     | `120` |
| 2 | `lblValColDriver`    | `"Driver"`     | `220` |
| 3 | `lblValColAmount`    | `"Value"`      | `120` |
| 4 | `lblValColFrequency` | `"Frequency"`  | `120` |
| 5 | `lblValColSignoff`   | `"Signoff"`    | `140` |
| 6 | `lblValColAction`    | `""`           | `90`  |

All shared: Size=`11`, FontWeight=`FontWeight.Semibold`,
Color=`gblTheme.Ink3`, Font=`gblTheme.FontFamily`, PaddingLeft=`0`.

### Step 43 — Value table gallery

Inside `conSectionValue`, Insert → **Gallery** → **Blank vertical**.

| Property | Value |
|----------|-------|
| Name | `galValueRows` |
| X | `28` |
| Y | `conValueHeader.Y + conValueHeader.Height` |
| Width | `Parent.Width - 56` |
| Height | `180` |
| Items | `ucValueRows` |
| TemplateSize | `44` |
| TemplatePadding | `0` |
| BorderThickness | `0` |
| ShowScrollbar | `true` |

### Step 44 — Value row template

Enter the template. Add a Horizontal Container that mirrors the
header:

| Property | Value |
|----------|-------|
| Name | `conValueRow` |
| X | `0` |
| Y | `0` |
| Width | `Parent.TemplateWidth` |
| Height | `Parent.TemplateHeight - 1` |
| Fill | `gblTheme.Surface` |
| BorderThickness | `0` |
| LayoutDirection | `LayoutDirection.Horizontal` |
| LayoutGap | `0` |
| LayoutAlignItems | `LayoutAlignItems.Center` |
| LayoutJustifyContent | `LayoutJustifyContent.Start` |
| PaddingLeft | `14` |

Then in the template (sibling of `conValueRow`), Insert →
**Rectangle**:

| Property | Value |
|----------|-------|
| Name | `recValueRowDivider` |
| X | `0` |
| Y | `Parent.TemplateHeight - 1` |
| Width | `Parent.TemplateWidth` |
| Height | `1` |
| Fill | `gblTheme.Border` |
| BorderThickness | `0` |

Inside `conValueRow`, insert 6 controls in order (matching the
header FillPortions exactly):

| # | Type | Name | Text / spec | FillPortions |
|---|------|------|-------------|--------------|
| 1 | Label | `lblRowPeriod` | `ThisItem.Period` | `120` |
| 2 | Label | `lblRowDriver` | `ThisItem.Driver` | `220` |
| 3 | Label | `lblRowAmount` | (formula below) | `120` |
| 4 | Label | `lblRowFrequency` | `ThisItem.Frequency` | `120` |
| 5 | Container (classic) | `conRowSignoff` | _(wrapper)_ | `140` |
| 6 | Container (classic) | `conRowAction` | _(wrapper)_ | `90` |

Shared label props: Size=`12`, Color=`gblTheme.Ink`,
Font=`gblTheme.FontFamily`, PaddingLeft=`0`. `lblRowAmount`
overrides FontWeight=`FontWeight.Semibold`.

`lblRowAmount.Text`:

```powerfx
If(ThisItem.Amount = 0,
   "— TBD",
   "$" & Text(ThisItem.Amount / 1000000, "0.0") & "M")
```

The two wrapper Containers need just `FillPortions` (from the table)
and `Fill: RGBA(0,0,0,0)`.

### Step 45 — Inlined Signoff pill (inside `conRowSignoff`)

This is the same gallery-container-component restriction as `srcList`
Step 28 — we can't drop a custom pill component inside a container
under a gallery. Inline a Circle + Label.

Inside `conRowSignoff`, Insert → **Icons** → **Circle**:

| Property | Value |
|----------|-------|
| Name | `circSignoffDot` |
| X | `0` |
| Y | `(Parent.Height - Self.Height) / 2` |
| Width | `8` |
| Height | `8` |
| BorderThickness | `0` |
| Fill | (formula below) |

```powerfx
Switch(ThisItem.Signoff,
    "Signed off", gblTheme.Ok,
    "Pending",    gblTheme.Warn,
    "Not started",gblTheme.Ink4,
    gblTheme.Ink4
)
```

Inside `conRowSignoff`, Insert → **Label**:

| Property | Value |
|----------|-------|
| Name | `lblSignoffText` |
| X | `circSignoffDot.X + circSignoffDot.Width + 6` |
| Y | `0` |
| Width | `Parent.Width - Self.X` |
| Height | `Parent.Height` |
| Text | `ThisItem.Signoff` |
| Size | `12` |
| Color | `gblTheme.Ink2` |
| VerticalAlign | `VerticalAlign.Middle` |
| PaddingLeft | `0` |

### Step 46 — Edit button (inside `conRowAction`)

Inside `conRowAction`, Insert → **Button**:

| Property | Value |
|----------|-------|
| Name | `btnEditValue` |
| Text | `"Edit"` |
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
| OnSelect | `Set(editingValueRow, ThisItem); Set(showValueModal, true)` |

Click outside the template to exit.

### Step 47 — "+ Add value entry" button

Inside `conSectionValue` (not the gallery template), Insert → **Button**:

| Property | Value |
|----------|-------|
| Name | `btnAddValue` |
| Text | `"+ Add value entry"` |
| X | `28` |
| Y | `galValueRows.Y + galValueRows.Height + 12` |
| Width | `Parent.Width - 56` |
| Height | `40` |
| Size | `13` |
| FontWeight | `FontWeight.Semibold` |
| Fill | `gblTheme.Surface` |
| HoverFill | `RGBA(245, 230, 233, 1)` |
| Color | `gblTheme.Maroon` |
| BorderColor | `gblTheme.Border` |
| BorderThickness | `1` |
| RadiusTopLeft / TopRight / BottomLeft / BottomRight | `4` |
| OnSelect | `Set(editingValueRow, Defaults(colValueEntries)); Set(showValueModal, true)` |

### Step 48 — Value edit modal (overlay)

> **Place this outside `conSectionValue` — at the root of `srcDetail`.**
> An overlay should cover the action bar and rail too.

On `srcDetail`, Insert → **Container**:

| Property | Value |
|----------|-------|
| Name | `conValueModalScrim` |
| X | `0` |
| Y | `0` |
| Width | `Parent.Width` |
| Height | `Parent.Height` |
| Fill | `RGBA(0, 0, 0, 0.45)` |
| BorderThickness | `0` |
| Visible | `showValueModal` |

Inside `conValueModalScrim`, Insert → **Container** (the white card):

| Property | Value |
|----------|-------|
| Name | `conValueModalCard` |
| X | `(Parent.Width - Self.Width) / 2` |
| Y | `(Parent.Height - Self.Height) / 2` |
| Width | `520` |
| Height | `440` |
| Fill | `gblTheme.Surface` |
| BorderColor | `gblTheme.Border` |
| BorderThickness | `1` |
| RadiusTopLeft / TopRight / BottomLeft / BottomRight | `6` |
| PaddingLeft | `28` |
| PaddingTop | `24` |
| PaddingRight | `28` |

Inside `conValueModalCard`, build:

1. **Title** `lblModalTitle`: Text=`If(IsBlank(editingValueRow.UCID), "Add value entry", "Edit value entry")`, X=`28`, Y=`24`, Width=`Parent.Width-56`, Size=`16`, FontWeight=`FontWeight.Bold`, Color=`gblTheme.Ink`.

2. **Five fields** (Period, Driver, Amount, Frequency, Signoff). Use
   the same caption + input pattern as Step 33. Stack them vertically
   starting at Y=`60` with 64px between rows. Suggested controls:

   | Field | Control | Items / Default |
   |-------|---------|-----------------|
   | Period | TextInput `txtModalPeriod` | `editingValueRow.Period` |
   | Driver | Dropdown `ddModalDriver` | `colBenefitType`, Default: `editingValueRow.Driver` |
   | Amount (dollars) | TextInput `txtModalAmount` | `Text(editingValueRow.Amount)` |
   | Frequency | Dropdown `ddModalFreq` | `colRefreshFreq`, Default: `editingValueRow.Frequency` |
   | Signoff | Dropdown `ddModalSignoff` | `["Not started", "Pending", "Signed off"]`, Default: `editingValueRow.Signoff` |

3. **Cancel button** `btnModalCancel`: X=`Parent.Width-260`,
   Y=`Parent.Height-60`, Width=`100`, Height=`36`, secondary style
   (white fill, maroon text + border), OnSelect:
   `Set(showValueModal, false)`.

4. **Save button** `btnModalSave`: X=`btnModalCancel.X +
   btnModalCancel.Width + 12`, Y=`btnModalCancel.Y`, Width=`120`,
   Height=`36`, primary style (maroon fill, white text), OnSelect:

   ```powerfx
   If(IsBlank(editingValueRow.UCID),
       // New row
       Collect(colValueEntries,
           {
               UCID: selectedUC.UCID,
               Period: txtModalPeriod.Text,
               Driver: ddModalDriver.Selected.Value,
               Amount: Value(txtModalAmount.Text),
               Frequency: ddModalFreq.Selected.Value,
               Signoff: ddModalSignoff.Selected.Value
           }
       ),
       // Update existing
       Patch(colValueEntries, editingValueRow,
           {
               Period: txtModalPeriod.Text,
               Driver: ddModalDriver.Selected.Value,
               Amount: Value(txtModalAmount.Text),
               Frequency: ddModalFreq.Selected.Value,
               Signoff: ddModalSignoff.Selected.Value
           }
       )
   );
   Set(showValueModal, false)
   ```

### Step 49 — Sanity check

Click "Value" in the section rail. You should see:

- [ ] Three summary tiles. Estimated and Realized show formatted
      dollar values; Realized is maroon.
- [ ] Below tiles: header row (light gray, 6 columns) and 3 value
      rows for UC-0142 with colored Signoff dots (green / amber /
      gray).
- [ ] Amount column: rows with `Amount=0` show `"— TBD"`, others
      show `"$0.9M"` etc., bold.
- [ ] "+ Add value entry" button below the gallery.
- [ ] Click "+ Add value entry" → modal overlays the whole screen,
      title is "Add value entry", all inputs empty.
- [ ] Fill out the modal, click Save → modal closes, new row appears
      in the gallery.
- [ ] Click "Edit" on an existing row → modal opens with title
      "Edit value entry", inputs pre-filled. Save updates the row.
- [ ] Click Cancel → modal closes without changes.

---

## Part 9 — Remaining sections (Funds, Gov, Tech, Updates)

These three sections share the same panel structure as Info (Container +
title + content). For brevity each is summarized; build them
following the same patterns as Parts 6–8.

### Step 50 — Section: Funds

Container `conSectionFunds` with `Visible: currentSection = "Funds"`,
title `"Funds"` + underline. Then a 2-column form with 6 fields,
laid out exactly like Step 34 (caption above input, half-column
widths, same X/Y math). Fields:

| # | Position | Caption | Control | Items / Default |
|---|----------|---------|---------|-----------------|
| 1 | R1 L | `"Funding Available"` | Dropdown `ddFunding` | `["Yes — Business Case Funded", "Partial", "No"]` |
| 2 | R1 R | `"Tracked by Finance Partner"` | Dropdown `ddTrackedFin` | `["Yes", "No"]` |
| 3 | R2 L | `"Estimated Monetary Benefit"` | TextInput `txtEstBenefit` | `"$" & Text(selectedUC.EstimatedValue/1000000, "0.0") & "M"` |
| 4 | R2 R | `"Estimated Benefit Type"` | Dropdown `ddBenefitType` | `colBenefitType` |
| 5 | R3 L | `"Actualized Monetary Benefit"` | TextInput `txtActBenefit` | `"$" & Text(selectedUC.RealizedValue/1000000, "0.0") & "M"` |
| 6 | R3 R | `"Estimated Other Benefit"` | TextInput `txtOtherBenefit` | `""` |

OnChange handlers are optional in v1 (only EstimatedValue /
RealizedValue map to existing fields). Add Patch handlers later when
the data model has Funding / TrackedByFinance columns.

### Step 51 — Section: Governance (checklist gallery)

Container `conSectionGov` with `Visible: currentSection = "Gov"`,
title `"Governance"`. Add a right-aligned progress label + bar at the
top of the section.

**Progress bar** (two stacked rectangles, right-aligned next to the
title row):

| Control | Name | X | Y | Width | Height | Fill |
|---------|------|---|---|-------|--------|------|
| Label | `lblGovProgress` | `Parent.Width - 200` | `28` | `100` | `20` | Text: `Text(govDoneCount) & " of " & Text(govTotalCount) & " complete"`, Size: `12`, Color: `gblTheme.Ink3`, Align: `Align.Right` |
| Rectangle (bg) | `recGovBarBg` | `Parent.Width - 92` | `30` | `80` | `6` | `RGBA(238,238,238,1)`, Radius `3` |
| Rectangle (fill) | `recGovBarFill` | `recGovBarBg.X` | `recGovBarBg.Y` | `80 * govProgressPct` | `6` | `gblTheme.Maroon`, Radius `3` |

**Checklist gallery** `galGovernance`:

| Property | Value |
|----------|-------|
| X | `28` |
| Y | `recGovTitleBorder.Y + recGovTitleBorder.Height + 24` |
| Width | `Parent.Width - 56` |
| Height | `Parent.Height - Self.Y - 24` |
| Items | `ucGovRows` |
| TemplateSize | `64` |
| BorderThickness | `0` |

Template (inside the gallery):

1. **Checkbox** (Rectangle): `recCheckBox`, X=`14`, Y=`(Parent.TemplateHeight-Self.Height)/2`, Width=`20`, Height=`20`, Radius `3`, Fill=`If(ThisItem.Done, gblTheme.Ok, gblTheme.Surface)`, BorderColor=`If(ThisItem.Done, gblTheme.Ok, gblTheme.BorderStrong)`, BorderThickness=`1`. Plus a `lblCheckMark` label inside (or a sibling) — Text=`"✓"`, Color=`White`, Visible=`ThisItem.Done`, centered in the same box.

2. **Name label** `lblCheckName`: X=`46`, Y=`12`, Width=`Parent.TemplateWidth-280`, Height=`20`, Text=`ThisItem.Name`, Size=`13`, FontWeight=`FontWeight.Semibold`, Color=`gblTheme.Ink`.

3. **Meta label** `lblCheckMeta`: X=`46`, Y=`lblCheckName.Y + lblCheckName.Height + 2`, Width=`Parent.TemplateWidth-280`, Height=`18`, Text=`ThisItem.Meta`, Size=`11`, Color=`gblTheme.Ink3`.

4. **Action button** `btnToggleDone`: X=`Parent.TemplateWidth-150`, Y=`(Parent.TemplateHeight-32)/2`, Width=`130`, Height=`32`. Style and text depend on done state:
   - Text: `If(ThisItem.Done, "Completed ✓", "Mark complete")`
   - Fill: `If(ThisItem.Done, gblTheme.Ok, gblTheme.Surface)`
   - Color: `If(ThisItem.Done, White, gblTheme.Maroon)`
   - BorderColor: `If(ThisItem.Done, gblTheme.Ok, gblTheme.Maroon)`
   - BorderThickness: `1`, Radius `4`, FontWeight=`FontWeight.Semibold`, Size=`12`
   - OnSelect: `Patch(colGovernance, ThisItem, { Done: !ThisItem.Done })`

5. **Bottom border** (sibling of the row, in the gallery template): Rectangle `recGovRowBorder`, X=`0`, Y=`Parent.TemplateHeight-1`, Width=`Parent.TemplateWidth`, Height=`1`, Fill=`gblTheme.Border`.

### Step 52 — Section: Technical Review

Container `conSectionTech` with `Visible: currentSection = "Tech"`,
title `"Technical Review"`. The checklist gallery is identical to
Governance but bound to `ucTechRows` (and Patch hits `colTechReview`).
Copy `galGovernance` and rename to `galTechReview` to save time.

Below the gallery, add a "Performance Metrics" multi-line text input:

| Property | Value |
|----------|-------|
| Name | `lblCapMetrics` | (caption) Text=`"Performance Metrics"`, Size=`12`, FontWeight=`FontWeight.Semibold`, Color=`gblTheme.Ink2` |
| Name | `txtPerfMetrics` | X=`28`, Y=`lblCapMetrics.Y+20`, Width=`Parent.Width-56`, Height=`120`, Mode=`TextMode.MultiLine`, Default=`""`, BorderColor=`gblTheme.Border` |

### Step 53 — Section: Monthly Update

Container `conSectionUpdates` with `Visible: currentSection = "Updates"`,
title `"Monthly Update"`. Children:

1. **Caption** `lblCapUpdate`: Text=`"Update for " & Text(Now(), "mmmm yyyy")`, Size=`12`, FontWeight=`FontWeight.Semibold`.

2. **Update text input** `txtUpdate`: X=`28`, Y=`lblCapUpdate.Y+20`, Width=`Parent.Width-56`, Height=`140`, Mode=`TextMode.MultiLine`, Default=`"Model is production-ready in shadow mode. Awaiting MRM sign-off…"`, BorderColor=`gblTheme.Border`.

3. **Hint label** below: Text=`"This update appears in the PDF report and the monthly Teams digest."`, Size=`11`, Color=`gblTheme.Ink3`.

4. **Toggle row.** Insert → **Toggle** `tglNotify`, with `Default=true`. Add a label next to it: `"Notify Executive Sponsor and AAI leadership via MS Teams when submitted"`. Toggle's `FillSelected: gblTheme.Maroon`.

### Step 54 — Sanity check (sections 9–13)

For each of Funds, Gov, Tech, Updates, click the section in the rail
and confirm:

- [ ] Section card appears with its title and maroon underline.
- [ ] **Funds:** 6 inputs in a 2×3 grid.
- [ ] **Gov:** 5 checklist rows for UC-0142; progress bar reads
      "3 of 5 complete" with the maroon bar filling 60%. Clicking
      "Mark complete" on a Done=false row flips it; progress bar
      and section-rail badge both update.
- [ ] **Tech:** 4 checklist rows; same toggle behavior as Gov.
      "Performance Metrics" multi-line input below.
- [ ] **Updates:** Multi-line text input, the hint label, and the
      Toggle (default ON) labeled "Notify Executive Sponsor…".

---

## Part 10 — Submit zone (pinned bottom)

The 60-px white strip at the bottom of `conDetailLayout`. Mirrors the
action bar's Save Draft + Submit buttons so the user doesn't have to
scroll back up.

### Step 55 — Top border

Inside `conSubmitZone`, Insert → **Rectangle**:

| Property | Value |
|----------|-------|
| Name | `recSubmitZoneBorder` |
| X | `0` |
| Y | `0` |
| Width | `Parent.Width` |
| Height | `1` |
| Fill | `gblTheme.Border` |
| BorderThickness | `0` |

### Step 56 — Helper text label (left)

Inside `conSubmitZone`, Insert → **Label**:

| Property | Value |
|----------|-------|
| Name | `lblSubmitHint` |
| Text | `"Save Draft keeps your changes without notifying anyone. Submit Assessment publishes the latest values and triggers Power Automate flows."` |
| X | `28` |
| Y | `(Parent.Height - Self.Height) / 2` |
| Width | `Parent.Width - 320` |
| Height | `36` |
| Size | `11` |
| Color | `gblTheme.Ink3` |
| Font | `gblTheme.FontFamily` |

### Step 57 — Save Draft (secondary)

Inside `conSubmitZone`, Insert → **Button**, named `btnSaveDraft2`,
same style as `btnSaveDraft` (Step 9), X=`Parent.Width - 260`,
Y=`(Parent.Height - Self.Height) / 2`, OnSelect identical to Step 9.

### Step 58 — Submit (primary)

Inside `conSubmitZone`, Insert → **Button**, named `btnSubmit2`, same
style as `btnSubmit` (Step 10), X=`btnSaveDraft2.X + btnSaveDraft2.Width + 8`,
Y=`btnSaveDraft2.Y`, OnSelect identical to Step 10.

### Step 59 — Sanity check

Preview:

- [ ] At the bottom of the detail layout: a 60-px white strip with a
      1px top border.
- [ ] Left: small gray helper text explaining Save Draft vs Submit.
- [ ] Right: Save Draft (outlined maroon) + Submit Assessment
      (solid maroon).
- [ ] Both buttons fire the same notifications as the action-bar
      versions.
- [ ] Strip stays at the bottom regardless of which section is
      visible.

---

## Part 11 — Full-screen sanity check

Click any row in `srcList` to land on `srcDetail`. Walk through:

- [ ] Header (top maroon) + action bar (white, 56) + detail layout
      (gray bg below).
- [ ] Action bar: Back → returns to `srcList`. Breadcrumb shows the
      row's UCID and name. Save Draft notifies. Submit fires
      success notification.
- [ ] Internal section rail (220 wide, white, on the left of the
      detail layout). Head shows UCID · FY, name, and a colored
      status pill. Below: 7 section links; "Use Case Info" is
      highlighted on entry.
- [ ] **Info:** All 12 fields render. Editing Name updates the
      rail head's name label and the breadcrumb. Changing Status
      advances the horizontal stepper.
- [ ] **Contacts:** 6 person cards in a 2×3 grid.
- [ ] **Value:** 3 summary tiles, header row, 3 value rows. The
      Edit button opens the modal pre-filled. The "+ Add" button
      opens it empty. Save persists; Cancel closes.
- [ ] **Funds:** 6-input form. Defaults match the use case.
- [ ] **Governance:** 5-item checklist with progress bar and badge
      counts. Clicking "Mark complete" updates everything reactively.
- [ ] **Technical Review:** Same checklist pattern for 4 items,
      followed by the Performance Metrics text input.
- [ ] **Monthly Update:** Multi-line input, hint, toggle.
- [ ] Submit zone at the bottom shows the helper text and mirror
      buttons.
- [ ] Toggle the left rail (hamburger). Action bar, detail layout,
      and submit zone all reflow. Internal section rail stays 220
      wide; content area resizes.
- [ ] Bouncing test: from the home screen, navigate to `srcDetail`
      directly (e.g. via a test button that runs
      `Set(selectedUC, Blank()); Navigate(srcDetail)`). You should
      land back on `srcList` because of the OnVisible guard.

If all 13 boxes pass, `srcDetail` is done. Move on to `srcNew`
(Screen 4 in `02-build-guide.md`).

---

## Quick reference — things that go wrong

| Symptom | Cause | Fix |
|---------|-------|-----|
| Screen errors with red squiggles on `selectedUC.Name`, `.UCID`, etc. on first load | `selectedUC` wasn't initialized as a typed record in `App.OnStart` | In `01-app-setup.md` section 2, ensure `selectedUC` is set to a literal record with **all 16 fields**, not `Blank()`. |
| Stepper circles never highlight | `selectedUC.Status` doesn't match a code in `lblCurrentStatusIdx`'s LookUp table | Status codes must be exactly: `Rationale`, `DataPrep`, `Development`, `Testing`, `Deployment`, `Monitoring`. Check `colUseCases` data. |
| Status dropdown changes don't update the stepper | Dropdown OnChange writes the Label back to `Status` instead of the Code | Use `LookUp(colStatus, Label = Self.Selected.Value, Code)` — see Step 34 row 2. |
| Sections all visible at once | A section's `Visible` formula is missing or evaluates to true unconditionally | Each `conSection*.Visible` must be `currentSection = "<key>"`. Check Step 27, 36, 40, 50, 51, 52, 53. |
| Clicking section nav does nothing | Gallery `OnSelect` is on a template control instead of the gallery itself | Click `galSectionNav` (not a child of it), set its `OnSelect` to `Set(currentSection, ThisItem.Key)`. |
| Section rail head doesn't update when navigating between use cases | `lblRailName`, etc. reference a hardcoded value | Their `Text` must reference `selectedUC.<Field>`, not a literal. |
| Custom component (`cmpStatusPill`) won't insert into `conRowSignoff` | Power Apps blocks components inside containers nested in gallery templates | Use the inline Circle + Label (Step 45). The component still works in `conRailHead` (Step 20) because that's not under a gallery. |
| Modal doesn't cover the whole screen | `conValueModalScrim` was created inside `conSectionValue` instead of at the screen root | Cut it, paste at the `srcDetail` root level. |
| Modal Save creates a row but with `UCID = ""` | New-row branch missing `UCID: selectedUC.UCID` in the `Collect` | See Step 48 — the first branch of the `If(IsBlank(editingValueRow.UCID), …)` must include `UCID: selectedUC.UCID`. |
| Progress bar fill is 0 width on first load | `govTotalCount` is 0 (no governance rows for this UCID), so `govProgressPct` is 0 | This is correct behavior. Confirm `colGovernance` has rows with `UCID = selectedUC.UCID`. |
| OnChange Patch saves but other sections still show old values | Missing the `Set(selectedUC, LookUp(…))` line after Patch | Every form input's OnChange must Patch, then re-Set `selectedUC` so dependent labels refresh. |
| Stepper labels overlap because columns are too narrow | Gallery `TemplateSize` (horizontal galleries = row width) is too small | Should be `Self.Width / 6`. If the gallery's Width is wrong, fix that first — should be `Parent.Width - 56`. |
| Two-column form fields overflow the right edge when rail expands | Field Width is hardcoded instead of using the half-col formula | Width must be `(Parent.Width - 56 - 24) / 2`. Same for right-column X anchor. |
| Action bar overlaps the detail layout below | `conDetailLayout.Y` is `52` instead of `108` | Y must be `52 + 56 = 108` to clear both the header and the action bar. |
| Toggle the rail and the action bar / detail layout don't shift | X formula is hardcoded instead of `If(sideCollapsed, 64, 220)` | Both `conActionBar.X` and `conDetailLayout.X` must use the formula. Width must subtract the same offset. |
| Stepper line crosses through the circles instead of behind them | Tree order: `galStepper` is below `recStepTrack` in the tree | In the tree, drag `galStepper` to be **above** `recStepTrack` (later in tree = on top visually). |
| `Navigate(srcList, …)` errors from Back button | `srcList` doesn't exist as a named screen | Build `srcList` first (`07-scrlist-guide.md`) or create a blank screen named exactly `srcList`. |
| `Now()` makes `lblSaveNote` jitter | `Now()` re-evaluates on any reactive change | For v1 this is acceptable. For v2, replace with `Set(lastSavedAt, Now())` in `btnSaveDraft.OnSelect` and reference `lastSavedAt` here. |
