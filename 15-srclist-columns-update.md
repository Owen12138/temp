# BOA — Update the srcList Columns

Change the Use Case List gallery to this **10-column** set:

> UC ID · Name · SBU · AI Solution Owner · Project Status · Estimated
> Completion Date · Realized Value (YTD) · Last Updated · AI Enablement
> Owner · Executive Sponsor

vs. the original 9 columns this:
- **removes** the `FY` column,
- **renames** `Status` → `Project Status` and `Realized Value` →
  `Realized Value (YTD)` (left **empty** for now — logic comes later),
- **adds** Estimated Completion Date, AI Enablement Owner, Executive Sponsor.

Assumes `srcList` is on **Dataverse** ([`10-srclist-dataverse-guide.md`](10-srclist-dataverse-guide.md)),
so fields are `Projects` columns. The gallery uses Horizontal Containers
(`conGalleryHeader` for headings, `conRow` for cells), where **column
width = `FillPortions`** and **column order = tree order** — and a column's
`FillPortions` must be **identical** in the header and the row or they
misalign.

---

## The target columns (master reference)

| # | Header text | Header label | Row control | Dataverse field (row `Text`) | FillPortions |
|---|---|---|---|---|---|
| 1 | `UCID` | `lblColUCID` | `lblUCID` | `ThisItem.'Use Case ID'` | `90` |
| 2 | `Use Case Name` | `lblColName` | `lblName` | `ThisItem.'Use Case Name'` | `190` |
| 3 | `SBU` | `lblColSBU` | `lblSBU` | `ThisItem.'Business Hierarchy'.'Strategic Business Unit'` | `120` |
| 4 | `AI Solution Owner` | `lblColOwner` | `lblOwner` | `ThisItem.'AI Solution Owner Name'` | `140` |
| 5 | `Project Status` | `lblColStatus` | `conStatusCol` (pill) | *(pill — unchanged)* | `150` |
| 6 | `Est. Completion` | `lblColEstDate` | `lblEstDate` | *(date — see Part C)* | `120` |
| 7 | `Realized Value (YTD)` | `lblColValueYTD` | `lblValueYTD` | `"—"` *(placeholder)* | `120` |
| 8 | `Last Updated` | `lblColUpdated` | `lblUpdated` | *(unchanged — relative date)* | `110` |
| 9 | `AI Enablement Owner` | `lblColEnablementOwner` | `lblEnablementOwner` | `ThisItem.'AI Enablement Owner Name'` | `140` |
| 10 | `Executive Sponsor` | `lblColExecSponsor` | `lblExecSponsor` | `ThisItem.'Executive Sponsor Name'` | `140` |

> Confirm the exact Dataverse display names against your tables (esp.
> **`Estimated Completion Time`** and the two new owner fields) via
> IntelliSense — type `ThisItem.` in the formula bar. If your "estimated
> completion" column is named differently, use that name in Part C.

> **Keeping the View button?** The original had a 9th "action" column
> (`conActionCol` + `btnView`). The whole row is already clickable
> (`galUseCases.OnSelect` navigates), so the explicit View button is
> optional. Keep it as an 11th column (`FillPortions ≈ 70`) **or** delete
> `lblColAction` + `conActionCol` to give the data columns more room — your
> call. This guide leaves it out for breathing space; add it back if you
> want it.

---

## Part A — Update the header row (`conGalleryHeader`)

Select `conGalleryHeader` and work through its child labels.

1. **Delete** the FY heading `lblColFY`.
2. **Rename text** on two existing labels:
   - `lblColStatus.Text` → `"Project Status"`
   - `lblColValue` → rename the control to `lblColValueYTD`, set
     `Text` → `"Realized Value (YTD)"`
3. **Add three** new Labels inside `conGalleryHeader` (Insert → Label).
   Give each the **same shared header properties** as the others (Size
   `12`, FontWeight `Semibold`, Color `White`, Font `gblTheme.FontFamily`,
   PaddingLeft `0`, Wrap `false`):
   - `lblColEstDate` — Text `"Est. Completion"`
   - `lblColEnablementOwner` — Text `"AI Enablement Owner"`
   - `lblColExecSponsor` — Text `"Executive Sponsor"`
