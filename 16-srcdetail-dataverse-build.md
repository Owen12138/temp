# BOA — srcDetail on Dataverse: Build from the Info Container (+ where each formula lives)

Pick up `srcDetail` **from the point where you've created the
`conSectionInfo` container** ([`08` Part 6, Step 27](08-srcdetail-guide.md))
and build the rest directly against **Dataverse**. Every command below is
tagged with **where it goes**:

- **[OnStart]** — `App.OnStart`
- **[Formulas]** — `App.Formulas` (named formula)
- **[OnVisible]** — `srcDetail.OnVisible`
- **[Inline]** — typed directly on a control's property

Assumes `srcList` is migrated (`selectedUC` is a **Projects** record) and
the Dataverse tables exist ([`09`](09-dataverse-schema.md)). For the deep
per-card field lists, this points at [`14`](14-srcdetail-dataverse-guide.md);
everything you need to decide *placement* is here.

---

## The placement rule (use this for any command)

| Put it in… | When | Can it see screen controls? |
|---|---|---|
| **[OnStart]** | Cheap, **static**, needed **app-wide**, once at launch (reference lists, typed-blank seeds) | No |
| **[Formulas]** | A **reactive, derived value reused in several places**, that references **no controls** | No |
| **[OnVisible]** | A **Dataverse query** you want off the launch path, or per-screen prep — guard with `If(IsEmpty(...))` to run once | Yes |
| **[Inline]** | Anything **bound to that control**, OR anything that **references a control** (only controls can see other controls) | — |

The decider most people miss: **`OnStart` and `Formulas` cannot reference
controls.** So a card's `Update` (reads `DataCardValue`), a button's
`SubmitForm`, a dropdown whose `Items` depend on another dropdown — all
**must be [Inline]**.

---

## Step 0 — Setup commands (do these first)

### colStatus + DVStatus — **[OnStart]**
Static 7-row reference list the stepper maps against. Already added in
[`13`](13-status-stepper-dataverse-fix.md). *Why OnStart:* tiny, static,
app-wide, must exist before any screen renders.

### selectedUC seed — **[OnStart]**
```powerfx
Set(selectedUC, Defaults(Projects))
```
Already added in [`10` Part 2B](10-srclist-dataverse-guide.md). *Why
OnStart:* a typed-blank so `selectedUC.'…'` resolves on every screen
before a row is picked.

### Value rollup — **[Formulas]**
Click `App` → **Formulas**:
```powerfx
ucValueRows = Filter(Values, Project.'Use Case ID' = selectedUC.'Use Case ID');
```
*Why Formulas:* it's **derived** from `selectedUC`, **reactive**, **reused**
by the Value tiles *and* the Value gallery, and references **no controls**.
Also **delete** the old child-row formulas (`ucGovRows`, `ucTechRows`,
`govDoneCount`, …) — governance is Projects fields now.

### Status-index — **[Formulas]** (recommended) *or* [Inline] label
The number that drives the whole stepper. It references `colStatus` +
`selectedUC` and **no controls**, and it's reused by the track fill + every
circle + every label — so it's a textbook **named formula**:
```powerfx
currentStatusIdx = Coalesce(LookUp(colStatus, DVStatus = Text(selectedUC.'Project Status'), Order), 0);
```
Then the stepper controls reference `currentStatusIdx` directly (a number —
no `Value()` needed, no hidden label).

> *Keeping the hidden `lblCurrentStatusIdx` label from guide 08 instead?*
> That's the **[Inline]** version — put the same `Coalesce(LookUp(... ))`
> in the label's `Text` and have controls read
> `Value(lblCurrentStatusIdx.Text)`. It works, but the named formula is
> cleaner. Pick one; the rest of this guide assumes `currentStatusIdx`.

### Your complete `App.Formulas` (copy-paste)

After the Dataverse migration, `App.Formulas` contains **only these two**
named formulas — the old collection ones (`filteredUseCases`, `ucGovRows`,
`ucTechRows`, `govDoneCount`, …) are gone (srcList filters inline;
governance is Projects fields):

