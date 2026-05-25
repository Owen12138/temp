# BOA Canvas App — Collapsible Left Rail (Inline, No Component)

Use this guide instead of `04-left-rail-guide.md` if you ran into trouble with
`cmpLeftRail`. Everything is built directly on each screen — no component tab,
no custom properties, no "Allow navigation from components" toggle needed.

The trade-off: you repeat the same controls on every screen. That's fine for
four screens and is much easier to debug.

---

## How it works

- One global variable `sideCollapsed` (Boolean) drives everything.
  - `true` = rail is 64 px wide (icons only)
  - `false` = rail is 220 px wide (icons + labels)
- Every screen has its own copy of the rail controls inside a plain Container.
- Active-item highlighting uses `App.ActiveScreen.Name`, which is always
  available on any screen without any setup.

---

## Prerequisites — App.OnStart

Open the **App** object in the tree, go to `OnStart`. Make sure this line is present
(it should already be there from `01-app-setup.md`):

```powerfx
Set(sideCollapsed, true);
```

Do not add it again if it's already there.

---

## Part 1 — Build the rail on one screen first

Do everything in this section on **srcHome**. Once it works there, you'll copy
the whole container to the other screens in Part 2.

---

### Step 1 — Add a Container for the rail

Go to **srcHome**. Insert → **Container** (not a Horizontal/Vertical layout
container — just a blank Container).

Set these properties:

| Property | Value |
|----------|-------|
| Name | `conLeftRail` |
| X | `0` |
| Y | `52` |
| Width | `If(sideCollapsed, 64, 220)` |
| Height | `Parent.Height - 52` |
| Fill | `RGBA(255,255,255,1)` |
| BorderThickness | `0` |

> The `Y: 52` places it below the maroon header. Adjust if your header height
> differs.

---

### Step 2 — Fake right border

Inside `conLeftRail`, Insert → **Rectangle**:

| Property | Value |
|----------|-------|
| Name | `recRailBorder` |
| X | `Parent.Width - 1` |
| Y | `0` |
| Width | `1` |
| Height | `Parent.Height` |
| Fill | `RGBA(225,225,225,1)` |
| BorderThickness | `0` |

---

### Step 3 — Toggle (hamburger) icon

Inside `conLeftRail`, Insert → **Icons** → choose the **hamburger / menu** icon
(three horizontal lines). If you can't find it by browsing, insert any icon and
then change the `Icon` property to `Icon.HamburgerMenu` in the formula bar.

| Property | Value |
|----------|-------|
| Name | `icnRailToggle` |
| X | `18` |
| Y | `14` |
| Width | `28` |
| Height | `28` |
| Color | `RGBA(74,74,74,1)` |
| OnSelect | `Set(sideCollapsed, !sideCollapsed)` |

---

### Step 4 — "BOA MENU" label

Inside `conLeftRail`, Insert → **Text label**:

| Property | Value |
|----------|-------|
| Name | `lblRailTitle` |
| X | `54` |
| Y | `20` |
| Width | `150` |
| Height | `16` |
| Text | `"BOA MENU"` |
| Size | `10` |
| FontWeight | `FontWeight.Bold` |
| Color | `RGBA(74,74,74,1)` |
| Visible | `!sideCollapsed` |

---

### Step 5 — Nav gallery

Inside `conLeftRail`, Insert → **Vertical gallery**:

| Property | Value |
|----------|-------|
| Name | `galRailNav` |
| X | `0` |
| Y | `56` |
| Width | `Parent.Width` |
| Height | `120` |
| TemplateSize | `40` |
| ShowScrollbar | `false` |
| Fill | `RGBA(0,0,0,0)` |
| BorderThickness | `0` |

**Items:**
```powerfx
Table(
    {Key: "Home", Label: "Home",                Icn: Icon.Home},
    {Key: "List", Label: "View/Edit Use Cases", Icn: Icon.DetailList},
    {Key: "New",  Label: "New Use Case",        Icn: Icon.Add}
)
```

**OnSelect:**
```powerfx
Switch(ThisItem.Key,
    "Home", Navigate(srcHome, ScreenTransition.None),
    "List", Navigate(srcList, ScreenTransition.None),
    "New",  Navigate(srcNew,  ScreenTransition.None)
)
```

> Because you're on a screen (not inside a component), `Navigate(srcHome, ...)`
> works immediately — no settings toggle required.

---

### Step 6 — Gallery template controls

