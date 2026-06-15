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

This guide is **fully written out — there is no "repeat for the others."**
Each of the four toggle cards is spelled out line by line, including the
controls **inside** the card and which ones to leave alone.

> **All control names are fixed — use these exact names.** When a step says
> "rename to X," type X exactly so the later formulas resolve.
>
> | Thing | Name |
> |---|---|
> | Section container (already exists) | `conSectionTech` |
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
  `Visible = (currentSection = "Tech")` ([guide 02](02-build-guide.md)).
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
   the column the toggle binds to.

Don't proceed until all four Yes/No columns exist with these exact names.

---

## Step 1 — Section title (`lblTechTitle` + underline)

Inside `conSectionTech`, Insert → **Label**. Rename it `lblTechTitle`. Set:

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

Insert → **Rectangle** (the underline). Rename it `recTechTitleBorder`. Set:

| Property | Value |
|----------|-------|
| X | `28` |
| Y | `lblTechTitle.Y + lblTechTitle.Height + 4` |
| Width | `Parent.Width - 56` |
| Height | `2` |
| Fill | `gblTheme.Maroon` |
| BorderThickness | `0` |

---

## Step 2 — Create the Edit form (`frmTech`)

Inside `conSectionTech`, Insert → **Edit form**. With the new form selected,
set these in the right pane / formula bar:

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

In the right pane, set **Snap to columns = On**. With `Columns = 2`, fields
flow two per row automatically — so the four toggle cards form the 2×2 grid,
and the fifth (made full-width in Step 9) drops onto its own row.

---

## Step 3 — Add the five fields, in order

With `frmTech` selected, open **Edit fields** (right pane) → **Add field**.
Add exactly these five `Projects` columns, **in this order** (order = layout
order):

1. `Code Review`
2. `Algorithm Review Completed`
3. `Standard Folder Structure Compliant`
4. `Model Monitoring`
5. `AI Solution Performance`

If **Edit fields** auto-added any other columns, remove them so only these
five remain. Close the Edit fields pane.

Studio just created **five data cards**, one per field, named like
`Code Review_DataCard1`. The next steps configure each card.

---

## Step 4 — What's inside a data card (read once)

Every data card is a small container with up to four child controls. Expand
a card in the **Tree view** (left panel) to see them:

| Child (auto-name) | What it is | What we do with it |
|---|---|---|
| `DataCardKey…` | The **caption label** (shows the field name) | Rename + reposition + set its `Text` |
| `DataCardValue…` | The **input control** — for a Yes/No it's a **combo box** | **Delete it**, replace with a Toggle |
| `StarVisible…` | The **required "\*" asterisk** label | **Leave it alone** — don't rename, don't move |
| `ErrorMessage…` | The **validation error** label | **Leave it alone** — we use no validation here |

> **Why the input is a combo box, not a toggle.** On current Power Apps, a
> Dataverse **Yes/No** column renders its data card as a **combo box** (Yes /
> No dropdown), not a Toggle. That's a Studio default, not a data problem.
> Steps 5–8 delete each combo box and drop in a real Toggle.
>
> The number suffix on these children (`DataCardKey1`, `DataCardValue2`, …)
> varies per card — identify them by the **base name**, after selecting the
> specific card you're working on.
>
> **Unlock first.** You can't edit a card's children until you unlock the
> card: select the card → right pane → **Advanced → Unlock** (the padlock).

---

## Step 5 — Card 1: Code Review

1. In the Tree view, find the Code Review data card (auto-named
   `Code Review_DataCard1` or similar). **Rename it `cardCodeReview`.**
2. Select `cardCodeReview` → right pane → **Advanced → Unlock**.
3. With `cardCodeReview` selected, set:
   | Property | Value |
   |---|---|
   | Height | `64` |
   | BorderThickness | `1` |
   | BorderColor | `gblTheme.Border` |
   | Fill | `White` |
   *(Leave **Width** as-is — Snap to columns sizes it to one column / half the form.)*
4. Expand `cardCodeReview`. Select its caption label (`DataCardKey…`).
   **Rename it `lblCodeReview`.** Set:
   | Property | Value |
   |---|---|
   | Text | `"Code Review"` |
   | X | `14` |
   | Y | `(Parent.Height - Self.Height) / 2` |
   | Width | `Parent.Width - 80` |
   | Size | `13` |
   | Color | `gblTheme.Ink` |
   | Font | `gblTheme.FontFamily` |
5. Select the input control (`DataCardValue…`, the **combo box**) and
   **delete it.**
6. With `cardCodeReview` selected, Insert → **Input → Toggle**. The toggle
   lands inside the card. **Rename it `tglCodeReview`.** Set:
   | Property | Value |
   |---|---|
   | X | `Parent.Width - Self.Width - 14` |
   | Y | `(Parent.Height - Self.Height) / 2` |
   | Width | `50` |
   | Height | `20` |
   | TrueFill | `gblTheme.Maroon` |
   | FalseFill | `gblTheme.BorderStrong` |
   | HandleFill | `White` |
   | Default | `If(selectedUC.'Code Review' = "Yes", true, false)` |