```powerfx
// Value rows for the selected use case (srcDetail Value section)
ucValueRows = Filter(Values, Project.'Use Case ID' = selectedUC.'Use Case ID');

// Active step (1–7) for the status stepper; 0 when blank/unknown
currentStatusIdx = Coalesce(LookUp(colStatus, DVStatus = Text(selectedUC.'Project Status'), Order), 0);
```

> **An empty `App.Formulas` is fine if you're only on srcList** — srcList
> needs none (it filters inline). Add `ucValueRows` when you build the
> Value section, and `currentStatusIdx` when you wire the stepper. If you
> kept the hidden `lblCurrentStatusIdx` label instead of the named formula,
> drop `currentStatusIdx` here — only `ucValueRows` remains.

### SBU edit-picker list — **[OnVisible]** *(only if you use the SBU→LOB cascade in Step 2)*

The **SBU edit picker** is the SBU dropdown (`ddSBU2`) inside the Use Case
Info **Edit form** — the first half of the cascade that resolves the
**Business Hierarchy** lookup. `colSBUEdit` is the list of choices it
shows: the distinct Strategic Business Units from the Business Hierarchy
table. (It's separate from srcList's *filter* SBU list `colSBUOptions`,
which carries an extra **"All SBUs"** sentinel — an edit picker has no
"All"; you must choose a real SBU.)

