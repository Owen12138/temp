# BOA Canvas App — Collapsible Left Rail (Buttons, No Gallery)

Use this guide if you hit any of these problems with `05-left-rail-inline-guide.md`:

- The row icons don't render (the Icon control stays blank or shows the
  fallback shape).
- Hovering a nav row does nothing — no background change, no color change.
- The active screen's row doesn't darken — only the 3px accent bar is maroon.
- The mouse cursor stays as an arrow over the nav items instead of turning
  into a hand.

All four are symptoms of the same root cause: **PowerApps galleries are bad
at the things a nav rail needs**. Galleries don't expose a per-row `HoverFill`,
`Icon.X` enums don't survive being stored in a Table column and read via
`ThisItem.Icn`, and the template controls eat clicks in ways that suppress
the hand cursor.

The fix is to drop the gallery and build each nav row out of one `Button`
(the click target, with native `HoverFill` / `PressedFill` / `Fill`) plus one
`Icon` overlaid on top. Three nav items means three rows — totally manageable.

---

## What you keep from guide 05

Steps 1–4 from `05-left-rail-inline-guide.md` are unchanged:

- `conLeftRail` container at `X:0, Y:52, Width: If(sideCollapsed, 64, 220)`,
  `Height: Parent.Height - 52`, `Fill: RGBA(255,255,255,1)`.
- `recRailBorder` rectangle (1px right border).
- `icnRailToggle` hamburger icon with
  `OnSelect: Set(sideCollapsed, !sideCollapsed)`.
- `lblRailTitle` "BOA MENU" label with `Visible: !sideCollapsed`.

If you already have these on `scrHome`, leave them alone. If you also added
the `galRailNav` gallery from step 5 of guide 05, **delete it now** — we're
replacing it.

---

## Part 1 — Build the three nav rows on `scrHome`

You'll add three rows: Home, View/Edit Use Cases, New Use Case. Each row is
a Button + an Icon overlaid on top. Build the Home row completely first, then
copy/paste it twice and tweak.

> All controls below go **inside `conLeftRail`** (drop them in the tree under
> the container, not on the screen root).

---

### Step 1 — Home row: the button

Inside `conLeftRail`, Insert → **Button**.

| Property | Value |
|----------|-------|
| Name | `btnNavHome` |
| X | `0` |
| Y | `56` |
| Width | `Parent.Width` |
| Height | `40` |
| Text | `If(!sideCollapsed, "Home", "")` |
| Align | `Align.Left` |
| PaddingLeft | `44` |
| Size | `12` |
| FontWeight | `FontWeight.Semibold` |
| BorderThickness | `0` |
| RadiusTopLeft | `0` |
| RadiusTopRight | `0` |
| RadiusBottomLeft | `0` |
| RadiusBottomRight | `0` |
| OnSelect | `Navigate(scrHome, ScreenTransition.None)` |

**Fill** — transparent when not active, light maroon tint when active:
```powerfx
If(
    App.ActiveScreen.Name = "scrHome",
    RGBA(122, 26, 46, 0.10),
    RGBA(0, 0, 0, 0)
)
```

**HoverFill** — slightly darker on hover (works whether active or not):
```powerfx
RGBA(122, 26, 46, 0.18)
```

**PressedFill** — slightly darker still on click:
```powerfx
RGBA(122, 26, 46, 0.28)
```

**Color** — maroon text when active, dark gray otherwise:
```powerfx
If(
    App.ActiveScreen.Name = "scrHome",
    RGBA(122, 26, 46, 1),
    RGBA(74, 74, 74, 1)
)
```

**HoverColor** — match the active color so hovered rows feel "live":
```powerfx
RGBA(122, 26, 46, 1)
```

**PressedColor** — same as HoverColor:
```powerfx
RGBA(122, 26, 46, 1)
```

> The button is full-width, so its `HoverFill` darkens the entire row when
> the mouse enters anywhere on the row that isn't covered by the icon. The
> hand cursor is automatic on any Button with an OnSelect.

---

### Step 2 — Home row: the icon overlay

Still inside `conLeftRail`, Insert → **Icons** → pick **Home** (the house
icon). If you can't find it, insert any icon and change `Icon` in the formula
bar to `Icon.Home`.

