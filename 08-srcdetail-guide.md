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
    │   ├── btnSecInfo + recSecAccentInfo
    │   ├── btnSecContacts + recSecAccentContacts
    │   ├── btnSecValue + recSecAccentValue + lblSecBadgeValue
    │   ├── btnSecFunds + recSecAccentFunds
    │   ├── btnSecGov + recSecAccentGov + lblSecBadgeGov
    │   ├── btnSecTech + recSecAccentTech
    │   └── btnSecUpdates + recSecAccentUpdates
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
variable: clicking a nav button sets `currentSection`, and each
section's `Visible` formula compares against it. No tab control, no
animation, no DOM manipulation — Power Apps native.

The nav rows are individual Buttons, not a gallery — same buttons-only
pattern as `06-left-rail-buttons-guide.md`. Galleries don't expose
per-row HoverFill, don't reliably show the hand cursor, and store-
then-read of style properties via `ThisItem.…` is flaky.

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

## Part 5 — Section nav (buttons, no gallery)

Seven nav rows for switching sections. We build them as individual
Buttons instead of a gallery — same reasoning as
`06-left-rail-buttons-guide.md` for the main left rail. Galleries
don't expose per-row `HoverFill`, don't reliably show the hand cursor
on click, and store-then-read of style properties via `ThisItem.…` is
flaky. Seven buttons is small enough to maintain by hand.

For each row you'll add **two controls** as siblings of `conRailHead`
(direct children of `conSectionRail`):

1. A `Button` — the click target. Native `HoverFill`, hand cursor,
   active-state Fill/Color formulas keyed off `currentSection`.
2. A 3-px `Rectangle` — the left accent stripe, maroon when active.

Two of the rows (Value and Governance) get a third sibling: a small
count `Label` overlaid on the right.

> All controls below go inside **`conSectionRail`** (siblings of
> `conRailHead`). Do not put them inside `conRailHead`.

The rows stack starting at `Y = conRailHead.Y + conRailHead.Height`
(= 100) with `Height = 40`. So row N sits at
`Y = 100 + (N - 1) * 40`.

| Row | Section key | Label | Y | Badge formula |
|-----|-------------|-------|---|---------------|
| 1 | `"Info"` | "Use Case Info" | `100` | none |
| 2 | `"Contacts"` | "Contacts" | `140` | none |
| 3 | `"Value"` | "Value" | `180` | `Text(CountRows(ucValueRows))` |
| 4 | `"Funds"` | "Funds" | `220` | none |
| 5 | `"Gov"` | "Governance" | `260` | `Text(govDoneCount)` |
| 6 | `"Tech"` | "Technical Review" | `300` | none |
| 7 | `"Updates"` | "Monthly Update" | `340` | none |

### Step 22 — Build the first row (Info) completely

You'll build the Info row in full, get it working, then copy/paste
six times and tweak the per-row fields (Y, Text, Key, OnSelect).

**Button.** Inside `conSectionRail`, Insert → **Button**.

| Property | Value |
|----------|-------|
| Name | `btnSecInfo` |
| X | `0` |
| Y | `100` |
| Width | `Parent.Width - 1` |
| Height | `40` |
| Text | `"Use Case Info"` |
| Align | `Align.Left` |
| PaddingLeft | `20` |
| Size | `13` |
| BorderThickness | `0` |
| FocusedBorderThickness | `0` |
| RadiusTopLeft / TopRight / BottomLeft / BottomRight | `0` |
| OnSelect | `Set(currentSection, "Info")` |

Width subtracts 1 so it doesn't sit on top of the rail's right
border (`recSectionRailBorder` from Step 13).

**Fill** — transparent normally, light maroon tint when active:

```powerfx
If(
    currentSection = "Info",
    RGBA(122, 26, 46, 0.10),
    RGBA(0, 0, 0, 0)
)
```

**HoverFill** — slightly darker on hover, regardless of active state:

```powerfx
RGBA(122, 26, 46, 0.18)
```

**PressedFill** — darker still on click:

```powerfx
RGBA(122, 26, 46, 0.28)
```

**Color** — maroon when active, ink2 otherwise:

```powerfx
If(
    currentSection = "Info",
    gblTheme.Maroon,
    gblTheme.Ink2
)
```

**HoverColor / PressedColor** — maroon (so hovered rows feel live):

```powerfx
gblTheme.Maroon
```

**FontWeight** — bold when active:

```powerfx
If(
    currentSection = "Info",
    FontWeight.Bold,
    FontWeight.Semibold
)
```

**Accent stripe.** Inside `conSectionRail` (not inside the button —
sibling), Insert → **Rectangle**:

| Property | Value |
|----------|-------|
| Name | `recSecAccentInfo` |
| X | `0` |
| Y | `btnSecInfo.Y` |
| Width | `3` |
| Height | `btnSecInfo.Height` |
| BorderThickness | `0` |
| DisplayMode | `DisplayMode.Disabled` |
| Fill | `If(currentSection = "Info", gblTheme.Maroon, RGBA(0,0,0,0))` |

`DisplayMode.Disabled` stops the rectangle from intercepting clicks on
the leftmost 3 px of the button.

### Step 23 — Sanity check the Info row

Click a row in `srcList` to land here. Confirm:

- [ ] "Use Case Info" row visible at Y=100, full rail width.
- [ ] Because `currentSection` defaults to `"Info"` (set in OnVisible
      Step 3), the row is tinted faintly maroon and the 3-px left
      stripe is maroon. Text is bold maroon.
- [ ] Hover the row — the tint darkens; cursor turns to a hand.
- [ ] Click the row — `currentSection` doesn't change (still `"Info"`)
      so visually nothing happens, but the press tint flashes.

Don't move on until this works. The next 6 rows are copies.

### Step 24 — Copy the Info row 6 times

Select **both** `btnSecInfo` and `recSecAccentInfo` in the tree
(Ctrl+click). Copy (Ctrl+C). Paste (Ctrl+V) inside `conSectionRail`.
Studio names the duplicates `btnSecInfo_1` and `recSecAccentInfo_1`.
Paste five more times — you'll end up with `_1` through `_6`.

For each pair, rename and reconfigure per the table below. Every
field references `currentSection = "<Key>"` — replace `"Info"` with
the row's key in **all four** formulas (Fill, Color, FontWeight,
accent Fill).

> **Tip:** to bulk-edit a formula, double-click the property in the
> right panel, swap the literal string, press Enter. Saves time on 6
> identical rows.

| Pair | Button name | Accent name | Y | Text | OnSelect | Active-key string |
|------|-------------|-------------|---|------|----------|-------------------|
| 2 | `btnSecContacts` | `recSecAccentContacts` | `140` | `"Contacts"` | `Set(currentSection, "Contacts")` | `"Contacts"` |
| 3 | `btnSecValue` | `recSecAccentValue` | `180` | `"Value"` | `Set(currentSection, "Value")` | `"Value"` |
| 4 | `btnSecFunds` | `recSecAccentFunds` | `220` | `"Funds"` | `Set(currentSection, "Funds")` | `"Funds"` |
| 5 | `btnSecGov` | `recSecAccentGov` | `260` | `"Governance"` | `Set(currentSection, "Gov")` | `"Gov"` |
| 6 | `btnSecTech` | `recSecAccentTech` | `300` | `"Technical Review"` | `Set(currentSection, "Tech")` | `"Tech"` |
| 7 | `btnSecUpdates` | `recSecAccentUpdates` | `340` | `"Monthly Update"` | `Set(currentSection, "Updates")` | `"Updates"` |

For each button, update:
- `Fill` — swap `"Info"` for the row's active-key string.
- `Color` — same.
- `FontWeight` — same.
- The accent rectangle's `Fill` — same.

Notice the Gov row's section key is `"Gov"`, not `"Governance"` — the
key has to match the `Visible` formula on `conSectionGov` (Step 51).

### Step 25 — Add badges to Value and Gov rows

