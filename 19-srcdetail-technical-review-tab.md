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

## Step 0 — Confirm the four columns are Yes/No

These toggles bind to four `Projects` columns from
[the schema §2.5](09-dataverse-schema.md#25-governance--model--review).
Each is a **Yes/No (boolean / Two Options)** column in Dataverse:

| Tile label (UI) | `Projects` column | Type |
|---|---|---|
| Code Review | `Code Review` | Yes/No |
| Algorithm Review | `Algorithm Review Completed` | Yes/No |
| Folder Structure adheres to standards | `Standard Folder Structure` | **see note** |
| Model Monitoring in place | `Model Monitoring` | Yes/No |
| Performance Metrics (text box) | `AI Solution Performance` | Multiline |

> **⚠ Folder Structure — confirm the column type first.** In the schema
> `Standard Folder Structure` is tentatively a **URL** ("link to / flag for
> the standard folder structure", marked †). A toggle needs a **Yes/No**
> column. Pick one before building this card:
> - **Recommended:** add a Yes/No column `Standard Folder Structure Compliant`
>   (or change `Standard Folder Structure` to Yes/No if no one stores a URL),
>   and bind the toggle to that. Use the name in the table below.
> - **Keep the URL:** then this isn't a toggle — drop the tile and put a URL
>   text input here instead. The other three tiles stay as toggles.
>
> The rest of this guide assumes a Yes/No column named
> `Standard Folder Structure Compliant`. Substitute your final name.

If any of the four shows as **Choice** or **Text** in Dataverse instead of
Yes/No, fix it there first — a toggle card only auto-generates for a Yes/No
column.

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
3. `Standard Folder Structure Compliant`  *(your Yes/No name from Step 0)*
4. `Model Monitoring`
5. `AI Solution Performance`

Remove any other auto-added cards.

A Yes/No column auto-generates a **Toggle** as its `DataCardValue`; a
Multiline column auto-generates a multi-line **Text input**. Don't replace
either — the default `Update` is correct.

## Step 4 — Card-by-card spec

For each card: select it, set **DataCardWidth** / **Height**, then unlock
(gear → **Advanced → Unlock**) only where you need to reposition the inner
controls. The four toggle cards share one recipe (4a); the text card is 4b.

| # | Card (field) | DataCardWidth | Height | Inner control | Card `Update` |
|---|---|---|---|---|---|
| 1 | Code Review | `frmTech.Width / 2` | `64` | Toggle (4a) | `DataCardValue1.Value` |
| 2 | Algorithm Review Completed | `frmTech.Width / 2` | `64` | Toggle (4a) | `DataCardValue2.Value` |
| 3 | Standard Folder Structure Compliant | `frmTech.Width / 2` | `64` | Toggle (4a) | `DataCardValue3.Value` |
| 4 | Model Monitoring | `frmTech.Width / 2` | `64` | Toggle (4a) | `DataCardValue4.Value` |
| 5 | AI Solution Performance | `frmTech.Width` (full) | `150` | Text input → `Mode = TextMode.MultiLine` (4b) | *(default)* |

> Each card's `Update` is set to **its own** `DataCardValue` (the inner
> toggle/input). The numbering (`DataCardValue1`…) is whatever Studio
> assigned — confirm via IntelliSense; don't assume the suffixes.

### 4a — Toggle tile layout (do this for cards 1–4)

The card auto-stacks **label on top, toggle below**. We want the mockup's
tile: a thin border, **label left, toggle right, vertically centred**, no
subtext. Unlock the card, then:

**Card (the `…_DataCard`) — make it a tile:**

| Property | Value |
|----------|-------|
| BorderThickness | `1` |
| BorderColor | `gblTheme.Border` |
| Fill | `White` |
| BorderRadius | `3` *(if available on the card; else ignore)* |

**Label (`DataCardKey` — the field title), set:**

| Property | Value |
|----------|-------|
| X | `14` |
| Y | `(Parent.Height - Self.Height) / 2` |
| Width | `Parent.Width - 80` |
| Size | `13` |
| Color | `gblTheme.Ink` |
| Font | `gblTheme.FontFamily` |
| Text | `"Code Review"` *(card 1; see label overrides below)* |

> **Label text overrides.** The `DataCardKey` defaults to the Dataverse
> display name. To match the mockup wording, set `Text` per card:
> card 1 `"Code Review"`, card 2 `"Algorithm Review"`,
> card 3 `"Folder Structure adheres to standards"`,
> card 4 `"Model Monitoring in place"`.

**Toggle (`DataCardValue`), set:**

| Property | Value |
|----------|-------|
| X | `Parent.Width - Self.Width - 14` |
| Y | `(Parent.Height - Self.Height) / 2` |
| Width | `50` |
| Height | `20` |
| FillSelected | `gblTheme.Maroon` |
| Default | *(leave the form's binding — `ThisItem.'Code Review'` etc.)* |

> **Don't touch `Default`.** The form wrote it as `ThisItem.'<column>'` when
> you added the field — that's what loads the saved value into the toggle.
> Overwriting it breaks the bind. We only restyle/reposition the toggle.

**Error message label** (auto-added at the card bottom): set
`Height = 0` / `Visible = false` to reclaim the space — a Yes/No has nothing
to validate.

### 4b — Performance Metrics card (card 5)

| Property | Value |
|----------|-------|
| DataCardWidth | `frmTech.Width` (full width → own row) |
| Height | `150` |
| Inner `DataCardValue` | `Mode = TextMode.MultiLine` |

Leave its `Update` at the default. Label sits on top (standard card),
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

> **Why a form + the shared action bar (not 4 toggles + `Patch`).** The form
> gives you `frmTech.Unsaved` (drives the button enable/disable for free),
> an atomic write of all four flags + the metrics on one Submit, and
> `ResetForm` for Discard. It also keeps **save-as-you-go** behavior
> consistent: Submit writes only the **open** tab, which is safer than
> submitting tabs the user never rendered (see guide 17 Step 10). If you'd
> rather hand-place toggles, see the alternative below — but then you own the
> `Patch`, the dirty-tracking, and the button wiring yourself.

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

> Toggle won't save? Confirm the column is **Yes/No** in Dataverse (Step 0)
> and the card `Update` reads that card's own `DataCardValue.Value`. Toggle
> always shows off? Its `Default` was overwritten — restore the form bind
> `ThisItem.'<column>'`.

---

## Alternative — manual toggles + `Patch` (no form)

If you don't want an Edit form, lay out four **Toggle** controls and one
**Text input** by hand inside `conSectionTech`, then save with one `Patch`.
You give up `frmTech.Unsaved`, so you also track "dirty" yourself.

**Each toggle** (e.g. `tglCodeReview`):

| Property | Value |
|----------|-------|
| Default | `selectedUC.'Code Review'` |
| FillSelected | `gblTheme.Maroon` |
| Width | `50` · Height `20` |

Place each in a bordered container tile with a label (same 2×2 geometry as
Step 4a). **Submit** (action-bar branch or a local button):

```powerfx
Patch(Projects, selectedUC, {
    'Code Review': tglCodeReview.Value,
    'Algorithm Review Completed': tglAlgoReview.Value,
    'Standard Folder Structure Compliant': tglFolder.Value,
    'Model Monitoring': tglMonitoring.Value,
    'AI Solution Performance': txtPerfMetrics.Text
});
Set(selectedUC, LookUp(Projects, Project = selectedUC.Project))
```

**Discard:** reset each toggle's `Default`/the text by re-seeding from
`selectedUC` (e.g. `Set(selectedUC, selectedUC)` forces the `Default`
expressions to re-read). The form approach in Steps 2–6 avoids all of this —
prefer it unless you have a specific reason not to.
