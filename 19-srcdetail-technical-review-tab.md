# BOA — srcDetail "Technical Review" Tab on Dataverse (Fully Specified)

Build the **Technical Review** section end-to-end against **Dataverse**.
The tab is four **Yes/No** flags shown as plain toggles — two per line in a
2×2 grid — followed by one full-width **Performance Metrics** text box:

```
┌─────────────────────────┐ ┌─────────────────────────┐
│ Code Review        [on] │ │ Algorithm Review   [on] │
└─────────────────────────┘ └─────────────────────────┘
┌─────────────────────────┐ ┌─────────────────────────┐
│ Folder Structure   [on] │ │ Model Monitoring  [off] │
└─────────────────────────┘ └─────────────────────────┘
┌───────────────────────────────────────────────────────┐
│ Performance Metrics (multiline)                       │
└───────────────────────────────────────────────────────┘
```

No subtext, no per-row "Mark complete" button — each tile is just a **label
+ a toggle** bound to one Dataverse Yes/No column. We build it as an **Edit
form** (`frmTech`) so it saves through the same action-bar **Submit
Assessment / Discard Changes** mechanism as the Info tab — see
[guide 17](17-srcdetail-info-tab-dataverse.md) Step 10.

> **Every control name is fixed in this guide — don't invent your own.** When
> a step says rename a card/label/toggle, use the **exact** name given so the
> formulas in later steps resolve. The full name list:
>
> | Thing | Name |
> |---|---|
> | Section container | `conSectionTech` |
> | Title label / underline | `lblTechTitle` / `recTechTitleBorder` |
> | Edit form | `frmTech` |
> | Card 1 — Code Review | card `cardCodeReview`, label `lblCodeReview`, toggle `tglCodeReview` |
> | Card 2 — Algorithm Review | card `cardAlgoReview`, label `lblAlgoReview`, toggle `tglAlgoReview` |
> | Card 3 — Folder Structure | card `cardFolderStructure`, label `lblFolderStructure`, toggle `tglFolderStructure` |
> | Card 4 — Model Monitoring | card `cardModelMonitoring`, label `lblModelMonitoring`, toggle `tglModelMonitoring` |
> | Card 5 — Performance Metrics | card `cardPerfMetrics`, label `lblPerfMetrics`, input `txtPerfMetrics` |

**Prerequisites** (already covered elsewhere):
- `Projects` added as a data source.
- `selectedUC` is a `Projects` record; `App.OnStart` seeds
  `Set(selectedUC, Defaults(Projects))`.
- The section container `conSectionTech` exists with
  `Visible = (currentSection = "Tech")` ([guide 02](02-build-guide.md),
  *conSectionTech*).
- The action-bar buttons `btnSubmit` / `btnDiscard` exist
  ([guide 17](17-srcdetail-info-tab-dataverse.md) Step 10).

---

## Step 0 — Create / confirm the four Yes/No columns

These toggles bind to four `Projects` columns. Each **must be a Yes/No
(boolean) column** in Dataverse with the **exact display name** below:

| Tile label (UI) | `Projects` column (exact display name) | Type |
|---|---|---|
| Code Review | `Code Review` | Yes/No |
| Algorithm Review | `Algorithm Review Completed` | Yes/No |
| Folder Structure adheres to standards | `Standard Folder Structure Compliant` | Yes/No |
| Model Monitoring in place | `Model Monitoring` | Yes/No |
| Performance Metrics (text box) | `AI Solution Performance` | Multiline text |

**Do this now, before building anything:**