It goes in **`srcDetail.OnVisible`** (that's where the form lives):

```powerfx
// srcDetail.OnVisible
If(IsEmpty(colSBUEdit),
    ClearCollect(colSBUEdit, Distinct('Business Hierarchy', 'Strategic Business Unit').Value)
)
```
*Why OnVisible:* it's a **Dataverse query** — keep it off app launch, and
the `If(IsEmpty(...))` makes it run once per session. (`ddSBU2.Items =
colSBUEdit`.)

> **You may not need this at all.** If you use the form's **auto lookup
> combo box** for Business Hierarchy (pick the `SBU/LOB` key directly),
> there's no cascade and no `colSBUEdit` — skip OnVisible entirely. And
> even with the cascade you can drop the collection and put
> `Distinct('Business Hierarchy', 'Strategic Business Unit')` **inline** on
> `ddSBU2.Items`; the OnVisible collection is only a load-once
> optimization. Either way, the **LOB** dropdown (`ddLOB2.Items`) stays
> **[Inline]** because it depends on the selected SBU.

---

## Step 1 — Status stepper (inside `conSectionInfo`)

Build the track + gallery exactly as in [`08` Steps 30–32](08-srcdetail-guide.md),
but everything reads `currentStatusIdx`. All of these are **[Inline]**
(control properties), because they reference controls (`galStepper`,
`recStepTrack`) and/or `ThisItem`:

- `recStepTrackFill.Width` — **[Inline]**
  ```powerfx
  recStepTrack.Width * Max(currentStatusIdx - 1, 0) / (CountRows(colStatus) - 1)
  ```
  (`Max(…,0)` keeps a blank/index-0 status from going negative.)
- `circStep.Fill` / `.BorderColor` — **[Inline]**
  ```powerfx
  If(ThisItem.Order <= currentStatusIdx, gblTheme.Maroon, gblTheme.Surface)
  ```
- The label color/weight comparisons — **[Inline]**, same `ThisItem.Order <= currentStatusIdx` pattern.
- `galStepper.Items = colStatus`, `TemplateSize = Self.Width / CountRows(colStatus)` — **[Inline]**.

---

## Step 2 — Use Case Info form (`frmInfo`)

Inside `conSectionInfo`, Insert → **Edit form**. All form wiring is
**[Inline]** (control properties):

- `frmInfo.DataSource = Projects` · `Item = selectedUC` · `DefaultMode = FormMode.Edit` · `Columns = 2` — **[Inline]**
- **Edit fields** + card layout — per [`14` Part 1](14-srcdetail-dataverse-guide.md). Choice/lookup/date columns **auto-build** the right control; leave the auto `Update` (`DataCardValue.Selected` / `.SelectedDate`). **[Inline]**

**Business Hierarchy — pick one:**

- *Auto lookup combo box (simplest):* leave the generated lookup control;
  `Update = DataCardValue.Selected`. **[Inline]**. (No OnVisible list needed.)
- *Cascading SBU → LOB:*
  - `ddSBU2.Items = colSBUEdit` — **[Inline]** (reads the [OnVisible] list from Step 0)
  - `ddLOB2.Items` — **[Inline]** *(must be inline — it depends on `ddSBU2`)*
    ```powerfx
    Distinct(Filter('Business Hierarchy', 'Strategic Business Unit' = ddSBU2.Selected.Value), 'Line of Business')
    ```
  - Business Hierarchy card `Update` — **[Inline]**
    ```powerfx
    LookUp('Business Hierarchy', 'Strategic Business Unit' = ddSBU2.Selected.Value && 'Line of Business' = ddLOB2.Selected.Value)
    ```

**Save / Cancel / refresh — all [Inline]:**
- `btnSaveInfo.OnSelect = SubmitForm(frmInfo)` · `btnCancelInfo.OnSelect = ResetForm(frmInfo)`
- `frmInfo.OnSuccess = Set(selectedUC, frmInfo.LastSubmit)` *(refreshes the record so the stepper advances)*
- `frmInfo.OnFailure = Notify("Couldn't save: " & frmInfo.Error, NotificationType.Error)`

---

## Step 3 — The other section forms (Contacts, Funds, Gov, Tech, Updates)

Each `conSection*` gets its own Edit form over `Projects`/`selectedUC` —
field lists in [`14` Part 2](14-srcdetail-dataverse-guide.md). **Every
command here is [Inline]** (form props, card Updates, Save/Cancel,
`OnSuccess = Set(selectedUC, frm….LastSubmit)`). Nothing for these belongs
in OnStart/Formulas/OnVisible — they're all control-bound.

---

## Step 4 — Value section (the child table)

- **Tiles — [Inline]:** `Sum(ucValueRows, 'Realized Value')` (realized),
  `selectedUC.'Estimated Monetary Benefit'` (estimated). They read the
  `[Formulas]` `ucValueRows`.
- **Gallery — [Inline]:** `galValueRows.Items = ucValueRows`; row `Text`
  bindings to `ThisItem.'…'` (wrap choice columns in `Text(...)`).
- **Add/Edit form `frmValue` — [Inline]:** `DataSource = Values`;
  `NewForm`/`EditForm`, `SubmitForm(frmValue)`, and the Project lookup
  card `Update = selectedUC` on insert — see [`14` Part 3](14-srcdetail-dataverse-guide.md).

---

## Placement recap

| Command | Home | Why |
|---|---|---|
| `colStatus` (+`DVStatus`) | **[OnStart]** | static, app-wide, once |
| `Set(selectedUC, Defaults(Projects))` | **[OnStart]** | typed blank, app-wide |
| `ucValueRows = Filter(Values, …)` | **[Formulas]** | derived, reactive, reused, no controls |
| `currentStatusIdx = Coalesce(LookUp(…))` | **[Formulas]** | derived, reused by stepper, no controls |
| `colSBUEdit` (cascade SBU list) | **[OnVisible]** | Dataverse query, defer + run once |
| `ddLOB2.Items` (depends on `ddSBU2`) | **[Inline]** | references a control |
| Form DataSource/Item/card Updates | **[Inline]** | control-bound |
| `SubmitForm` / `ResetForm` / `OnSuccess` | **[Inline]** | act on a control |
| Stepper fill/circle/label formulas | **[Inline]** | reference controls / `ThisItem` |
| Gallery `Items`, row `Text`, tiles | **[Inline]** | control-bound |

Rule of thumb to remember: **static & app-wide → OnStart; reactive & reused
& control-free → Formulas; Dataverse query to defer → OnVisible;
control-bound or control-referencing → Inline.**

---

## Sanity check

- [ ] Stepper advances to the right step for the open record, and on Save.
- [ ] Info form saves to Dataverse; choice cards show labels; SBU/LOB (or
      lookup) resolves.
- [ ] Each section form loads/saves its Projects fields.
- [ ] Value tiles + gallery reflect the project's `Value` rows; Add/Edit
      writes via `frmValue`.
- [ ] App launch isn't slowed by the Business Hierarchy query (it's in
      `OnVisible`, guarded — or you used the auto lookup combo box).