The Value row should show how many value entries exist
(`CountRows(ucValueRows)`) and Gov should show the done count
(`govDoneCount`). Add a small count label on top of each button.

For the **Value** badge, inside `conSectionRail`, Insert → **Label**:

| Property | Value |
|----------|-------|
| Name | `lblSecBadgeValue` |
| Text | `Text(CountRows(ucValueRows))` |
| Visible | `CountRows(ucValueRows) > 0` |
| X | `btnSecValue.X + btnSecValue.Width - 38` |
| Y | `btnSecValue.Y + (btnSecValue.Height - Self.Height) / 2` |
| Width | `28` |
| Height | `20` |
| Size | `10` |
| FontWeight | `FontWeight.Semibold` |
| Align | `Align.Center` |
| VerticalAlign | `VerticalAlign.Middle` |
| PaddingLeft | `0` |
| PaddingRight | `0` |
| BorderThickness | `0` |
| DisplayMode | `DisplayMode.Disabled` |
| RadiusTopLeft / TopRight / BottomLeft / BottomRight | `10` |
| Color | `If(currentSection = "Value", White, gblTheme.Ink3)` |
| Fill | `If(currentSection = "Value", gblTheme.Maroon, gblTheme.Border)` |

`DisplayMode.Disabled` keeps the badge from blocking clicks on the
button underneath.

For the **Gov** badge, repeat: name `lblSecBadgeGov`,
Text=`Text(govDoneCount)`, Visible=`govTotalCount > 0`, X/Y anchored
to `btnSecGov`, active-key `"Gov"` in the Color and Fill formulas.

> Prefer "3/5" over plain "3" for Gov? Change Text to
> `Text(govDoneCount) & "/" & Text(govTotalCount)` and widen the
> label to `36` so two-digit pairs fit. Same idea for any other row
> if you add more sections later.

### Step 26 — Sanity check the whole nav

Press F5 → click any row in `srcList` to land on Detail. Confirm:

- [ ] Seven nav rows visible in the internal rail: Use Case Info,
      Contacts, Value, Funds, Governance, Technical Review, Monthly
      Update.
- [ ] "Use Case Info" is tinted maroon with a 3-px maroon left
      stripe and bold maroon text (because `currentSection = "Info"`
      from OnVisible).
- [ ] Hover any row — background darkens; cursor turns into a hand.
- [ ] Click "Governance" — its row tints + accent + bold text
      activates; Info's tint and accent clear.
- [ ] Click back to "Use Case Info" — Info activates again.
- [ ] Value row shows a small `3` badge on the right
      (`CountRows(ucValueRows) = 3` for UC-0142). Gov row shows `3`
      too (`govDoneCount = 3` for UC-0142).
- [ ] Clicking the badge area still navigates because the badge has
      `DisplayMode.Disabled` and forwards clicks to the button.
- [ ] The content area on the right is still empty at this stage;
      you'll add the section panels in Parts 6–10. For now, the
      cards visibility toggles invisibly — selection state is
      visible only in the nav.

---

## Part 6 — Section: Use Case Info (form panel)

The biggest section. Header, horizontal status stepper, then a
two-column **Edit form** bound to `colUseCases` with a Save button.

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
| Text | `LookUp(colStatus, Code = selectedUC.Status, Order)` |
| X | `0` |
| Y | `0` |
| Width | `1` |
| Height | `1` |
| Visible | `false` |

Setting Visible=false hides it but the Text formula still evaluates
and is reachable as `lblCurrentStatusIdx.Text` from sibling controls.
Because it reads the `Order` field from `colStatus` (set in
App.OnStart, see `01-app-setup.md`), there is no separate copy of the
status list to keep in sync — all seven statuses, including
**Decommissioning**, flow from the one collection.

> **On Dataverse?** `selectedUC.Status` (a text code) becomes
> `selectedUC.'Project Status'` (an option set), and the codes here don't
> match the Dataverse choice labels — so this `Text` formula must change.
> See [`13-status-stepper-dataverse-fix.md`](13-status-stepper-dataverse-fix.md).

