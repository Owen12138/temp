# BOA Canvas App — Build Bundle

This folder contains everything you need to build the Canvas Power App for
the Business Opportunity Assessment, sized to match your HTML prototype.

## Why a build guide and not a packed .msapp?

I evaluated building a packed `.msapp` for direct import. The honest answer:
`pac canvas pack` was designed to roundtrip apps that already exist in
Studio, not to author them from scratch. A hand-authored YAML bundle for a
4-screen app of this complexity has a high probability of producing a
corrupted package that won't open. So instead, I produced a comprehensive
build guide with every formula, control, and property pre-decided. You
build in Studio in a few hours with no design decisions left to make.

## How to use these files

In Power Apps Studio, create a new blank Tablet-landscape Canvas app, then:

1. **`01-app-setup.md`** — Configure app settings, paste `App.OnStart`,
   paste `App.Formulas`, create the `cmpStatusPill` component.
   *Estimated time: 20–30 min.*

2. **Build the collapsible left rail first.** Three guides exist for this,
   pick one:
   - `04-left-rail-guide.md` — component-based (`cmpLeftRail`) with a
     gallery inside. Cleanest if everything works first time, but the
     component + gallery combo has the most failure modes.
   - `05-left-rail-inline-guide.md` — same gallery, but inline on each
     screen (no component). Easier to debug.
   - **`06-left-rail-buttons-guide.md` — recommended.** No component, no
     gallery; each nav row is a Button + Icon. Fixes icon-not-rendering,
     no-hover-effect, no-active-darken, and no-hand-cursor issues that
     show up with the gallery approach.

   *Estimated time: 30–45 min.*

3. **`02-build-guide.md`** — Build each of the 4 screens. Each screen has
   a control tree, every control's key properties, and per-control formulas.
   Work top-to-bottom; don't skip ahead. For the left rail section, follow
   `04-left-rail-guide.md` instead. For the Use Case List screen
   (`srcList`), follow the step-by-step **`07-scrlist-guide.md`** — it
   replaces section "Screen 2 · srcList" with numbered Insert steps,
   full property tables, and a sanity check after each part. For the
   Detail screen (`srcDetail`), follow **`08-srcdetail-guide.md`** —
   same step-by-step format, covers the two-column layout, section nav,
   status stepper, value table with edit modal, and checklist galleries.
   *Estimated time: 4–6 hrs total (~1 hr per screen, Detail page is heaviest).*

3. **`03-formulas-reference.md`** — A flat lookup of every Power Fx formula
   organized by purpose (filters, value rollups, governance, submit flow,
   theming, Dataverse migration). Use this as you build — Studio's formula
   bar is friendlier when you have the formula already written.

## Dataverse migration & layout (v2)

Once the standalone app works, these cover moving `srcList` to live data
and tuning its layout:

- **`09-dataverse-schema.md`** — the authoritative 3-table Dataverse
  schema (Business Hierarchy, Projects, Value): fields, types,
  relationships, choice option sets.
- **`10-srclist-dataverse-guide.md`** — convert `srcList` from
  `colUseCases` to the live Dataverse tables (inline filtering, choice
  columns, lookup-resolved SBU, Realized Value rollup, delegation).
- **`11-srclist-layout-tuning.md`** — optional layout pass: full-text
  (wrapped) rows, smaller font, shorter title row / filter card, and a
  bigger gallery. Supersedes the clip-based row settings in 07/10.
- **`12-office365-user-guide.md`** — wire the header to the real signed-in
  Office 365 user (name, initials, photo) in a corporate tenant, plus the
  X-value pattern for right-aligning the header user cluster.
- **`13-status-stepper-dataverse-fix.md`** — fix the `srcDetail` status
  stepper after moving to Dataverse (the `Project Status` option set +
  label mismatch); adds a `DVStatus` bridge field to `colStatus`.
- **`14-srcdetail-dataverse-guide.md`** — migrate the whole `srcDetail`
  screen to Dataverse with Edit forms: Info form rebind, the
  section-form recipe for Contacts/Funds/Gov/Tech/Updates (now Projects
  fields), and the Value child table (gallery + form).
- **`15-srclist-columns-update.md`** — change the srcList gallery to the
  10-column set + View button (adds Est. Completion, AI Enablement Owner,
  Executive Sponsor; drops FY; Realized Value (YTD) left as a placeholder).
- **`16-srcdetail-dataverse-build.md`** — build srcDetail forward from the
  Use Case Info container on Dataverse, with every command tagged by where
  it lives (OnStart / Formulas / OnVisible / Inline).

## Order to ship

- **v1 (now)**: Build against `colUseCases` collections (sample data lives
  in `App.OnStart`). The app is fully functional standalone — you can demo
  to stakeholders without touching Dataverse.
- **v2**: Create the Dataverse solution (separate work), then follow the
  migration cheat sheet in section N of `03-formulas-reference.md`.
- **v3**: Build the `BOA_SubmitAssessment` Power Automate flow and wire
  it to `btnSubmit` (section K).
- **v4**: Embed Power BI tile on a new "Tracker" screen.

## What's NOT here, and what to expect

- **The collapsible side rail will snap, not animate.** Canvas has no CSS
  transitions. This is a permanent limitation, not a v1 cut.
- **The list shows every matching row at once — no pagination, no
  lazy-load.** The gallery sets `Items = filteredUseCases` and Canvas
  virtualises rendering. Scrolling moves through the loaded list; it
  doesn't fetch more. The footer reports filter state (e.g. `"8 hidden
  by filters"`), not load state. Switch to `FirstN(...) / Skip(...)`
  if you later need true paging.
- **Pixel-perfect match to the HTML is ~90%.** Power Apps controls have
  their own internal padding and focus rings that won't perfectly match
  the prototype's CSS. Users won't notice; designers will.
- **No PCF (code components).** Everything uses native Power Apps controls.
  If you need drag-and-drop, animated transitions, or custom scrollbars,
  those are separate PCF work.

## Questions / things I assumed

- **Tablet landscape** at 1366×768 (matches enterprise web app pattern).
  If you need to support phones, build a separate phone app or use
  responsive containers — flagged in the guide.
- **Sample data uses Owen Huang as the current user.** Change
  `currentUser` in App.OnStart, or replace with `Office365Users.MyProfile()`
  when ready.
- **Realistic dollar formatting** (`$1.8M`) is calculated from raw integers
  stored as cents — i.e., `RealizedValue: 1800000`. Adjust if your
  financial data is in different units.

Open `01-app-setup.md` first.
