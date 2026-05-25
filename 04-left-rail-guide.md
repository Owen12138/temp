# BOA Canvas App — Collapsible Left Rail (Full Step-by-Step)

This replaces the left rail section in `02-build-guide.md`. Follow this instead.

## Existing variables — do not re-create these

From `App.OnStart` we already have:

```powerfx
Set(sideCollapsed, true);   // true = 64px collapsed, false = 220px expanded
```

That is the only variable the rail needs. Do NOT add `varMenuExpanded` or
anything else. For active-screen detection we use `App.ActiveScreen.Name`
which is a built-in Power Apps property accessible from anywhere including
inside components.

---

## How it works

The component `cmpLeftRail` takes one custom input property:

- `MenuExpanded` (Boolean) — driven by `!sideCollapsed` on each screen instance

The toggle button inside the component calls `Set(sideCollapsed, !sideCollapsed)`.
Active-item highlighting reads `App.ActiveScreen.Name` directly — no extra
variable or input property needed.

---

## Part 1 — Enable navigation from components (do this first)

Components cannot reference screen objects (`srcHome`, `srcList`, etc.) by
default. You must enable it:

**File → Settings → Upcoming features → find "Allow canvas components to
navigate" → turn ON → Save.**

Without this, the gallery's `Navigate(srcHome, ...)` calls will error. This
is the correct fix — do not try to work around it by moving navigation outside
the component.

---

## Part 2 — Create the component

### Step 1 — New component

Tree view → **Components tab** → **+ New component**.
Rename it to `cmpLeftRail`.

> If the name is taken after deleting a previous attempt: save (Ctrl+S),
> close the app, reopen it, then try again.

### Step 2 — Set the component canvas size

Click `cmpLeftRail` in the tree (the component root, not a child control).
Right panel → **Properties tab**:
- Width: `220`
- Height: `768`

Width becomes dynamic in Step 10. Use fixed numbers for now.

### Step 3 — Add custom input property: MenuExpanded

Top toolbar → **New custom property**:

| Field | Value |
|-------|-------|
| Display name | `MenuExpanded` |
| Property type | `Input` |
| Data type | `Boolean` |
| Default value | `false` |

Click **Create**. This is the only custom property the component needs.

---

## Part 2 — Build controls inside the component

> Rules inside a component:
> - No `gblTheme.X` — use hardcoded `RGBA(...)`
> - No `Transparent` — use `RGBA(0,0,0,0)`
> - `App.ActiveScreen.Name` ✓ works (built-in, not a user variable)
> - `Set(sideCollapsed, ...)` ✓ works (writing globals from a component is allowed)

### Step 4 — Background fill

Click `cmpLeftRail` in the tree. Properties tab → **Fill**: `RGBA(255,255,255,1)`.

### Step 5 — Fake right border

Insert → **Rectangle**:

| Property | Value |
|----------|-------|
| Name | `recRailBorder` |
| X | `Parent.Width - 1` |
| Y | `0` |
| Width | `1` |
| Height | `Parent.Height` |
| Fill | `RGBA(225,225,225,1)` |
| BorderThickness | `0` |

### Step 6 — Toggle button

Insert → **Icons** → pick the hamburger/menu icon (three horizontal lines):

| Property | Value |
|----------|-------|
| Name | `icnToggle` |
| X | `18` |
| Y | `(56 - 28) / 2` |
| Width | `28` |
| Height | `28` |
| Color | `RGBA(74,74,74,1)` |
| OnSelect | `Set(sideCollapsed, !sideCollapsed)` |

### Step 7 — "BOA MENU" label

Insert → **Text label**:

| Property | Value |
|----------|-------|
| Name | `lblRailTitle` |
| X | `54` |
| Y | `20` |
| Width | `150` |
| Height | `16` |
| Text | `"BOA MENU"` |
| Size | `10` |
| FontWeight | `Bold` |
| Color | `RGBA(74,74,74,1)` |
| Visible | `cmpLeftRail.MenuExpanded` |

