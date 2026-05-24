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
   (`scrList`), follow the step-by-step **`07-scrlist-guide.md`** — it
   replaces section "Screen 2 · scrList" with numbered Insert steps,
   full property tables, and a sanity check after each part.
   *Estimated time: 4–6 hrs total (~1 hr per screen, Detail page is heaviest).*

3. **`03-formulas-reference.md`** — A flat lookup of every Power Fx formula
   organized by purpose (filters, value rollups, governance, submit flow,
   theming, Dataverse migration). Use this as you build — Studio's formula
   bar is friendlier when you have the formula already written.

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
- **The list uses infinite scroll, not pagination.** The gallery loads all
  filtered rows at once and lets Canvas's built-in virtualisation handle
  rendering. The footer label updates reactively; no page-index variable
  is needed.
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
