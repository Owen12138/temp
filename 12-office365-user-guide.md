# BOA — Wire in the Office 365 User (Corporate) + Right-Aligning the Header

Two things in one guide:

- **Part 1–4:** replace the hardcoded sample user (`currentUser =
  { FullName: "Owen Huang", Initials: "OH" }` from
  [`01-app-setup.md` §3](01-app-setup.md)) with the **real signed-in user**
  from **Office 365 Users** — name, initials, and profile photo — the way
  it works in an Azure AD / Microsoft Entra corporate tenant.
- **Part 5:** the **X-value pattern for right-aligning** the user text +
  icons + avatar in the header (your second question), with the exact
  formulas to drop into `conHeader`.

Header controls referenced are from
[`02-build-guide.md` → `conHeader`](02-build-guide.md): `circUserAv`
(avatar circle), `lblUserInit` (initials), `lblWelcome` (the "Welcome, …"
label).

---

## Part 1 — Add the Office 365 Users connector (corporate notes)

In Studio: **Data → + Add data → search "Office 365 Users" → Add a
connection** (sign in with your corporate account if prompted).

What to know in a corporate tenant:

- **It's a Standard connector** — no premium license needed, and it's
  almost always allowed. If it's missing or greyed out, your admin's
  **DLP (Data Loss Prevention) policy** has blocked it; ask them to allow
  *Office 365 Users* for this environment.
- **It runs as the signed-in user.** `Office365Users.MyProfileV2()`
  returns *that* user's own Entra profile — exactly what "current user"
  should be. No app-only/service account needed.
- **Sharing:** when you share the app, the connection is shared with it;
  each user **consents once** on first launch and sees their own profile.
- **Guests/external users** may have sparse profiles (blank `surname`,
  no photo) — Parts 2–3 handle those gracefully.

---

## Part 2 — Replace `currentUser` in `App.OnStart`

Open `App` → **OnStart**. Replace the sample line

```powerfx
// OLD: Set(currentUser, { FullName: "Owen Huang", Initials: "OH" });
```

with a single profile fetch cached into a variable, then a friendly
`currentUser` shape (so every control that already reads
`currentUser.FullName` / `.Initials` keeps working):

```powerfx
// One network call — cache it
Set(gblMe, Office365Users.MyProfileV2());

Set(currentUser,
    {
        FullName: Coalesce(gblMe.displayName, gblMe.userPrincipalName, "User"),
        Email:    Coalesce(gblMe.mail, gblMe.userPrincipalName, ""),
        JobTitle: Coalesce(gblMe.jobTitle, ""),
        Initials:
            Upper(
                With({ parts: Split(Coalesce(gblMe.displayName, "U"), " ") },
                    Left(First(parts).Value, 1) &
                    If(CountRows(parts) > 1, Left(Last(parts).Value, 1), "")
                )
            )
    }
);
```

Notes:

- **`MyProfileV2()` field names are camelCase** (`displayName`,
  `givenName`, `surname`, `mail`, `jobTitle`, `userPrincipalName`, `id`).
  The older `MyProfile()` uses PascalCase (`DisplayName`) — pick one and be
  consistent. V2 is recommended.
- **`Coalesce(...)`** falls back when a field is blank (common for guests):
  display name → UPN → "User".
- **Initials** are derived from the display name's first + last word, so
  you don't depend on `givenName`/`surname` being populated.

> **Call it once.** `MyProfileV2()` is a network round-trip — fetch it in
> `OnStart` and read `gblMe` / `currentUser` everywhere else. Never put
> `Office365Users.MyProfileV2()` directly on a control property (it would
> re-call on every render).

---

## Part 3 — Profile photo (with a safe fallback)

Photos live in Exchange Online and **many corporate users haven't set
one**, so the call can error or return blank — always guard it.

Add to `App.OnStart` (right after the `Set(currentUser, …)` block):

```powerfx
Set(gblHasPhoto, false);
IfError(
    Set(gblUserPhoto, Office365Users.UserPhotoV2(gblMe.id)); Set(gblHasPhoto, true),
    Set(gblHasPhoto, false)
);
```

- `UserPhotoV2(gblMe.id)` returns the signed-in user's photo. On any
  error (no photo / not permitted), `gblHasPhoto` stays `false`.
- We'll show the photo when `gblHasPhoto` is true, and fall back to the
  initials circle otherwise (Part 4).

---

## Part 4 — Wire the header controls

In `conHeader` you already have `circUserAv` + `lblUserInit` + `lblWelcome`.
Update their text/data and add one image on top:

**`lblWelcome.Text`** — greet by first name from the live profile:

```powerfx
"Welcome, " & First(Split(currentUser.FullName, " ")).Value
```

**`lblUserInit.Text`** — already `currentUser.Initials`; no change needed
(now fed by Part 2).

**Add the photo on top of the circle.** Inside `conHeader`, Insert →
**Image**:

| Property | Value |
|----------|-------|
| Name | `imgUserAv` |
| Image | `gblUserPhoto` |
| Visible | `gblHasPhoto` |
| X / Y / Width / Height | same as `circUserAv` (see Part 5) |
| RadiusTopLeft / TopRight / BottomLeft / BottomRight | `Self.Width / 2` (round it) |