The gallery template comes with some default controls. Rename and configure
them as below. To identify which rectangle is which, click each one and check
its Height in the formula bar: the tall one (full row height) becomes
`recRailAccent`; the short one (Height 1) becomes `recRowSeparator`.

---

**recRailAccent** — tall rectangle, active-state accent bar:

| Property | Value |
|----------|-------|
| Name | `recRailAccent` |
| X | `0` |
| Y | `0` |
| Width | `3` |
| Height | `Parent.TemplateHeight` |
| BorderThickness | `0` |

Fill — highlights the row whose screen is currently active:
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

---

**recRowSeparator** — thin 1px separator:

| Property | Value |
|----------|-------|
| Name | `recRowSeparator` |
| X | `0` |
| Y | `Parent.TemplateHeight - 1` |
| Width | `Parent.TemplateWidth` |
| Height | `1` |
| Fill | `RGBA(225,225,225,1)` |
| BorderThickness | `0` |

---

**icnRailItem** — nav icon per row (rename from `NextArrow2` or similar):

| Property | Value |
|----------|-------|
| Name | `icnRailItem` |
| X | `14` |
| Y | `(Parent.TemplateHeight - 16) / 2` |
| Width | `16` |
| Height | `16` |
| Icon | `ThisItem.Icn` |

Color:
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

---

**lblRailItem** — nav label per row (rename from `Title2` or similar):

| Property | Value |
|----------|-------|
| Name | `lblRailItem` |
| X | `40` |
| Y | `0` |
| Width | `Parent.TemplateWidth - 40` |
| Height | `Parent.TemplateHeight` |
| Text | `ThisItem.Label` |
| Size | `12` |
| VerticalAlign | `VerticalAlign.Middle` |
| Visible | `!sideCollapsed` |

Color (same active-state formula as the icon):
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

---

### Step 7 — Test on srcHome before copying

Press **F5** (or the Play button). Confirm:

- [ ] Rail appears on the left, 64px wide (icons only) on load.
- [ ] Clicking the hamburger expands it to 220px and shows labels.
- [ ] Clicking again collapses it back.
- [ ] The Home row's accent bar is maroon; the others are transparent.
- [ ] Clicking a nav item navigates to the correct screen.

Fix any issues before copying to other screens.

---

## Part 2 — Copy the rail to the other screens

Once srcHome is working, this is the fastest path:

1. In the tree view, click `conLeftRail` on srcHome to select it.
2. **Ctrl+C** to copy.
3. Click on **srcList** in the tree to make it the active screen.
4. **Ctrl+V** to paste. The container lands at the same X/Y/Width/Height.
5. Repeat for **srcDetail** and **srcNew**.

> The paste lands at the same position, so no repositioning should be needed.
> All formulas — including `If(sideCollapsed, 64, 220)` and
> `App.ActiveScreen.Name` — are global references and work correctly on every
> screen without any changes.

---

## Part 3 — Header and content offsets

Each screen's header bar and content container must start to the right of the
rail so they don't overlap. These properties should already reference
`sideCollapsed` from `02-build-guide.md` — just confirm they are set correctly.

**Header (conHeader) on each screen:**

| Property | Value |
|----------|-------|
| X | `If(sideCollapsed, 64, 220)` |
| Width | `Parent.Width - If(sideCollapsed, 64, 220)` |

**Content container (conPage) on each screen:**

| Property | Value |
|----------|-------|
| X | `If(sideCollapsed, 64, 220)` |
| Width | `Parent.Width - If(sideCollapsed, 64, 220)` |

---

## Quick-reference — things that go wrong

| Symptom | Cause | Fix |
|---------|-------|-----|
| Rail doesn't collapse | `conLeftRail.Width` is a fixed number | Set Width to `If(sideCollapsed, 64, 220)` |
| Toggle does nothing | `icnRailToggle.OnSelect` wrong | Must be `Set(sideCollapsed, !sideCollapsed)` |
| Labels still show when collapsed | `lblRailItem.Visible` wrong | Set to `!sideCollapsed` |
| Active row never highlights | Screen name typo | Check your screens are named exactly `srcHome`, `srcList`, `srcDetail`, `srcNew` |
| Content overlaps rail | Header/conPage X not updated | Set X and Width to use `If(sideCollapsed, 64, 220)` |
| Navigate errors on paste | Screen reference missing | Make sure all four screens exist before pasting |
| Rail appears behind content | Z-order issue | In tree view, drag `conLeftRail` to be above other containers |
