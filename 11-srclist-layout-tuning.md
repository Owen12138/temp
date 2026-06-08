# BOA — srcList Layout Tuning: Full-Text Rows + Bigger Gallery

A standalone tuning pass for the **Use Case List screen (`srcList`)**, run
**after** you've built it ([`07-scrlist-guide.md`](07-scrlist-guide.md))
and — if applicable — migrated it to Dataverse
([`10-srclist-dataverse-guide.md`](10-srclist-dataverse-guide.md)). It:

1. Makes each row show its **full text** (no truncation) by letting cells
   wrap, with a smaller font so more fits per line.
2. **Shrinks the title row and filter card** (shorter, not narrower) to
   reclaim vertical space.
3. Confirms the **footer is gone** and the **gallery fills** the space.

Every property you need to change is listed here — you don't have to
reopen guides 07 or 10. Works the same whether the gallery is bound to
`colUseCases` (v1) or `Projects` (v2); these are layout-only changes.

> **This guide supersedes** the `Wrap = false` row-label setting and the
> row font / `TemplateSize` from [`07` Steps 25 & 27](07-scrlist-guide.md)
> and the overflow note in [`10` Step 8](10-srclist-dataverse-guide.md).
> Where they differ, **use the values here.**

> **Why not auto-height rows?** A classic canvas gallery uses a single
> fixed `TemplateSize` for every row — there's no per-row auto-height. So
> "show everything" = wrap on + a row tall enough for the longest realistic
> value + a smaller font so values wrap less. One tall row height applies
> to all rows, so we pick the smallest height that fits the worst case.

---

## Part A — Rows show full text (wrap on, smaller font)

Enter the gallery template (chevron next to `galUseCases`). Update the
**seven row labels** — `lblUCID`, `lblName`, `lblSBU`, `lblOwner`,
`lblFY`, `lblValue`, `lblUpdated` — **and** the status text label
`lblStatusText`:

| Property | Value |
|----------|-------|
| Wrap | `true` |
| VerticalAlign | `VerticalAlign.Middle` |
| Size | `12` |
| Tooltip | *(optional)* keep `= ThisItem.<field>` as a safety net for a value so long it still overflows the row |

Then select the gallery `galUseCases` itself (exit the template):

| Property | Value |
|----------|-------|
| TemplateSize | `56` |

`56` comfortably fits **two** 12-pt lines (and tight three). Since every
row uses this height, taller = fewer rows visible, so `56` is the sweet
spot for typical Use Case Names.

> **Want it more compact / more rows on screen?** Drop the row `Size` to
> `11` and `TemplateSize` to `48`. **Want longer values fully on three
> lines?** Raise `TemplateSize` to `72`. Adjust the two together.

The header labels in `conGalleryHeader` stay at `Size 12` / `Wrap false`
(headings are short and shouldn't wrap).

---

## Part B — Shrink the title row

Select `conTitleRow` (inside `conPage`):

| Property | Old | New |
|----------|-----|-----|
| Y | `24` | `16` |
| Height | `80` | `56` |

The inner controls keep their positions — the title (`lblPageTitle`,
ends at y≈30) and subtitle (`lblPageSub`, ends at y≈52) still fit inside
56, and the Export / New buttons re-center via their `(Parent.Height-36)/2`
formula.

> Optional: to reclaim ~22px more, delete or hide `lblPageSub` (the
> subtitle). The count chip (`lblPageCount`) already conveys the essential
> info.

---

## Part C — Shrink the filter card

Select `conFilterCard`:

| Property | Old | New |
|----------|-----|-----|
| Y | `conTitleRow.Y + conTitleRow.Height + 16` | `conTitleRow.Y + conTitleRow.Height + 12` |
| Height | `92` | `76` |

The captions (`Y=10`) and inputs (`Y=30`, `Height=40`, bottom at 70)
inside the card don't move — they still fit within `76`. Width stays full
(`Parent.Width - 48`) so the six filter columns keep their room.

---

## Part D — Expand the gallery into the reclaimed space

Select `conGallery`:

| Property | Old | New |
|----------|-----|-----|
| Y | `conFilterCard.Y + conFilterCard.Height + 16` | `conFilterCard.Y + conFilterCard.Height + 12` |
| Height | `Parent.Height - Self.Y - 24` | `Parent.Height - Self.Y - 16` |

Because `conGallery.Y` is a formula chaining off the now-shorter title row
and filter card, the gallery **automatically starts higher** and grows.
The `- 16` (was `- 24`) trims the bottom margin for a bit more.

Inside `conGallery`, the gallery itself fills the card (footer removed):

| Control | Property | Value |
|---------|----------|-------|
| `galUseCases` | Height | `Parent.Height - 36` |

There is **no footer** ([`07` Part 6](07-scrlist-guide.md)); the
visible/total count lives in the title chip `lblPageCount`
(`"Showing " & Text(CountRows(galUseCases.AllItems)) & " of " & Text(CountRows(<source>))`,
where `<source>` is `colUseCases` for v1 or `Projects` for v2).

---

## Part E — All changes at a glance

| Control | Property | Old → New |
|---------|----------|-----------|
| `lblUCID`, `lblName`, `lblSBU`, `lblOwner`, `lblFY`, `lblValue`, `lblUpdated`, `lblStatusText` | Wrap | `false` → **`true`** |
| (same labels) | VerticalAlign | → **`Middle`** |
| (same labels) | Size | `13` → **`12`** |
| `galUseCases` | TemplateSize | `52` → **`56`** |
| `galUseCases` | Height | → **`Parent.Height - 36`** (footer removed) |
| `conTitleRow` | Y | `24` → **`16`** |
| `conTitleRow` | Height | `80` → **`56`** |
| `conFilterCard` | Y gap | `+ 16` → **`+ 12`** |
| `conFilterCard` | Height | `92` → **`76`** |
| `conGallery` | Y gap | `+ 16` → **`+ 12`** |
| `conGallery` | Height | `… - 24` → **`… - 16`** |

Net effect at 1366×768: the gallery grows from ~464px tall to ~528px
(plus the rows now show full, wrapped text). ~64px reclaimed from chrome,
~40px already reclaimed by removing the footer.

---

## Part F — Sanity check

Press **F5**:

- [ ] Long Use Case Names / SBUs / owners show **in full**, wrapped onto a
      second line where needed — nothing clipped, nothing spilling outside
      the row.
- [ ] Rows are a touch taller; the font is slightly smaller but readable.
- [ ] The title row and filter card are noticeably shorter, and the
      gallery starts higher and reaches the bottom of the page.
- [ ] Column cells still line up under their headers (FillPortions
      unchanged).
- [ ] The "Showing X of Y" chip in the title row still updates as you
      filter.
- [ ] Scrolling still works — it's a scrolling list, no pagination.

> If a particular value is so long it *still* overflows even a wrapped row,
> either raise `TemplateSize` (e.g. `72` for three lines) or widen that
> column's `FillPortions` (and trim another by the same amount, since
> they're proportional — see [`07` Steps 23 & 27](07-scrlist-guide.md)).
