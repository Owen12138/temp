# BOA — srcDetail "Use Case Info" Tab on Dataverse (Fully Specified)

Build the **Use Case Info** section end-to-end against **Dataverse**, with
every position, dimension, and field spelled out. This continues from the
controls you already have in `conSectionInfo`:

- `lblInfoTitle` (title), `recUInfoTitleBorder` (underline)
- `recStepTrack`, `recStepTrackFill` (stepper track + fill)
- `galStepper` (stepper gallery)

We'll set their exact properties (Dataverse-aware), then add the **Edit
form** `frmInfo` and its Save/Cancel buttons.

**Prerequisites** (one-time, already covered elsewhere):
- `Projects` and `Values` added as data sources.
- `selectedUC` is a `Projects` record; `App.OnStart` seeds
  `Set(selectedUC, Defaults(Projects))`.
- `colStatus` has the `DVStatus` field ([guide 13](13-status-stepper-dataverse-fix.md)).
- `App.Formulas` has the status index (Step 0).

> **Names:** I use your control names (`recUInfoTitleBorder`,
> `recStepTrack`, …). If a name differs slightly in your tree, use yours.

---

## Step 0 — App.Formulas (named formula the stepper reads)

Click `App` → **Formulas** and make sure this is present:

```powerfx
currentStatusIdx = Coalesce(LookUp(colStatus, DVStatus = Text(selectedUC.'Project Status'), Order), 0);
```

This returns the active step (1–7), or 0 when blank/unknown. The stepper
controls below read it directly (no hidden label, no `Value()`).

---

## Step 1 — Title label (`lblInfoTitle`)

Select `lblInfoTitle`, set:

| Property | Value |
|----------|-------|
| Text | `"Use Case Info"` |
| X | `28` |
| Y | `24` |
| Width | `300` |
| Height | `24` |
| Size | `16` |
| FontWeight | `FontWeight.Bold` |
| Color | `gblTheme.Maroon` |
| Font | `gblTheme.FontFamily` |

## Step 2 — Title underline (`recUInfoTitleBorder`)

| Property | Value |
|----------|-------|
| X | `28` |
| Y | `lblInfoTitle.Y + lblInfoTitle.Height + 4` |
| Width | `Parent.Width - 56` |
| Height | `2` |
| Fill | `gblTheme.Maroon` |
| BorderThickness | `0` |

---

## Step 3 — Stepper track (`recStepTrack`)

The grey line behind the circles. Its X/Width derive from the gallery so
the line starts/ends exactly on the first/last circle center.

| Property | Value |
|----------|-------|
| X | `galStepper.X + galStepper.Width / (CountRows(colStatus) * 2)` |
| Y | `recUInfoTitleBorder.Y + recUInfoTitleBorder.Height + 36` |
| Width | `galStepper.Width * (CountRows(colStatus) - 1) / CountRows(colStatus)` |
| Height | `2` |
| Fill | `gblTheme.BorderStrong` |
| BorderThickness | `0` |

## Step 4 — Stepper fill (`recStepTrackFill`)

The maroon portion up to the current step. **This is the Dataverse-aware
part** — width is driven by `currentStatusIdx`:

| Property | Value |
|----------|-------|
| X | `recStepTrack.X` |
| Y | `recStepTrack.Y` |
| Width | `recStepTrack.Width * Max(currentStatusIdx - 1, 0) / (CountRows(colStatus) - 1)` |
| Height | `2` |
| Fill | `gblTheme.Maroon` |
| BorderThickness | `0` |

`Max(currentStatusIdx - 1, 0)` keeps the width at 0 (not negative) when the
status is blank/unknown.

---

## Step 5 — Stepper gallery (`galStepper`)

| Property | Value |
|----------|-------|
| X | `28` |
| Y | `recStepTrack.Y - 24` |
| Width | `Parent.Width - 56` |
| Height | `64` |
| TemplateSize | `Self.Width / CountRows(colStatus)` |
| TemplatePadding | `0` |
| ShowScrollbar | `false` |
| BorderThickness | `0` |
| Items | `colStatus` |

> **Tree order:** drag `galStepper` to sit **above** `recStepTrack` (and
> `recStepTrackFill`) in the tree so the circles render on top of the line.

