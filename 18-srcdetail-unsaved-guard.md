# BOA — srcDetail Unsaved-Changes Guard (per-tab forms)

Keep the original design — **one Edit form per tab** (`frmInfo`,
`frmContacts`, …) — and add a guard so that if a user edits a tab and tries
to **switch tabs before saving**, a popup stops them and offers:

- **Save changes** — submit this tab, then go to the new tab
- **Discard changes** — throw away the edits, then go to the new tab
- **Cancel** — stay on the current tab (don't navigate)

This prevents the "edited a tab, switched away, lost it on close" problem
that comes with per-tab forms.

## How it works (the flow)

1. The section-nav buttons normally do `Set(currentSection, "X")`.
2. We intercept: if the **current** tab's form is `Unsaved`, the button
   instead **remembers where you wanted to go** (`pendingSection`) and
   **opens the warning** (`showNavWarning`) — it does *not* navigate yet.
3. The three popup buttons resolve it. **Save changes** can't navigate
   immediately (`SubmitForm` is asynchronous), so it submits and lets the
   form's **`OnSuccess`** do the navigation once the save lands.

---

## Step 1 — State variables

Add to `App.OnStart` (so they start clean):

```powerfx
Set(showNavWarning, false);
Set(pendingSection, Blank());
```

- `pendingSection` — the tab the user is trying to reach (set when a guarded
  nav is intercepted).
- `showNavWarning` — controls the popup's `Visible`.

---

## Step 2 — The "is the current tab dirty?" check

Only the **currently-visible** tab's form can have unsaved edits, so the
dirty test is a `Switch` on `currentSection`. You'll use this exact
expression in the nav buttons (Step 3):

```powerfx
Switch(currentSection,
    "Info",     frmInfo.Unsaved,
    /* "Contacts", frmContacts.Unsaved, */
    /* "Funds",    frmFunds.Unsaved,    */
    /* "Gov",      frmGov.Unsaved,      */
    /* "Tech",     frmTech.Unsaved,     */
    /* "Updates",  frmUpdates.Unsaved,  */
    false   // default (e.g. Value tab has no form → never dirty)
)
```

> **Add a branch only once that tab's form exists** — referencing a
> non-existent `frm…` errors. For now (Info only) it's just the `"Info"`
> branch + the `false` default. The **Value** tab has no Edit form (it's a
> child gallery with its own add/edit), so it stays on the `false` default
> and never triggers the warning.

---

## Step 3 — Rewire each section-nav button

For **every** section-nav button, change `OnSelect` from
`Set(currentSection, "<target>")` to the guarded version below. Only the
**`"<target>"`** string differs per button (e.g. the Contacts button uses
`"Contacts"`):

```powerfx
If(
    // is the tab I'm leaving dirty?
    Switch(currentSection, "Info", frmInfo.Unsaved, /* …other built tabs… */ false),
    // yes → stash target + open the warning (don't navigate yet)
    Set(pendingSection, "<target>"); Set(showNavWarning, true),
    // no → navigate normally
    Set(currentSection, "<target>")
)
```

*(If repeating the `Switch` in every button feels heavy, you can put it once
in a hidden label — `lblDirty.Text = If(Switch(...), "1", "0")` — and have
each button test `lblDirty.Text = "1"`. Optional; the inline version is
fine.)*

---

## Step 4 — Build the warning popup

Place it at the **root of `srcDetail`** (not inside a section) so it covers
the whole screen. Insert → **Container** (the scrim):

| Property | Value |
|----------|-------|
| Name | `conNavWarnScrim` |
| X | `0` |
| Y | `0` |
| Width | `Parent.Width` |
| Height | `Parent.Height` |
| Fill | `RGBA(0, 0, 0, 0.45)` |
| BorderThickness | `0` |
| Visible | `showNavWarning` |

Inside it, Insert → **Container** (the dialog card):

| Property | Value |
|----------|-------|
| Name | `conNavWarnCard` |
| X | `(Parent.Width - Self.Width) / 2` |
| Y | `(Parent.Height - Self.Height) / 2` |
| Width | `460` |
| Height | `200` |
| Fill | `gblTheme.Surface` |
| BorderColor | `gblTheme.Border` |
| BorderThickness | `1` |
| RadiusTopLeft / TopRight / BottomLeft / BottomRight | `6` |

Inside `conNavWarnCard`, add a title and message Label:

| Control | Name | Key properties |
|---|---|---|
| Label | `lblNavWarnTitle` | Text `"Unsaved changes"`, X `28`, Y `24`, Width `Parent.Width - 56`, Height `24`, Size `16`, FontWeight `FontWeight.Bold`, Color `gblTheme.Ink`, Font `gblTheme.FontFamily` |
| Label | `lblNavWarnMsg` | Text `"You have unsaved changes on this tab. What would you like to do before leaving?"`, X `28`, Y `60`, Width `Parent.Width - 56`, Height `48`, Size `13`, Color `gblTheme.Ink2`, Font `gblTheme.FontFamily` |

---

## Step 5 — The three buttons (inside `conNavWarnCard`)

Lay them right-aligned along the bottom. All: `Y = Parent.Height - 52`,
`Height 36`, `RadiusAll 4`, `Font gblTheme.FontFamily`, `Size 13`.

**`btnNavSave` — "Save changes"** (primary, rightmost):

| Property | Value |
|----------|-------|
| Text | `"Save changes"` |
| X | `Parent.Width - 28 - Self.Width` |
| Width | `140` |
| Fill | `gblTheme.Maroon` |
| Color | `White` |
| BorderThickness | `0` |
| OnSelect | `Set(showNavWarning, false); Switch(currentSection, "Info", SubmitForm(frmInfo) /* , …other tabs… */ )` |

**`btnNavDiscard` — "Discard changes"** (secondary):

| Property | Value |
|----------|-------|
| Text | `"Discard changes"` |
| X | `btnNavSave.X - 12 - Self.Width` |
| Width | `130` |
| Fill | `gblTheme.Surface` |
| Color | `gblTheme.Maroon` |
| BorderColor | `gblTheme.Maroon` |
| BorderThickness | `1` |
| OnSelect | `Switch(currentSection, "Info", ResetForm(frmInfo) /* , …other tabs… */ ); Set(currentSection, pendingSection); Set(pendingSection, Blank()); Set(showNavWarning, false)` |

**`btnNavCancel` — "Cancel"** (ghost):

| Property | Value |
|----------|-------|
| Text | `"Cancel"` |
| X | `btnNavDiscard.X - 12 - Self.Width` |
| Width | `90` |
| Fill | `gblTheme.Surface` |
| Color | `gblTheme.Ink2` |
| BorderColor | `gblTheme.Border` |
| BorderThickness | `1` |
| OnSelect | `Set(pendingSection, Blank()); Set(showNavWarning, false)` |

What each does:

- **Save changes:** hides the popup, then `SubmitForm`s the current tab's
  form. Navigation to `pendingSection` happens in the form's `OnSuccess`
  (Step 6) — *after* the save actually succeeds.
- **Discard changes:** `ResetForm` throws away the edits, then navigates to
  `pendingSection` immediately and clears state.
- **Cancel:** clears the pending target and closes — you stay put.

---

## Step 6 — Make each form navigate after a guarded save

Each section form's `OnSuccess` must (a) refresh `selectedUC` as before,
and (b) **if a navigation was pending, go there now**. Set every section
form's `OnSuccess` to:

```powerfx
Set(selectedUC, <thisForm>.LastSubmit);
If(!IsBlank(pendingSection),
    Set(currentSection, pendingSection);
    Set(pendingSection, Blank())
)
```

e.g. for `frmInfo`: `Set(selectedUC, frmInfo.LastSubmit); If(!IsBlank(pendingSection), Set(currentSection, pendingSection); Set(pendingSection, Blank()))`.

And `OnFailure` — clear the pending nav so a failed save doesn't strand it:

```powerfx
Notify("Couldn't save: " & <thisForm>.Error, NotificationType.Error);
Set(pendingSection, Blank())
```

Now: a normal **Submit Assessment** (no pending nav) just saves and stays
(`pendingSection` is blank → no navigation). A **Save changes** from the
popup saves *and* moves to the pending tab once the save lands.

---

## Step 7 — Notes

- **Add `Switch`/button branches per tab as you build each form** — not
  before (missing-control references error).
- **Back button (leaving srcDetail entirely):** apply the same guard — its
  `OnSelect` becomes `If(Switch(currentSection, …)…, Set(pendingSection, "__back"); Set(showNavWarning, true), Back())`,
  and handle `"__back"` in the Save/Discard buttons (navigate with `Back()`
  instead of `Set(currentSection, …)`). Optional; do it once tabs are built.
- The action-bar **Submit Assessment** / **Discard Changes** (guide 17 Step
  10) stay as-is — they let you save/discard *without* leaving the tab. The
  popup is only for the **navigate-away** case.
- **Value tab:** no form, so it never triggers the warning (the `false`
  default). Its rows are saved by its own + Add / Edit.

---

## Step 8 — Sanity check

- [ ] On a tab with **no edits**, clicking another tab switches instantly
      (no popup).
- [ ] Edit a field, click another tab → popup appears; the current tab is
      still showing behind it.
- [ ] **Cancel** → popup closes, you're still on the tab, edits intact.
- [ ] **Discard changes** → edits revert, you move to the tab you clicked.
- [ ] **Save changes** → row saves to Dataverse, *then* you land on the tab
      you clicked; the stepper/pill reflect the save.
- [ ] If a save fails, you stay put with an error notification.