1. Open the `Projects` table in Dataverse.
2. `Code Review`, `Algorithm Review Completed`, `Model Monitoring`,
   `AI Solution Performance` already exist from the schema
   ([§2.5](09-dataverse-schema.md#25-governance--model--review)). Confirm the
   first three are type **Yes/No** and the last is **Multiple lines of text**.
   If any Yes/No one shows as Choice or Text, change it to Yes/No here first.
3. **Create the folder-structure column.** The schema's
   `Standard Folder Structure` is a **URL** field (a link, not a flag) — leave
   it as-is. Add a **new** column: **New column → Display name
   `Standard Folder Structure Compliant`, Data type Yes/No → Save**. This is
   the column the toggle binds to. (Accept whatever schema name Dataverse
   auto-assigns, e.g. `cr_standardfolderstructurecompliant`; the form uses the
   display name.)

Don't proceed until all four Yes/No columns exist with these exact names — a
toggle card only generates for a Yes/No column, and later formulas reference
these names literally.

---

## Step 1 — Section title (`lblTechTitle` + underline)

Inside `conSectionTech`, same title pattern as the Info tab
([guide 17](17-srcdetail-info-tab-dataverse.md) Steps 1–2).

**`lblTechTitle`:**

| Property | Value |
|----------|-------|
| Text | `"Technical Review"` |
| X | `28` |
| Y | `24` |
| Width | `300` |
| Height | `24` |
| Size | `16` |
| FontWeight | `FontWeight.Bold` |
| Color | `gblTheme.Maroon` |
| Font | `gblTheme.FontFamily` |

**`recTechTitleBorder`** (underline):

| Property | Value |
|----------|-------|
| X | `28` |
| Y | `lblTechTitle.Y + lblTechTitle.Height + 4` |
| Width | `Parent.Width - 56` |
| Height | `2` |
| Fill | `gblTheme.Maroon` |
| BorderThickness | `0` |

---

## Step 2 — Insert & bind the Edit form (`frmTech`)

Inside `conSectionTech`, Insert → **Edit form**. Set:

| Property | Value |
|----------|-------|
| Name | `frmTech` |
| DataSource | `Projects` |
| Item | `selectedUC` |
| DefaultMode | `FormMode.Edit` |
| X | `28` |
| Y | `recTechTitleBorder.Y + recTechTitleBorder.Height + 20` |
| Width | `Parent.Width - 56` |
| Height | `420` |
| Columns | `2` |
| BorderColor | `gblTheme.Border` |

Turn **Snap to columns = On** (right pane) so the four toggle cards flow two
per row. The fifth card (Performance Metrics) is set to **full width** in
Step 4, so it drops onto its own row below the grid.

## Step 3 — Pick the fields (Edit fields → in this order)

Open **Edit fields** and add exactly these five `Projects` columns, **in
this order** (order = layout order with Snap to columns on):

1. `Code Review`
2. `Algorithm Review Completed`
3. `Standard Folder Structure Compliant`
4. `Model Monitoring`
5. `AI Solution Performance`

Remove any other auto-added cards.

> **Heads-up — the Yes/No cards come in as combo boxes, not toggles.**
> On current Power Apps, a Dataverse **Yes/No (boolean)** column
> auto-generates a **combo box** (a Yes / No dropdown) as its
> `DataCardValue`, *not* a Toggle — even though the column is correctly typed
> Yes/No in Dataverse. This is a Studio default, not a data problem. Step 4a
> **swaps each combo box for a Toggle** to get the mockup look. (The
> Multiline `AI Solution Performance` card *does* generate the right control
> — a multi-line Text input — so leave card 5 alone.)

## Step 4 — Rename the five cards, then spec each

When you added the fields, Studio created five cards with auto names like
`Code Review_DataCard1`. **Rename each card** (double-click its name in the
left tree → type the new name) to the fixed names below. Do the same for
each card's inner label and value control as you reach it in 4a/4b.

| # | Field (column) | Rename card to | DataCardWidth | Height | Inner value control | Card `Update` |
|---|---|---|---|---|---|---|
| 1 | Code Review | `cardCodeReview` | `frmTech.Width / 2` | `64` | swap → toggle `tglCodeReview` (4a) | `tglCodeReview.Value` |
| 2 | Algorithm Review Completed | `cardAlgoReview` | `frmTech.Width / 2` | `64` | swap → toggle `tglAlgoReview` (4a) | `tglAlgoReview.Value` |
| 3 | Standard Folder Structure Compliant | `cardFolderStructure` | `frmTech.Width / 2` | `64` | swap → toggle `tglFolderStructure` (4a) | `tglFolderStructure.Value` |
| 4 | Model Monitoring | `cardModelMonitoring` | `frmTech.Width / 2` | `64` | swap → toggle `tglModelMonitoring` (4a) | `tglModelMonitoring.Value` |
| 5 | AI Solution Performance | `cardPerfMetrics` | `frmTech.Width` (full) | `150` | text input `txtPerfMetrics`, `Mode = TextMode.MultiLine` (4b) | *(default)* |

Cards 1–4 arrive as Yes/No **combo boxes**; 4a deletes each combo box,
inserts the named Toggle, and points the card `Update` at it. Card 5 keeps
its default text input (just rename it) and default `Update`.

To unlock a card before editing its inner controls: select it → right pane
**Advanced → Unlock**.

### 4a — Swap the combo box for a Toggle, then tile it (cards 1–4)

Each card arrives with a **Yes/No combo box** stacked under the label. Run
these **exact steps on each of the four cards**, substituting that card's
names from this table:

| Card | Card name | Label name | Toggle name | Label `Text` |
|---|---|---|---|---|
| 1 | `cardCodeReview` | `lblCodeReview` | `tglCodeReview` | `"Code Review"` |
| 2 | `cardAlgoReview` | `lblAlgoReview` | `tglAlgoReview` | `"Algorithm Review"` |
| 3 | `cardFolderStructure` | `lblFolderStructure` | `tglFolderStructure` | `"Folder Structure adheres to standards"` |
| 4 | `cardModelMonitoring` | `lblModelMonitoring` | `tglModelMonitoring` | `"Model Monitoring in place"` |

**Steps (using card 1 `cardCodeReview` as the worked example):**

1. Select `cardCodeReview` → **Advanced → Unlock**.
2. Set the card's tile look: `BorderThickness = 1`,
   `BorderColor = gblTheme.Border`, `Fill = White`.
3. Select the inner **label** (auto-named `DataCardKey…`) → **rename it to
   `lblCodeReview`** → set its properties from the **Label** table below
   (including `Text = "Code Review"`).
4. Select the inner **combo box** (auto-named `DataCardValue…`) and
   **delete it**.
5. **Insert → Input → Toggle** (classic Toggle). It lands in the card.
   **Rename it to `tglCodeReview`** → set its properties from the **Toggle**
   table below.
6. Select `cardCodeReview` again → set its **`Update`** property to
   `tglCodeReview.Value`. (Until you do, the card still references the deleted
   combo box and shows a red error.)
7. Select the card's **error message label** (auto-named `ErrorMessage…` at
   the bottom) → set `Height = 0` and `Visible = false` — a Yes/No has
   nothing to validate.

Repeat 1–7 for `cardAlgoReview`, `cardFolderStructure`, `cardModelMonitoring`
using their row from the table above.

**Label** (e.g. `lblCodeReview`) — set:

| Property | Value |
|----------|-------|
| X | `14` |
| Y | `(Parent.Height - Self.Height) / 2` |
| Width | `Parent.Width - 80` |
| Size | `13` |
| Color | `gblTheme.Ink` |
| Font | `gblTheme.FontFamily` |
| Text | the card's `Text` from the table above |

**Toggle** (e.g. `tglCodeReview`) — set:

| Property | Value |
|----------|-------|
| X | `Parent.Width - Self.Width - 14` |
| Y | `(Parent.Height - Self.Height) / 2` |
| Width | `50` |
| Height | `20` |
| FillSelected | `gblTheme.Maroon` |
| Default | `Parent.Default` |

> **`Default = Parent.Default` for all four toggles** — identical formula on
> every card. The card's own `Default` is auto-bound to its Yes/No column, so
> `Parent.Default` is the saved boolean; no `If(... = "Yes")` needed. Don't
> type the column name into the toggle — `Parent.Default` already resolves to
> the right column per card.

### 4b — Performance Metrics card (`cardPerfMetrics`)

1. Select `cardPerfMetrics` → set `DataCardWidth = frmTech.Width` (full width,
   so it drops to its own row) and `Height = 150`.
2. Unlock it → select the inner label (`DataCardKey…`) → **rename to
   `lblPerfMetrics`** → set `Text = "Performance Metrics"`.
3. Select the inner text input (`DataCardValue…`) → **rename to
   `txtPerfMetrics`** → set `Mode = TextMode.MultiLine`.

Leave the card's `Update` at its default. The label sits on top, the
multiline input fills the rest.

---

## Step 5 — Form behavior (`frmTech`)

Select `frmTech`:

| Property | Value |
|----------|-------|
| OnSuccess | `Set(selectedUC, frmTech.LastSubmit)` |
| OnFailure | `Notify("Couldn't save: " & frmTech.Error, NotificationType.Error)` |

`OnSuccess` refreshes `selectedUC` from the saved row so any other tab /
rail element reading these flags updates immediately.

---

## Step 6 — Wire the action bar (add the "Tech" branch)

This tab saves through the **same** action-bar buttons as Info — you just
add a `"Tech"` branch to each `Switch` ([guide 17](17-srcdetail-info-tab-dataverse.md)
Step 10). **No per-section Save button on this tab.**

**`btnSubmit` — "Submit Assessment"** → OnSelect:

```powerfx
Switch(currentSection,
    "Info", SubmitForm(frmInfo),
    "Tech", SubmitForm(frmTech)
    /* , add other sections as you build them */
)
```

DisplayMode (enable only when the open tab has unsaved edits):

```powerfx
If(
    Switch(currentSection, "Info", frmInfo.Unsaved, "Tech", frmTech.Unsaved, false),
    DisplayMode.Edit, DisplayMode.Disabled
)
```

**`btnDiscard` — "Discard Changes"** → OnSelect:

```powerfx
Switch(currentSection,
    "Info", ResetForm(frmInfo),
    "Tech", ResetForm(frmTech)
)
```

Use the **same `DisplayMode`** formula as `btnSubmit`.

> **Why a form + the shared action bar.** The form gives you `frmTech.Unsaved`
> (drives the button enable/disable for free), an atomic write of all four
> flags + the metrics on one Submit, and `ResetForm` for Discard. It also
> keeps **save-as-you-go** behavior consistent: Submit writes only the
> **open** tab, which is safer than submitting tabs the user never rendered
> (see guide 17 Step 10).

---

## Step 7 — Sanity check

From `srcList`, open a use case → **Technical Review**:

- [ ] Title "Technical Review" + maroon underline.
- [ ] Four toggle tiles, **two per row**; each shows a label left, a switch
      right, no subtext, no "Mark complete" button.
- [ ] Toggles load **on/off from the record** (a use case with
      `Code Review = Yes` shows that switch on, maroon).
- [ ] "Performance Metrics" multiline box spans full width on its own row
      below the grid, showing `AI Solution Performance`.
- [ ] Action bar **Submit Assessment / Discard Changes** are disabled until
      you flip a toggle or edit the text; then both enable.
- [ ] Flip a toggle → **Submit Assessment** → reopen the record: the new
      state persisted to Dataverse.
- [ ] Flip a toggle → **Discard Changes** → it snaps back to the saved value.
- [ ] Toggle the side rail — both form columns reflow; tiles stay 2-up and
      don't clip.

> Toggle won't save? Confirm the card's `Update` reads its toggle's `.Value`
> (e.g. `cardCodeReview.Update = tglCodeReview.Value`) — after the swap the
> card still points at the deleted combo box until you fix it. Toggle always
> shows off? Its `Default` isn't `Parent.Default`. Still a combo box? You
> haven't done the Step 4a swap — the Yes/No card never auto-generates a
> toggle.