## Step 6 — Stepper row template (circle + label)

Enter the template (chevron next to `galStepper`).

**Circle** — Insert → **Icons → Circle**:

| Property | Value |
|----------|-------|
| Name | `circStep` |
| X | `(Parent.TemplateWidth - Self.Width) / 2` |
| Y | `16` |
| Width | `18` |
| Height | `18` |
| BorderThickness | `2` |
| BorderColor | `If(ThisItem.Order <= currentStatusIdx, gblTheme.Maroon, gblTheme.BorderStrong)` |
| Fill | `If(ThisItem.Order <= currentStatusIdx, gblTheme.Maroon, gblTheme.Surface)` |

**Label** — still in the template, Insert → **Label**:

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
| Color | `If(ThisItem.Order <= currentStatusIdx, gblTheme.Maroon, gblTheme.Ink3)` |
| FontWeight | `If(ThisItem.Order = currentStatusIdx, FontWeight.Bold, FontWeight.Normal)` |
| Font | `gblTheme.FontFamily` |

Click outside to exit the template. The stepper now lights up to the
record's `Project Status`.

---

## Step 7 — Insert & bind the Edit form (`frmInfo`)

Inside `conSectionInfo`, Insert → **Edit form**. Set:

| Property | Value |
|----------|-------|
| Name | `frmInfo` |
| DataSource | `Projects` |
| Item | `selectedUC` |
| DefaultMode | `FormMode.Edit` |
| X | `28` |
| Y | `galStepper.Y + galStepper.Height + 24` |
| Width | `Parent.Width - 56` |
| Height | `560` |
| Columns | `2` |
| BorderColor | `gblTheme.Border` |

Turn **Snap to columns = On** (right pane) so cards auto-flow into the two
columns. (The form manages each card's X/Y; you control card **Width** and
**Height** and the inner control.)

## Step 8 — Pick the fields (Edit fields → in this order)

Open **Edit fields** and add exactly these 8 `Projects` columns, in order:

1. `Use Case Name`
2. `Project Problem Statement`
3. `Description of AI Solution`
4. `Type of Use Case`
5. `Project Status`
6. `Business Hierarchy`  *(the lookup)*
7. `Estimated Completion Time`
8. `Output Refresh Frequency`

Remove any other auto-added cards. *(Contacts, funds, governance, etc. live
on their own tabs — see [guide 14](14-srcdetail-dataverse-guide.md).)*

## Step 9 — Card-by-card spec