7. Select `cardCodeReview` again → set its **Update** property to
   `If(tglCodeReview.Value, "Yes", "No")`. *(This clears the red error that
   appeared when you deleted the combo box in step 5, and writes the value
   back in the "Yes"/"No" form the column stores.)*
8. **Do not touch** `StarVisible…` or `ErrorMessage…` in this card.

---

## Step 6 — Card 2: Algorithm Review

1. In the Tree view, find the Algorithm Review Completed data card
   (`Algorithm Review Completed_DataCard…`). **Rename it `cardAlgoReview`.**
2. Select `cardAlgoReview` → **Advanced → Unlock**.
3. With `cardAlgoReview` selected, set:
   | Property | Value |
   |---|---|
   | Height | `64` |
   | BorderThickness | `1` |
   | BorderColor | `gblTheme.Border` |
   | Fill | `White` |
4. Expand `cardAlgoReview`. Select its caption label (`DataCardKey…`).
   **Rename it `lblAlgoReview`.** Set:
   | Property | Value |
   |---|---|
   | Text | `"Algorithm Review"` |
   | X | `14` |
   | Y | `(Parent.Height - Self.Height) / 2` |
   | Width | `Parent.Width - 80` |
   | Size | `13` |
   | Color | `gblTheme.Ink` |
   | Font | `gblTheme.FontFamily` |
5. Select the input control (`DataCardValue…`, the **combo box**) and
   **delete it.**
6. With `cardAlgoReview` selected, Insert → **Input → Toggle**.
   **Rename it `tglAlgoReview`.** Set:
   | Property | Value |
   |---|---|
   | X | `Parent.Width - Self.Width - 14` |
   | Y | `(Parent.Height - Self.Height) / 2` |
   | Width | `50` |
   | Height | `20` |
   | TrueFill | `gblTheme.Maroon` |
   | FalseFill | `gblTheme.BorderStrong` |
   | HandleFill | `White` |
   | Default | `If(selectedUC.'Algorithm Review Completed' = "Yes", true, false)` |
7. Select `cardAlgoReview` again → set its **Update** property to
   `If(tglAlgoReview.Value, "Yes", "No")`.
8. **Do not touch** `StarVisible…` or `ErrorMessage…` in this card.

---

## Step 7 — Card 3: Folder Structure

1. In the Tree view, find the Standard Folder Structure Compliant data card
   (`Standard Folder Structure Compliant_DataCard…`). **Rename it
   `cardFolderStructure`.**
2. Select `cardFolderStructure` → **Advanced → Unlock**.
3. With `cardFolderStructure` selected, set:
   | Property | Value |
   |---|---|
   | Height | `64` |
   | BorderThickness | `1` |
   | BorderColor | `gblTheme.Border` |
   | Fill | `White` |
4. Expand `cardFolderStructure`. Select its caption label (`DataCardKey…`).
   **Rename it `lblFolderStructure`.** Set:
   | Property | Value |
   |---|---|
   | Text | `"Folder Structure adheres to standards"` |
   | X | `14` |
   | Y | `(Parent.Height - Self.Height) / 2` |
   | Width | `Parent.Width - 80` |
   | Size | `13` |
   | Color | `gblTheme.Ink` |
   | Font | `gblTheme.FontFamily` |
5. Select the input control (`DataCardValue…`, the **combo box**) and
   **delete it.**
6. With `cardFolderStructure` selected, Insert → **Input → Toggle**.
   **Rename it `tglFolderStructure`.** Set:
   | Property | Value |
   |---|---|
   | X | `Parent.Width - Self.Width - 14` |
   | Y | `(Parent.Height - Self.Height) / 2` |
   | Width | `50` |
   | Height | `20` |
   | TrueFill | `gblTheme.Maroon` |
   | FalseFill | `gblTheme.BorderStrong` |
   | HandleFill | `White` |
   | Default | `If(selectedUC.'Standard Folder Structure Compliant' = "Yes", true, false)` |
7. Select `cardFolderStructure` again → set its **Update** property to
   `If(tglFolderStructure.Value, "Yes", "No")`.
8. **Do not touch** `StarVisible…` or `ErrorMessage…` in this card.

---

## Step 8 — Card 4: Model Monitoring

1. In the Tree view, find the Model Monitoring data card
   (`Model Monitoring_DataCard…`). **Rename it `cardModelMonitoring`.**
2. Select `cardModelMonitoring` → **Advanced → Unlock**.
3. With `cardModelMonitoring` selected, set:
   | Property | Value |
   |---|---|
   | Height | `64` |
   | BorderThickness | `1` |
   | BorderColor | `gblTheme.Border` |
   | Fill | `White` |
4. Expand `cardModelMonitoring`. Select its caption label (`DataCardKey…`).
   **Rename it `lblModelMonitoring`.** Set:
   | Property | Value |
   |---|---|
   | Text | `"Model Monitoring in place"` |
   | X | `14` |
   | Y | `(Parent.Height - Self.Height) / 2` |
   | Width | `Parent.Width - 80` |
   | Size | `13` |
   | Color | `gblTheme.Ink` |
   | Font | `gblTheme.FontFamily` |