### Step 30 — Status stepper — track line

Inside `conSectionInfo`, Insert → **Rectangle** (this is the inactive
track behind all 7 circles). Its endpoints are derived from where the
gallery (Step 31) actually places the first and last circle centers,
so the line always lines up no matter how many statuses `colStatus`
holds:

| Property | Value |
|----------|-------|
| Name | `recStepTrack` |
| X | `galStepper.X + galStepper.Width / (CountRows(colStatus) * 2)` |
| Y | `recInfoTitleBorder.Y + recInfoTitleBorder.Height + 36` |
| Width | `galStepper.Width * (CountRows(colStatus) - 1) / CountRows(colStatus)` |
| Height | `2` |
| Fill | `gblTheme.BorderStrong` |
| BorderThickness | `0` |

Why these formulas: the gallery lays out `CountRows(colStatus)` equal
cells, each `galStepper.Width / CountRows(colStatus)` wide, and Step 32
centers each circle in its cell. So the **first** circle's center is
half a cell in from the gallery's left edge — that's the `/ 2` in the
`X` formula — and the distance from the first to the last center is
`(CountRows - 1)` whole cells, which is the `Width`. The track now
starts and ends exactly on a circle center.

> Forward reference: `recStepTrack.X` and `.Width` point at
> `galStepper`, which you create in Step 31. Power Apps resolves
> references by name regardless of tree/creation order, and there's no
> cycle here (the gallery only reads `recStepTrack.Y`, a different
> property), so the formulas turn valid the moment `galStepper` exists.

Then Insert → **Rectangle** for the filled portion:

| Property | Value |
|----------|-------|
| Name | `recStepTrackFill` |
| X | `recStepTrack.X` |
| Y | `recStepTrack.Y` |
| Width | `recStepTrack.Width * (Value(lblCurrentStatusIdx.Text) - 1) / (CountRows(colStatus) - 1)` |
| Height | `2` |
| Fill | `gblTheme.Maroon` |
| BorderThickness | `0` |

`(currentIdx - 1) / (CountRows(colStatus) - 1)` is the fraction of the
track to fill — 0 at Rationale (step 1), 1.0 at Decommissioning
(step 7). Dividing by `CountRows - 1` keeps it correct if the status
list grows or shrinks.

### Step 31 — Status stepper — horizontal gallery

Inside `conSectionInfo`, Insert → **Gallery** → **Blank horizontal**.

| Property | Value |
|----------|-------|
| Name | `galStepper` |
| X | `28` |
| Y | `recStepTrack.Y - 24` |
| Width | `Parent.Width - 56` |
| Height | `64` |
| TemplateSize | `Self.Width / CountRows(colStatus)` |
| TemplatePadding | `0` |
| ShowScrollbar | `false` |
| BorderThickness | `0` |
| Items | `colStatus` |

Set **Items** to the collection directly:

```powerfx
colStatus
```

`colStatus` (built in App.OnStart) already carries `Order`, `Code`,
and `Label` for all seven statuses — Rationale through
**Decommissioning** — so there's no second copy of the list to
maintain. The template in Step 32 reads `ThisItem.Order` and
`ThisItem.Label` from it.

`TemplateSize` for a horizontal gallery is the **width** of each row,
not the height. Dividing the gallery width by `CountRows(colStatus)`
gives seven equal cells (and stays correct if the list changes).

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
| Y | `16` |
| Width | `18` |
| Height | `18` |
| BorderThickness | `2` |
| BorderColor | `If(ThisItem.Order <= Value(lblCurrentStatusIdx.Text), gblTheme.Maroon, gblTheme.BorderStrong)` |
| Fill | `If(ThisItem.Order <= Value(lblCurrentStatusIdx.Text), gblTheme.Maroon, gblTheme.Surface)` |