For each card: select it, set **DataCardWidth**/**Height**, and (where
noted) adjust the inner control. Choice and lookup columns **auto-generate
the correct control** — don't replace them; the default `Update` is right.

| # | Card (field) | DataCardWidth | Height | Inner control + key settings | Card `Update` (default unless noted) |
|---|---|---|---|---|---|
| 1 | Use Case Name | `frmInfo.Width` (full) | `90` | Text input | *(default)* |
| 2 | Project Problem Statement | `frmInfo.Width` (full) | `150` | Text input → `Mode = TextMode.MultiLine` | *(default)* |
| 3 | Description of AI Solution | `frmInfo.Width` (full) | `150` | Text input → `Mode = TextMode.MultiLine` | *(default)* |
| 4 | Type of Use Case | `frmInfo.Width / 2` (half) | `90` | **Auto choice combo box** | `DataCardValue.Selected` |
| 5 | Project Status | `frmInfo.Width / 2` (half) | `90` | **Auto choice combo box** | `DataCardValue.Selected` |
| 6 | Business Hierarchy | `frmInfo.Width / 2` (half) | `90` | **Auto lookup combo box** (see cascade option below) | `DataCardValue.Selected` |
| 7 | Estimated Completion Time | `frmInfo.Width / 2` (half) | `90` | Date picker | `DataCardValue.SelectedDate` |
| 8 | Output Refresh Frequency | `frmInfo.Width / 2` (half) | `90` | **Auto choice combo box** | `DataCardValue.Selected` |

To set a card full-width: select the card → in the right pane set
**Width** to `frmInfo.Width` (it pushes the next card to a new row). For
multiline: select the card → unlock (gear → Advanced → Unlock) → select the
inner `DataCardValue` text input → set `Mode = TextMode.MultiLine` → raise
the card `Height`.

> **Status card — no code/label hack anymore.** Because `Project Status` is
> a real Dataverse choice, the auto combo box shows the labels and writes
> the choice on save. Leave `Update = DataCardValue.Selected`. The stepper
> advances after Save via `OnSuccess` (Step 11) → `currentStatusIdx`
> recomputes.

### Optional — SBU → LOB cascade on the Business Hierarchy card

If you'd rather pick **SBU then LOB** than the combined `SBU/LOB` key:

1. Select the Business Hierarchy card → unlock → delete the default combo
   box (`DataCardValue`).
2. Add the SBU list once in **`srcDetail.OnVisible`**:
   ```powerfx
   If(IsEmpty(colSBUEdit),
       ClearCollect(colSBUEdit, Distinct('Business Hierarchy', 'Strategic Business Unit').Value)
   )
   ```
3. Inside the card, Insert two **Dropdowns**:

   | Control | X | Y | Width | Height | Items | Default |
   |---|---|---|---|---|---|---|
   | `ddSBU2` | `12` | `36` | `(Parent.Width - 36) / 2` | `36` | `colSBUEdit` | `selectedUC.'Business Hierarchy'.'Strategic Business Unit'` |
   | `ddLOB2` | `ddSBU2.X + ddSBU2.Width + 12` | `36` | `ddSBU2.Width` | `36` | `Distinct(Filter('Business Hierarchy', 'Strategic Business Unit' = ddSBU2.Selected.Value), 'Line of Business')` | `selectedUC.'Business Hierarchy'.'Line of Business'` |

   *(`Y = 36` leaves room for the card's label above; adjust to match the
   other cards' inner input Y.)*

4. Set the **card `Update`** (resolve the pair to the lookup row):
   ```powerfx
   LookUp('Business Hierarchy',
          'Strategic Business Unit' = ddSBU2.Selected.Value
          && 'Line of Business' = ddLOB2.Selected.Value)
   ```

> **How this `Update` works (and why it's a record, not text).** This card
> is bound to the **Business Hierarchy lookup** column. On `SubmitForm`, the
> form writes the card's `Update` value into that lookup — and a lookup
> column requires a **record from the related table**, not text. Your two
> dropdowns only hold text (`"Capital Markets"`, `"Equities"`), so neither
> one *is* the value; the `LookUp(...)` **reassembles the actual Business
> Hierarchy row** from the SBU + LOB pair, and that row is what gets saved.
> `Update` is evaluated at save time, so it always reads the current
> selections — no extra variable needed.
>
> **If `Update` shows a red error:**
> 1. It likely still references the **deleted** `DataCardValue` (e.g.
>    `DataCardValue.Selected`). Replace the whole property with the
>    `LookUp(...)` above.
> 2. **`'Business Hierarchy'` must be an added data source** (Data → Add
>    data) for `LookUp('Business Hierarchy', …)` to resolve. (If your
>    dropdowns already show real SBU/LOB values, it is.)
> 3. Confirm the column names `'Strategic Business Unit'` /
>    `'Line of Business'` via IntelliSense.
>
> Quick test: a temporary label set to
> `LookUp('Business Hierarchy', 'Strategic Business Unit' = ddSBU2.Selected.Value && 'Line of Business' = ddLOB2.Selected.Value).'Business Hierarchy Key'`
> should show the `SBU/LOB` key — then `Update` resolves too.

---

## Step 10 — Saving: Submit Assessment + Discard Changes (action bar)

One save mechanism, in the action bar — **no per-section Save buttons, and
no bottom submit zone.**

**Delete `conSubmitZone`** (the empty 60-px bottom strip from `08` Step
15 — scaffolding for an older single-submit design). Give back its space:
wherever a height subtracted `60` for it, drop the `- 60` (e.g.
`conSectionInfo.Height = Parent.Height - 48`).

You already have two action-bar buttons. Repurpose them so they act on the
**currently-visible section's form**:

**`btnSubmit` — "Submit Assessment"** — commits the open section's edits to
Dataverse:

| Property | Value |
|----------|-------|
| Text | `"Submit Assessment"` |
| OnSelect | `Switch(currentSection, "Info", SubmitForm(frmInfo) /* , "Contacts", SubmitForm(frmContacts), … add a branch per section as you build it */ )` |
| DisplayMode | `If(Switch(currentSection, "Info", frmInfo.Unsaved, false), DisplayMode.Edit, DisplayMode.Disabled)` |

**`btnSaveDraft` → rename to `btnDiscard`, Text "Discard Changes"** —
reverts unsaved edits in the open section's form:

| Property | Value |
|----------|-------|
| Text | `"Discard Changes"` |
| OnSelect | `Switch(currentSection, "Info", ResetForm(frmInfo) /* , per section as built */ )` |
| DisplayMode | `If(Switch(currentSection, "Info", frmInfo.Unsaved, false), DisplayMode.Edit, DisplayMode.Disabled)` |

Notes:

- **Add a `Switch` branch per section only once that section's form
  exists** — a branch referencing a non-existent `frm…` errors. For now
  (Info only), each is just the single `"Info"` branch.
- **Discard only reverts *unsaved* edits** (changes since the last Submit /
  since the record loaded). It can't undo a Submit that already wrote to
  Dataverse.
- Both buttons disable when there's nothing pending (`…Unsaved = false`).
- **Scope — it saves only the OPEN tab, on purpose.** The `Switch` runs the
  one branch for `currentSection`, so Submit writes only the visible
  section's form. This is intentional and *safer* than submitting every
  form: a form on a tab you've **never opened** may not have rendered/loaded
  its values yet, and submitting it could write blanks and **wipe data**.
  So **save as you go** — Submit each tab before leaving it (the enabled
  state flags when the open tab has unsaved edits). Don't try to
  `SubmitForm` all tabs from one button; if you ever need a single
  save-everything, use one combined form or `Patch`, not many `SubmitForm`s.
  To stop users from *accidentally* losing edits when they switch tabs, add
  the **unsaved-changes guard** (Save / Discard / Cancel popup) from
  [`18-srcdetail-unsaved-guard.md`](18-srcdetail-unsaved-guard.md).
- *Semantic note:* here **Submit Assessment is the save** (it writes the
  open section to Dataverse). If you later want a separate "finalize the
  whole intake" action (lock it / set a submitted status / trigger the
  `BOA_SubmitAssessment` flow, README v3), that's an additional action on
  top of this — not the same button.

## Step 11 — Form behavior (`frmInfo`)

Select `frmInfo`:

| Property | Value |
|----------|-------|
| OnSuccess | `Set(selectedUC, frmInfo.LastSubmit)` |
| OnFailure | `Notify("Couldn't save: " & frmInfo.Error, NotificationType.Error)` |

`OnSuccess` refreshes `selectedUC` from the saved row, so the rail-head pill
and the **stepper advance the instant Save completes**.

---

## Step 12 — Sanity check

From `srcList`, open a use case:

- [ ] Title "Use Case Info" + maroon underline.
- [ ] Stepper: 7 circles + labels; filled maroon up to the record's status;
      the fill line ends at the current circle.
- [ ] Form below: Name / Problem Statement / AI Solution span full width;
      Type, Status, Business Hierarchy, Est. Completion, Refresh sit in two
      columns.
- [ ] Status/Type/Refresh show **labels** in a combo box (not codes).
- [ ] The action bar's **Submit Assessment** and **Discard Changes** are
      disabled until you edit a field; editing enables both. (No per-section
      Save buttons; no bottom submit zone.)
- [ ] Change Status → **Submit Assessment** → row saves to Dataverse and the
      **stepper advances**.
- [ ] Edit a field → **Discard Changes** → the field reverts to its saved
      value (only unsaved edits are discarded).
- [ ] Toggle the rail — both form columns reflow; no card clips.

> Stepper didn't move after Save? Confirm `frmInfo.OnSuccess =
> Set(selectedUC, frmInfo.LastSubmit)` and that `currentStatusIdx` is in
> `App.Formulas` matching on `DVStatus` (guide 13).