| Property | Value |
|----------|-------|
| Name | `icnNavHome` |
| X | `14` |
| Y | `btnNavHome.Y + (btnNavHome.Height - 16) / 2` |
| Width | `16` |
| Height | `16` |
| Icon | `Icon.Home` |
| Fill | `RGBA(0, 0, 0, 0)` |
| HoverFill | `RGBA(0, 0, 0, 0)` |
| PressedFill | `RGBA(0, 0, 0, 0)` |
| BorderThickness | `0` |
| OnSelect | `Select(btnNavHome)` |

**Color** — same active formula as the button:
```powerfx
If(
    App.ActiveScreen.Name = "scrHome",
    RGBA(122, 26, 46, 1),
    RGBA(74, 74, 74, 1)
)
```

**HoverColor**:
```powerfx
RGBA(122, 26, 46, 1)
```

**PressedColor**:
```powerfx
RGBA(122, 26, 46, 1)
```

> `Select(btnNavHome)` is the critical trick: when the user clicks the icon
> area, the icon's OnSelect programmatically triggers the button's OnSelect,
> which runs the `Navigate`. So clicking anywhere on the row navigates,
> whether on the icon or the text area.

---

### Step 3 — (Optional) Home row: active accent bar

If you want the 3px maroon strip on the left edge of the active row, add a
rectangle. Skip this if the row tint from Step 1 is enough for you.

Inside `conLeftRail`, Insert → **Rectangle**:

| Property | Value |
|----------|-------|
| Name | `recAccentHome` |
| X | `0` |
| Y | `btnNavHome.Y` |
| Width | `3` |
| Height | `btnNavHome.Height` |
| BorderThickness | `0` |
| DisplayMode | `DisplayMode.Disabled` |

**Fill**:
```powerfx
If(
    App.ActiveScreen.Name = "scrHome",
    RGBA(122, 26, 46, 1),
    RGBA(0, 0, 0, 0)
)
```

> `DisplayMode.Disabled` keeps the rectangle from intercepting clicks on the
> leftmost 3px of the button.

---

### Step 4 — Sanity check on `scrHome`

Press F5. Confirm on `scrHome`:

- [ ] The "Home" row's background is faintly maroon (it's the active screen).
- [ ] Hovering "Home" darkens the maroon tint slightly.
- [ ] The cursor turns into a hand when you move over the row.
- [ ] The home icon is visible at the left of the row.
- [ ] (When expanded) the "Home" text appears in maroon to the right of the icon.

Don't move on until all five are working — the next two rows are copies of
this one, so any bug here gets multiplied.

---

### Step 5 — View/Edit Use Cases row

Select **both** `btnNavHome` and `icnNavHome` in the tree (Ctrl-click).
Copy (Ctrl+C), paste (Ctrl+V) inside `conLeftRail`. You'll get
`btnNavHome_1` and `icnNavHome_1`.

Rename and reconfigure them:

**`btnNavHome_1` → `btnNavList`:**

| Property | Value |
|----------|-------|
| Name | `btnNavList` |
| Y | `96` |
| Text | `If(!sideCollapsed, "View/Edit Use Cases", "")` |
| OnSelect | `Navigate(scrList, ScreenTransition.None)` |

**Fill** — match against both `scrList` and `scrDetail` so the detail page
also lights up the list row:
```powerfx
If(
    App.ActiveScreen.Name in ["scrList", "scrDetail"],
    RGBA(122, 26, 46, 0.10),
    RGBA(0, 0, 0, 0)
)
```

**Color**:
```powerfx
If(
    App.ActiveScreen.Name in ["scrList", "scrDetail"],
    RGBA(122, 26, 46, 1),
    RGBA(74, 74, 74, 1)
)
```

Leave `HoverFill`, `HoverColor`, `PressedFill`, `PressedColor` as they are.

**`icnNavHome_1` → `icnNavList`:**

| Property | Value |
|----------|-------|
| Name | `icnNavList` |
| Y | `btnNavList.Y + (btnNavList.Height - 16) / 2` |
| Icon | `Icon.DetailList` |
| OnSelect | `Select(btnNavList)` |

**Color** (same active formula as `btnNavList`):
```powerfx
If(
    App.ActiveScreen.Name in ["scrList", "scrDetail"],
    RGBA(122, 26, 46, 1),
    RGBA(74, 74, 74, 1)
)
```