Set the fallback to hide when a photo exists:

- `lblUserInit.Visible` = `!gblHasPhoto`
- (`circUserAv` can stay visible as the backing circle, or also set
  `Visible = !gblHasPhoto`.)

Result: users with a photo see their headshot; everyone else sees the
maroon initials bubble.

---

## Part 5 — Right-aligning the user text + icons (X-value pattern)

**The core rule:** a control's `X` is the distance from its parent's left
edge. To pin something to the **right**, compute that distance from the
parent's width:

```powerfx
X = Parent.Width - Self.Width - RIGHT_MARGIN
```

That's "right edge of parent, minus my own width, minus a margin" — it
stays glued to the right even when the screen/header resizes (which the
old fixed offsets like `Parent.Width - 180` only approximate).

**For a row of items on the right, lay them out right-to-left:** place the
right-most control with the formula above, then each control to its left
anchors off the previous one:

```powerfx
X = <controlToMyRight>.X - Self.Width - GAP
```

### Apply it to the header cluster (avatar ▸ name ▸ icon)

Order on screen, right to left: **avatar** (far right) ◂ **"Welcome,
…"** ◂ optional **icon** (e.g. a bell). Set these inside `conHeader`
(its `Parent.Width` is the header's own width):

| Control | X | Y (vertically centered) |
|---------|---|--------------------------|
| `circUserAv` (avatar, right-most) | `Parent.Width - Self.Width - 16` | `(Parent.Height - Self.Height) / 2` |
| `imgUserAv` (photo overlay) | `circUserAv.X` | `circUserAv.Y` |
| `lblUserInit` (initials overlay) | `circUserAv.X` | `circUserAv.Y` |
| `lblWelcome` (text, left of avatar) | `circUserAv.X - Self.Width - 8` | `(Parent.Height - Self.Height) / 2` |
| `icnBell` *(optional, left of text)* | `lblWelcome.X - Self.Width - 12` | `(Parent.Height - Self.Height) / 2` |

Two finishing touches for the text label:

- Set **`lblWelcome.Align = Align.Right`** so the words hug the avatar (the
  label's *box* is wider than the text; right-aligning the text removes the
  gap).
- `(Parent.Height - Self.Height) / 2` vertically centers each item in the
  52-px bar regardless of its height — no more hand-tuned `Y: 16`.

> **Why this beats `Parent.Width - 180`:** the magic-number version assumes
> a fixed cluster width and breaks if you rename the greeting, add an icon,
> or change the avatar size. The chained `…- Self.Width - GAP` version
> re-flows automatically.

### Cleaner alternative — a right-docked container

If you'd rather not chain X's, drop a **horizontal container**
`conUser` in `conHeader`, put the icon + label + avatar inside (the
container lays them out left-to-right with `LayoutGap`), then right-dock
the container itself:

```powerfx
conUser.X = Parent.Width - conUser.Width - 16
conUser.Y = (Parent.Height - conUser.Height) / 2
```

Set `conUser.LayoutJustifyContent = LayoutJustifyContent.End` so the
contents sit at the container's right edge. This is the most robust option
and matches the `conUser` node already sketched in
[`02-build-guide.md`](02-build-guide.md).

---

## Sanity check

Press **F5** (and ideally test in the Player, signed in as yourself):

- [ ] The header greets you by your **real** first name.
- [ ] Your **initials** are correct (and your **photo** shows if you have
      one; initials bubble if you don't).
- [ ] The avatar sits flush to the right with a consistent margin; the
      greeting hugs its left; everything is vertically centered in the bar.
- [ ] Resize the Studio preview / collapse the rail — the user cluster
      stays pinned to the right (no clipping, no drift).
- [ ] In Studio you may see a one-time consent prompt for the Office 365
      Users connection; approve it.

---

## Troubleshooting

| Symptom | Cause | Fix |
|---|---|---|
| `Office365Users` not recognized in a formula | Connector not added | Data → Add data → Office 365 Users. |
| Connector missing / can't add | DLP policy blocks it in this environment | Ask your admin to allow the *Office 365 Users* standard connector. |
| Name/initials blank in Studio but fine in Player | Connection not yet consented in the editor | Re-open, approve the connection prompt; or test in the Player. |
| Photo never shows | User has no Exchange photo, or `UserPhotoV2` errored | Expected — `gblHasPhoto` stays false and the initials bubble shows. |
| Profile fields blank for some users | Guest/external account with a sparse profile | The `Coalesce(...)` fallbacks (Part 2) cover name/email; initials derive from display name. |
| Header user cluster drifts / clips on resize | Still using fixed offsets (`Parent.Width - 180`) | Switch to the `Parent.Width - Self.Width - margin` chain (Part 5). |
| App feels slow to open | `MyProfileV2()` / `UserPhotoV2()` called on a control instead of once in `OnStart` | Cache in `gblMe` / `gblUserPhoto` in `OnStart`; reference the variables. |