4. **Set `FillPortions`** on every header label per the master table.
5. **Reorder** the labels in the tree (top→bottom = left→right) to match
   the column order 1→10.

---

## Part B — Update the row template (`conRow`)

Enter the gallery template (chevron next to `galUseCases`) and edit
`conRow`'s children. Each cell's `FillPortions` **must equal its header's**.

1. **Delete** the FY cell `lblFY`.
2. **Realized Value (YTD) placeholder:** rename `lblValue` →
   `lblValueYTD` and set its `Text` to a literal dash for now:
   ```powerfx
   "—"
   ```
   (We'll replace this with the YTD rollup later — see Part C.)
3. **Keep** `lblUCID`, `lblName`, `lblSBU`, `lblOwner`, the Status pill
   wrapper `conStatusCol`, and `lblUpdated` — only their `FillPortions`
   change (per the table).
4. **Add three** new Labels inside `conRow` (Insert → Label), each with the
   **shared row-label properties** the others use (Size `12`, Font
   `gblTheme.FontFamily`, PaddingLeft `0`, `Wrap`/`VerticalAlign`/`Tooltip`
   per [`11-srclist-layout-tuning.md`](11-srclist-layout-tuning.md)):

   | Control | Text | Tooltip |
   |---|---|---|
   | `lblEstDate` | *(date — Part C)* | same as Text |
   | `lblEnablementOwner` | `ThisItem.'AI Enablement Owner Name'` | same |
   | `lblExecSponsor` | `ThisItem.'Executive Sponsor Name'` | same |

5. **Set `FillPortions`** on every cell to match the header.
6. **Reorder** `conRow`'s children in the tree to match column order 1→10.

---

## Part C — Estimated Completion Date + the YTD placeholder

**Estimated Completion Date (`lblEstDate.Text`)** — format the date column,
blank-safe:

```powerfx
If(
    IsBlank(ThisItem.'Estimated Completion Time'),
    "—",
    Text(ThisItem.'Estimated Completion Time', "mmm d, yyyy")
)
```

(Replace `'Estimated Completion Time'` with your actual column name if it
differs.)

**Realized Value (YTD) (`lblValueYTD.Text`)** — intentionally empty for now:

```powerfx
"—"
```

> **Coming later:** this will sum the current fiscal year's `Realized
> Value` from the `Value` child table, e.g.
> `Sum(Filter('Value', Project.'Use Case ID' = ThisItem.'Use Case ID' && 'Fiscal Year' = <currentFY>), 'Realized Value')`.
> Leave it as `"—"` until that logic is decided.

---

## Density note

Ten columns across ~1,146 px is **tight** (~110 px each). With wrapping on
(guide 11), longer names/owners will wrap to two lines — acceptable, but if
it feels cramped:

- Drop the **View** column (the row is clickable anyway) — already omitted here.
- Shorten headers (e.g. "AI Enablement Owner" → "AI Enablement").
- Give the long text columns (Name, the three owners) more `FillPortions`
  and trim the short ones (UCID, dates) — they're proportional, so only the
  ratios matter.

---

## Sanity check

Press **F5** on `srcList`:

- [ ] Ten columns appear in order: UC ID, Name, SBU, AI Solution Owner,
      Project Status, Est. Completion, Realized Value (YTD), Last Updated,
      AI Enablement Owner, Executive Sponsor.
- [ ] No FY column.
- [ ] AI Solution Owner, AI Enablement Owner, and Executive Sponsor each
      populate from their Projects fields.
- [ ] Est. Completion shows a formatted date (or "—" when blank).
- [ ] Realized Value (YTD) shows "—" in every row (placeholder).
- [ ] Project Status still shows the colored pill.
- [ ] Each row cell lines up under its header (FillPortions match).
- [ ] Toggle the rail — all columns reflow proportionally, no clipping.

> If a cell drifts from its header, that column's `FillPortions` differs
> between `conGalleryHeader` and `conRow` — make them identical.