> If you added the accent rectangle in Step 3, copy that too, rename to
> `recAccentList`, set `Y: btnNavList.Y`, and use the same active formula in
> its Fill.

---

### Step 6 — New Use Case row

Same drill — copy `btnNavHome` and `icnNavHome` once more, rename, retarget.

**`btnNavNew`:**

| Property | Value |
|----------|-------|
| Name | `btnNavNew` |
| Y | `136` |
| Text | `If(!sideCollapsed, "New Use Case", "")` |
| OnSelect | `Navigate(scrNew, ScreenTransition.None)` |

**Fill**:
```powerfx
If(
    App.ActiveScreen.Name = "scrNew",
    RGBA(122, 26, 46, 0.10),
    RGBA(0, 0, 0, 0)
)
```

**Color**:
```powerfx
If(
    App.ActiveScreen.Name = "scrNew",
    RGBA(122, 26, 46, 1),
    RGBA(74, 74, 74, 1)
)
```

**`icnNavNew`:**

| Property | Value |
|----------|-------|
| Name | `icnNavNew` |
| Y | `btnNavNew.Y + (btnNavNew.Height - 16) / 2` |
| Icon | `Icon.Add` |
| OnSelect | `Select(btnNavNew)` |

**Color**:
```powerfx
If(
    App.ActiveScreen.Name = "scrNew",
    RGBA(122, 26, 46, 1),
    RGBA(74, 74, 74, 1)
)
```

---

### Step 7 — Test the whole rail on `scrHome`

Press F5. Confirm:

- [ ] Three nav rows are visible, stacked vertically starting at Y=56.
- [ ] Each row's icon renders correctly (house, list, plus).
- [ ] Hovering any row darkens its background and turns the cursor into a hand.
- [ ] Clicking each row navigates to the right screen.
- [ ] On `scrHome`, only the Home row is tinted maroon.
- [ ] On `scrList`, only the View/Edit row is tinted maroon.
- [ ] On `scrDetail`, the View/Edit row is also tinted (because it shares
  the lookup).
- [ ] On `scrNew`, only the New Use Case row is tinted maroon.
- [ ] Collapse the rail with the hamburger. The text disappears; the icons
  remain centered-left. Hover still works. Clicking still navigates.

---

## Part 2 — Copy the rail to the other screens

Once `scrHome` is clean:

1. In the tree, click `conLeftRail` on `scrHome`.
2. **Ctrl+C**, then click `scrList` in the tree, then **Ctrl+V**.
3. Repeat for `scrDetail` and `scrNew`.

Because all formulas use `App.ActiveScreen.Name` and `sideCollapsed`, they
work identically on every screen with no edits.

---

## Part 3 — Header and content offsets

Unchanged from guide 05. Confirm on every screen:

**Header (`conHeader`):**

| Property | Value |
|----------|-------|
| X | `If(sideCollapsed, 64, 220)` |
| Width | `Parent.Width - If(sideCollapsed, 64, 220)` |

**Content (`conPage`):** same X and Width.

---

## Quick reference — things that go wrong

| Symptom | Cause | Fix |
|---------|-------|-----|
| Icon still blank | You set `Icon: ThisItem.Icn` somewhere | Use the literal enum: `Icon.Home`, `Icon.DetailList`, `Icon.Add` |
| Hover does nothing | Button has `HoverFill: Self.Fill` (default in some templates) | Set HoverFill to the formula in Step 1 |
| Cursor still an arrow | You're testing in Studio edit mode, not Play | Press F5 |
| Clicking the icon doesn't navigate | `icnNavHome.OnSelect` is empty | Set it to `Select(btnNavHome)` |
| Row tint never applies | Screen name typo | Screens must be named exactly `scrHome`, `scrList`, `scrDetail`, `scrNew` |
| Whole rail won't collapse | `conLeftRail.Width` is a fixed number | `If(sideCollapsed, 64, 220)` |
| Text shows when collapsed | Button `Text` is a plain string | `If(!sideCollapsed, "Home", "")` |
| Button has a maroon border outline on focus | Default `FocusedBorderThickness` | Set `FocusedBorderThickness: 0` if you don't want the focus ring |