5. Select the input control (`DataCardValue…`, the **combo box**) and
   **delete it.**
6. With `cardModelMonitoring` selected, Insert → **Input → Toggle**.
   **Rename it `tglModelMonitoring`.** Set:
   | Property | Value |
   |---|---|
   | X | `Parent.Width - Self.Width - 14` |
   | Y | `(Parent.Height - Self.Height) / 2` |
   | Width | `50` |
   | Height | `20` |
   | TrueFill | `gblTheme.Maroon` |
   | FalseFill | `gblTheme.BorderStrong` |
   | HandleFill | `White` |
   | Default | `If(selectedUC.'Model Monitoring' = "Yes", true, false)` |
7. Select `cardModelMonitoring` again → set its **Update** property to
   `If(tglModelMonitoring.Value, "Yes", "No")`.
8. **Do not touch** `StarVisible…` or `ErrorMessage…` in this card.

> **Why the `If(... = "Yes", true, false)` wrapper, and `TrueFill` not
> `FillSelected`.** Two Dataverse/Toggle quirks this handles:
>
> 1. **The column value is `"Yes"`/`"No"`, not a real Boolean.** Dataverse
>    surfaces this column's value as the text `"Yes"`/`"No"`, but a Toggle's
>    `Default` only accepts `true`/`false`. So we **read** with
>    `If(selectedUC.'<column>' = "Yes", true, false)` (text → boolean) and
>    **write** the card's `Update` with `If(<toggle>.Value, "Yes", "No")`
>    (boolean → text), so the value round-trips in the form the column stores.
> 2. **The classic Toggle has no `FillSelected`.** Its colour properties are
>    `TrueFill` (the on / right state), `FalseFill` (off / left), `HandleFill`
>    (the knob), plus `TrueHoverFill` / `FalseHoverFill`. So the maroon
>    "on" colour goes on **`TrueFill`**, not `FillSelected`.

---

## Step 9 — Card 5: Performance Metrics

This card's input is a **text input** (not a combo box), so you **keep** it —
don't delete anything.

1. In the Tree view, find the AI Solution Performance data card
   (`AI Solution Performance_DataCard…`). **Rename it `cardPerfMetrics`.**
2. Select `cardPerfMetrics` → set:
   | Property | Value |
   |---|---|
   | Width | `frmTech.Width` *(full width → it moves to its own row below the grid)* |
   | Height | `150` |
3. Select `cardPerfMetrics` → **Advanced → Unlock**.
4. Expand `cardPerfMetrics`. Select its caption label (`DataCardKey…`).
   **Rename it `lblPerfMetrics`.** Set `Text = "Performance Metrics"`.
   *(Leave its position — label-on-top is correct for this card.)*
5. Select the input control (`DataCardValue…`, the **text input**).
   **Rename it `txtPerfMetrics`.** Set `Mode = TextMode.MultiLine`.
6. **Leave the card's `Update` at its default** (it already writes
   `txtPerfMetrics.Text`).
7. **Do not touch** `StarVisible…` or `ErrorMessage…` in this card.

---

## Step 10 — Form behavior (`frmTech`)

Select `frmTech` and set:

| Property | Value |
|----------|-------|
| OnSuccess | `Set(selectedUC, frmTech.LastSubmit)` |
| OnFailure | `Notify("Couldn't save: " & frmTech.Error, NotificationType.Error)` |

`OnSuccess` refreshes `selectedUC` from the saved row so the toggles (and any
other tab reading these flags) reflect the saved state immediately.

---

## Step 11 — Wire the action bar (add the "Tech" branch)

This tab saves through the **same** action-bar buttons as Info — you just add
a `"Tech"` branch to each `Switch` ([guide 17](17-srcdetail-info-tab-dataverse.md)
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
> keeps **save-as-you-go** consistent: Submit writes only the **open** tab.

---

## Step 12 — Sanity check

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

**If something's wrong:**

- **Toggle won't save** → the card's `Update` isn't writing the toggle. It
  must be `If(<toggle>.Value, "Yes", "No")`, e.g.
  `cardCodeReview.Update = If(tglCodeReview.Value, "Yes", "No")`.
- **Toggle always off / red error on `Default`** → the value is `"Yes"`/`"No"`
  text, not a Boolean. `Default` must be
  `If(selectedUC.'<column>' = "Yes", true, false)`, and the card `Update` must
  be `If(<toggle>.Value, "Yes", "No")`.
- **Toggle's "on" colour won't set** → there is no `FillSelected` on the
  classic Toggle. Set **`TrueFill`** (= `gblTheme.Maroon`) for the on state.
- **Still a combo box** → you didn't delete `DataCardValue…` and insert a
  Toggle (Steps 5–8, step 5–6). Yes/No cards never auto-generate a toggle.
- **Card shows a red error after deleting the combo box** → you haven't set
  that card's `Update` to its toggle's `.Value` yet (step 7 of that card).