### Step 8 — Nav gallery

Insert → **Vertical gallery**:

| Property | Value |
|----------|-------|
| Name | `galRailNav` |
| X | `0` |
| Y | `56` |
| Width | `Parent.Width` |
| Height | `120` |
| TemplateSize | `40` |
| ShowScrollbar | `false` |

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

### Step 9 — Gallery template controls

The gallery template has 4 default controls. Map them as follows.
To tell the two rectangles apart: click each and check Height in the formula
bar — the tall one is the accent, the short one (Height 1) is the separator.

---

**recRailAccent** — first rectangle (tall, full row height):

| Property | Value |
|----------|-------|
| Name | `recRailAccent` |
| X | `0` |
| Y | `0` |
| Width | `3` |
| Height | `Parent.TemplateHeight` |
| BorderThickness | `0` |

Fill — uses `App.ActiveScreen.Name` to detect the active screen:
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

**recRowSeparator** — second rectangle (thin, Height 1):

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

**icnRailItem** — rename from NextArrow2:

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

**lblRailItem** — rename from Title2:

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
| Visible | `cmpLeftRail.MenuExpanded` |

Color (same formula as icon):
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

### Step 10 — Make component Width dynamic (design canvas only)

Click `cmpLeftRail` in the tree (component root). Formula bar → switch
property dropdown to **Width**:

```powerfx
If(cmpLeftRail.MenuExpanded, 220, 64)
```

> **Important:** This only sets the design canvas size while you are inside
> the component editor. It does NOT control the rendered width on screen.
> The instance Width you set in Step 11 is what actually collapses the
> component visually. Both must be set.

---

## Part 3 — Place the component on each screen

Repeat for srcHome, srcList, srcDetail, srcNew.

### Step 11 — Insert and configure the instance

Go to a screen → Insert → **Custom** → `cmpLeftRail`. Set:

| Property | Value |
|----------|-------|
| X | `0` |
| Y | `52` |
| Width | `If(sideCollapsed, 64, 220)` |
| Height | `Parent.Height - 52` |
| MenuExpanded | `!sideCollapsed` |

> **This Width formula is what actually collapses the component on screen.**
> The component's internal Width (Step 10) is only for the design canvas.
> If Width here is a fixed number, the component will never visually collapse
> no matter what the toggle does.

`MenuExpanded` appears in the **Advanced tab** once the component is on the screen.

> No `OnVisible` changes needed for the rail — `App.ActiveScreen.Name`
> updates automatically when navigation happens.

---

## Part 4 — Header and content offsets

Each screen's header and content container must start to the right of the rail.
These already use `sideCollapsed` in the existing guide — confirm they are set to:

**Header X:**
```powerfx
If(sideCollapsed, 64, 220)
```

**Header Width:**
```powerfx
Parent.Width - If(sideCollapsed, 64, 220)
```

**Content container (conPage) X and Width:** same as above.

---

## Quick reference — things that go wrong

| Symptom | Cause | Fix |
|---------|-------|-----|
| `gblTheme.X` error | Component can't read the `gblTheme` record variable | Use hardcoded `RGBA(...)` |
| `Transparent` error | Not a valid keyword inside a component | Use `RGBA(0,0,0,0)` |
| Active item never highlights | `App.ActiveScreen.Name` not matching screen name | Check your screen is named exactly `srcHome`, `srcList`, `srcDetail`, `srcNew` |
| Width not collapsing | Instance Width is a fixed number | Set to `If(sideCollapsed, 64, 220)` |
| Toggle does nothing | `icnToggle.OnSelect` wrong | Must be `Set(sideCollapsed, !sideCollapsed)` |
| Label still shows when collapsed | `lblRailItem.Visible` wrong | Must be `cmpLeftRail.MenuExpanded` |