> **Vertical alignment:** `circStep.Y = 16` is what puts the circle's
> center on the track line. The gallery sits at `recStepTrack.Y - 24`,
> so the circle's center lands at `-24 + 16 + (18 / 2) = recStepTrack.Y
> + 1` — the exact vertical center of the 2 px track. If you change the
> gallery's `Y` offset or the circle's `Height`, re-balance this so
> `galStepper.Y_offset + circStep.Y + circStep.Height / 2` equals
> `recStepTrack.Height / 2` (= 1).

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
| Color | `If(ThisItem.Order <= Value(lblCurrentStatusIdx.Text), gblTheme.Maroon, gblTheme.Ink3)` |
| FontWeight | `If(ThisItem.Order = Value(lblCurrentStatusIdx.Text), FontWeight.Bold, FontWeight.Normal)` |
| Font | `gblTheme.FontFamily` |

Click outside to exit the template.

### Step 33 — Form fields — Edit form (insert & bind)

We're using a Power Apps **Edit form** instead of hand-placed control
pairs. The form auto-generates one data card per field, gives us
built-in Required/validation and an explicit Save, and reflows its two
columns responsively when the rail toggles.

> **On Dataverse?** Rebind this form (and the rest of `srcDetail`) to the
> `Projects` table — the Status/Type cards become real choice combo boxes
> (no more `colStatus` code/label `LookUp`), and Contacts/Funds/Gov/Tech
> become their own Projects-bound forms. See
> [`14-srcdetail-dataverse-guide.md`](14-srcdetail-dataverse-guide.md).

> **Behaviour change vs. live-patch:** with a form, edits live inside
> the form until you press **Save** (`SubmitForm`). So the rail-head
> pill and the status stepper update *on save*, not on every
> keystroke. We wire `frmInfo.OnSuccess` to refresh `selectedUC` so
> they update the instant the save completes.

If you already built the old manual controls (`lblFormStart`, every
`lblCap…`, `txt…`, `dd…`, `dp…`), **delete them now.** Keep the section
container, the title + underline, and the entire status stepper.

Inside `conSectionInfo`, Insert → **Edit form**:

| Property | Value |
|----------|-------|
| Name | `frmInfo` |
| DataSource | `colUseCases` |
| Item | `selectedUC` |
| DefaultMode | `FormMode.Edit` |
| X | `28` |
| Y | `galStepper.Y + galStepper.Height + 24` |
| Width | `Parent.Width - 56` |
| Height | `560` |
| Columns | `2` |
| BorderColor | `gblTheme.Border` |

`Columns = 2` with **Snap to columns** on gives the two-column grid:
cards flow left→right and wrap to the next row. `Height` is fixed (the
form does not auto-grow) — if cards clip at the bottom, raise it.

**Pick the fields.** With `frmInfo` selected, open **Edit fields** in
the right pane and make the form show exactly these nine, in this
order (drag to reorder):

1. `Name`
2. `ProblemStatement`
3. `AISolution`
4. `Type`
5. `Status`
6. `SBU`
7. `LOB`
8. `TargetDate`
9. `RefreshFreq`

Remove every other auto-added card — `UCID`, `Owner`, `OwnerInitials`,
`FY`, `RealizedValue`, `EstimatedValue`, `LastUpdated` — they aren't
edited on this screen.

### Step 34 — Form fields — card widths, controls & Save

**Full-width cards.** Select the data card for `Name`,
`ProblemStatement`, and `AISolution` in turn and set each card's
**Width** to `frmInfo.Width`, so it takes a whole row and pushes the
next card beneath it. For `ProblemStatement` and `AISolution`, also
unlock the card (gear icon → **Advanced → Unlock**), select the inner
text input, set `Mode = TextMode.MultiLine`, and raise the card
**Height** (e.g. `120`).

The remaining six cards keep the default half-column width and flow
into two columns automatically.

**Collection-bound dropdowns.** Because `colUseCases` is a plain
collection (no choice metadata), the form generates *text inputs* for
`Type`, `SBU`, and `RefreshFreq`. Convert each to a dropdown:

1. Select the card → gear → **Advanced → Unlock**.
2. Delete the default `DataCardValue` text input.
3. Insert → **Dropdown** inside the card; name it `ddType`, `ddSBU2`,
   `ddRefresh` respectively.
4. Set the dropdown **Items**, its **Default**, and the card's
   **Update**:

| Card | Dropdown Items | Dropdown Default | DataCard **Update** |
|------|----------------|------------------|---------------------|
| Type | `colUseCaseType` | `Parent.Default` | `ddType.Selected.Value` |
| SBU | `colSBU` | `Parent.Default` | `ddSBU2.Selected.Value` |
| RefreshFreq | `colRefreshFreq` | `Parent.Default` | `ddRefresh.Selected.Value` |

`Parent.Default` is the card's bound value (the field from
`selectedUC`). The card's **Update** property is exactly what
`SubmitForm` writes back to `colUseCases`.

**Status card (code ↔ label).** `Status` stores a *code*
(`DataPrep`) but must display a *label* (`Data Prep`). Unlock the
`Status` card, replace its text input with a Dropdown `ddStatus2`,
then set:

| Property | Value |
|----------|-------|
| `ddStatus2.Items` | `colStatus.Label` |
| `ddStatus2.Default` | `LookUp(colStatus, Code = Parent.Default, Label)` |
| `Status` DataCard **Update** | `LookUp(colStatus, Label = ddStatus2.Selected.Value, Code)` |

This shows the label but writes the **code** back on save — the same
code/label split the old standalone dropdown handled, and what the
stepper's `LookUp(colStatus, Code = selectedUC.Status, Order)` needs.

**Date & text cards.** `TargetDate` auto-generates a Date picker —
leave it (its Update is already `DataCardValue.SelectedDate`). `Name`
and `LOB` stay as the default text inputs.

**Optional v1-only fields (not persisted).** The old form also showed
*Other LOBs Impacted*, *Output Deliverable*, and *Prerequisite for
Other Initiatives*, which aren't columns in `colUseCases`. If you want
them on screen, add each via **Edit fields → Add a custom card**, drop
in a control, and leave the card **Update** blank — they'll display
but won't be saved in v1. Otherwise skip them.

**Save / Cancel.** Below the form, inside `conSectionInfo`, add two
buttons:

| Property | `btnSaveInfo` | `btnCancelInfo` |
|----------|---------------|------------------|
| Text | `"Save"` | `"Cancel"` |
| X | `28` | `btnSaveInfo.X + btnSaveInfo.Width + 12` |
| Y | `frmInfo.Y + frmInfo.Height + 12` | `btnSaveInfo.Y` |
| Width | `120` | `120` |
| Height | `36` | `36` |
| OnSelect | `SubmitForm(frmInfo)` | `ResetForm(frmInfo)` |
| Fill | `gblTheme.Maroon` | `gblTheme.Surface` |
| Color | `White` | `gblTheme.Ink` |
| DisplayMode | `If(frmInfo.Unsaved, DisplayMode.Edit, DisplayMode.Disabled)` | `DisplayMode.Edit` |

Then select `frmInfo` itself and set:

| Property | Value |
|----------|-------|
| OnSuccess | `Set(selectedUC, frmInfo.LastSubmit)` |
| OnFailure | `Notify("Couldn't save: " & frmInfo.Error, NotificationType.Error)` |

`OnSuccess` refreshes `selectedUC` from the just-saved record, so the
rail-head pill and the status stepper advance the moment the save
lands. `btnSaveInfo` stays disabled until there are unsaved edits
(`frmInfo.Unsaved`); **Cancel** discards them via `ResetForm`.

### Step 35 — Sanity check

From `srcList`, click UC-0142. On Detail with section = Info:

- [ ] Section card visible: white, rounded corners, gray border.
- [ ] "Use Case Info" title in maroon, with a 2px maroon underline.
- [ ] Status stepper: 7 circles with labels (Rationale through
      Decommissioning). Circles 1–3 are filled maroon (Development is
      step 3); circles 4–7 are hollow gray. Each circle's center sits
      exactly on the connecting line, and the line begins at the first
      circle's center and ends at the last circle's center.
      The line behind connects them with maroon up to step 3.
- [ ] Edit form below: Name, Problem Statement, AI Solution
      Description span full width; Type, Status, SBU, LOB, Completion
      Date, Refresh sit in a 2-column grid. A **Save** and **Cancel**
      button sit under the form.
- [ ] **Save** is greyed out until you change a field; editing any
      field enables it.
- [ ] The Status dropdown shows labels (e.g. "Data Prep"), not codes.
- [ ] Change Status to "Testing" and press **Save**: the stepper
      advances (circle 4 fills maroon, the track extends) and the rail
      head pill updates. **Cancel** instead reverts unsaved edits.
- [ ] Toggle the rail. Both form columns widen/narrow proportionally;
      no card overflows.

If the stepper doesn't advance after saving, check `frmInfo.OnSuccess`
is `Set(selectedUC, frmInfo.LastSubmit)` and that the `Status` card's
**Update** writes the **code** back (`LookUp(colStatus, Label =
ddStatus2.Selected.Value, Code)`) — `lblCurrentStatusIdx` looks up the
code, so saving the label would never match.

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
   a caption label above each input (caption Size `12`, Semibold,
   Color `gblTheme.Ink2`; input Height `36`, full card width). Stack
   them vertically starting at Y=`60` with 64px between rows.
   Suggested controls:

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
title `"Funds"` + underline. Then a manual 2-column grid of 6 fields:
a caption label above each input, half-column width
`(Parent.Width - 56 - 24) / 2`, left col X `28`, right col X
`28 + ((Parent.Width - 56 - 24) / 2) + 24`. Fields:

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
- [ ] **Info:** the Edit form renders all fields. Changing Status and
      pressing **Save** updates the rail head's name label, the
      breadcrumb, and advances the horizontal stepper.
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
| Stepper circles never highlight | `selectedUC.Status` doesn't match a `Code` in `colStatus` | Status codes must be exactly one of `colStatus.Code`: `Rationale`, `DataPrep`, `Development`, `Testing`, `Deployment`, `Monitoring`, `Decommissioning`. Check `colUseCases` data. |
| Stepper circles never highlight **after switching to Dataverse** | `selectedUC.Status` is now the `'Project Status'` option set, and codes don't match the choice labels | Match on a new `DVStatus` field via `Text(selectedUC.'Project Status')` — see [`13-status-stepper-dataverse-fix.md`](13-status-stepper-dataverse-fix.md). |
| Status changes don't update the stepper after Save | The `Status` card's **Update** writes the Label back to `Status` instead of the Code | Set the card **Update** to `LookUp(colStatus, Label = ddStatus2.Selected.Value, Code)` — see Step 34, Status card. |
| Edits don't appear elsewhere after Save | `frmInfo.OnSuccess` doesn't refresh `selectedUC` | Set `OnSuccess = Set(selectedUC, frmInfo.LastSubmit)` — see Step 34. |
| Save button never enables | Bound to the wrong state | `DisplayMode = If(frmInfo.Unsaved, DisplayMode.Edit, DisplayMode.Disabled)`; it enables only once a card changes. |
| Sections all visible at once | A section's `Visible` formula is missing or evaluates to true unconditionally | Each `conSection*.Visible` must be `currentSection = "<key>"`. Check Step 27, 36, 40, 50, 51, 52, 53. |
| Clicking section nav does nothing | Button's `OnSelect` is empty (e.g. when you copied `btnSecInfo` you forgot to retarget OnSelect) | Each button's OnSelect must be `Set(currentSection, "<Key>")` with the row's key — see Step 24 table. |
| All seven nav rows look "active" (or none do) | The Fill/Color/FontWeight formulas on a copied button still compare against `"Info"` | Update all four formulas on each copy (button Fill, Color, FontWeight, and the accent rectangle's Fill) to compare against the row's own key. |
| Badge label blocks clicks on the Value or Gov row | `lblSecBadgeValue.DisplayMode` isn't `DisplayMode.Disabled` | Set it; the badge will let clicks pass through to the button underneath. |
| Cursor stays an arrow over a nav row | You're testing in Studio edit mode | Press F5; hand cursor only appears in preview. |
| Hovering a nav row does nothing | Button's `HoverFill` is `Self.Fill` (Studio default in some templates) | Set it to the formula in Step 22 (`RGBA(122, 26, 46, 0.18)`). |
| Section rail head doesn't update when navigating between use cases | `lblRailName`, etc. reference a hardcoded value | Their `Text` must reference `selectedUC.<Field>`, not a literal. |
| Custom component (`cmpStatusPill`) won't insert into `conRowSignoff` | Power Apps blocks components inside containers nested in gallery templates | Use the inline Circle + Label (Step 45). The component still works in `conRailHead` (Step 20) because that's not under a gallery. |
| Modal doesn't cover the whole screen | `conValueModalScrim` was created inside `conSectionValue` instead of at the screen root | Cut it, paste at the `srcDetail` root level. |
| Modal Save creates a row but with `UCID = ""` | New-row branch missing `UCID: selectedUC.UCID` in the `Collect` | See Step 48 — the first branch of the `If(IsBlank(editingValueRow.UCID), …)` must include `UCID: selectedUC.UCID`. |
| Progress bar fill is 0 width on first load | `govTotalCount` is 0 (no governance rows for this UCID), so `govProgressPct` is 0 | This is correct behavior. Confirm `colGovernance` has rows with `UCID = selectedUC.UCID`. |
| OnChange Patch saves but other sections still show old values | Missing the `Set(selectedUC, LookUp(…))` line after Patch | Every form input's OnChange must Patch, then re-Set `selectedUC` so dependent labels refresh. |
| Stepper labels overlap because columns are too narrow | Gallery `TemplateSize` (horizontal galleries = row width) is too small | Should be `Self.Width / CountRows(colStatus)` (7 cells). If the gallery's Width is wrong, fix that first — should be `Parent.Width - 56`. |
| Track line doesn't reach / overshoots the end circles | `recStepTrack.X` / `.Width` still use the old magic numbers (`60`, `Parent.Width - 56 - 80`) | Derive them from the gallery: `X = galStepper.X + galStepper.Width / (CountRows(colStatus) * 2)`, `Width = galStepper.Width * (CountRows(colStatus) - 1) / CountRows(colStatus)` — see Step 30. |
| Circles sit above or below the line | `circStep.Y` doesn't balance the gallery's `Y` offset | With `galStepper.Y = recStepTrack.Y - 24` and an 18 px circle, `circStep.Y` must be `16` so the center lands at `recStepTrack.Y + 1` — see Step 32. |
| Two-column form fields overflow the right edge when rail expands | Field Width is hardcoded instead of using the half-col formula | Width must be `(Parent.Width - 56 - 24) / 2`. Same for right-column X anchor. |
| Action bar overlaps the detail layout below | `conDetailLayout.Y` is `52` instead of `108` | Y must be `52 + 56 = 108` to clear both the header and the action bar. |
| Toggle the rail and the action bar / detail layout don't shift | X formula is hardcoded instead of `If(sideCollapsed, 64, 220)` | Both `conActionBar.X` and `conDetailLayout.X` must use the formula. Width must subtract the same offset. |
| Stepper line crosses through the circles instead of behind them | Tree order: `galStepper` is below `recStepTrack` in the tree | In the tree, drag `galStepper` to be **above** `recStepTrack` (later in tree = on top visually). |
| `Navigate(srcList, …)` errors from Back button | `srcList` doesn't exist as a named screen | Build `srcList` first (`07-scrlist-guide.md`) or create a blank screen named exactly `srcList`. |
| `Now()` makes `lblSaveNote` jitter | `Now()` re-evaluates on any reactive change | For v1 this is acceptable. For v2, replace with `Set(lastSavedAt, Now())` in `btnSaveDraft.OnSelect` and reference `lastSavedAt` here. |
