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

### Why this needs real thought

In a real corporate directory the name fields are messy:

- **`displayName` is not always "First Last".** Banks and large enterprises
  very often set it to **"Surname, Given"** (e.g. `"Huang, Owen"`), or add
  suffixes (`"Owen Huang (Contractor)"`, `"Owen Huang | Capital Markets"`).
  Parsing first-word/last-word off that gives the *wrong* first name and
  *reversed* initials.
- **`givenName` / `surname` are the authoritative, order-independent
  fields** — when populated, always prefer them. But for service accounts,
  some guests, and older AD records they can be **blank**.
- **Single-token names** ("Cher", "Madonna") must not produce doubled
  initials ("CC").

So the logic is: **prefer `givenName` + `surname`; fall back to parsing
`displayName`, detecting the "Surname, Given" comma format; and never
double a single token.** Compute it once in `OnStart` into a rich
`currentUser` record.

### The OnStart block

Open `App` → **OnStart** and replace the sample line
`Set(currentUser, { FullName: "Owen Huang", Initials: "OH" });` with:

```powerfx
// One network call — cache the raw profile
Set(gblMe, Office365Users.MyProfileV2());

With(
    {
        _gn: Trim(Coalesce(gblMe.givenName, "")),                          // given name (may be blank)
        _sn: Trim(Coalesce(gblMe.surname,  "")),                           // surname    (may be blank)
        _dn: Trim(Coalesce(gblMe.displayName, gblMe.userPrincipalName, "User")),
        _comma: !IsBlank(Find(",", Coalesce(gblMe.displayName, "")))       // "Surname, Given" format?
    },
    With(
        {
            // First name: prefer givenName; else parse displayName
            //   - comma format  -> text AFTER the comma
            //   - normal format -> first word
            _first: If(!IsBlank(_gn),
                       _gn,
                       If(_comma, Trim(Last (Split(_dn, ",")).Value),
                                  Trim(First(Split(_dn, " ")).Value))),
            // Last name: prefer surname; else parse displayName
            //   - comma format  -> text BEFORE the comma
            //   - normal format -> last word
            _last:  If(!IsBlank(_sn),
                       _sn,
                       If(_comma, Trim(First(Split(_dn, ",")).Value),
                                  Trim(Last (Split(_dn, " ")).Value)))
        },
        Set(currentUser,
            {
                FirstName: _first,
                LastName:  _last,
                // Natural-order full name even if the directory stored "Surname, Given"
                FullName:  If(_comma, Trim(_first & " " & _last), _dn),
                Email:     Coalesce(gblMe.mail, gblMe.userPrincipalName, ""),
                JobTitle:  Coalesce(gblMe.jobTitle, ""),
                // First + last initial; single-token names get just one
                Initials:  Upper(Left(_first, 1) & If(_last = _first, "", Left(_last, 1)))
            }
        )
    )
);
```

### How it resolves, case by case

| `givenName` / `surname` | `displayName` | → FirstName / LastName | → Initials |
|---|---|---|---|
| `Owen` / `Huang` | anything | Owen / Huang | **OH** |
| *(blank)* / *(blank)* | `Owen Huang` | Owen / Huang | **OH** |
| *(blank)* / *(blank)* | `Huang, Owen` | Owen / Huang | **OH** |
| *(blank)* / *(blank)* | `Owen Michael Huang` | Owen / Huang | **OH** |
| *(blank)* / *(blank)* | `Cher` | Cher / Cher | **C** |
| *(blank)* / *(blank)* | *(blank)* → UPN `owen.h@cibc.com` | owen.h@cibc.com / … | first letter(s) |

Notes:

- **`MyProfileV2()` fields are camelCase** (`displayName`, `givenName`,
  `surname`, `mail`, `jobTitle`, `userPrincipalName`, `id`). The older
  `MyProfile()` is PascalCase (`DisplayName`) — don't mix them. Prefer V2.
- **`Coalesce`** chains fallbacks (blank → next), so guests with sparse
  profiles still get a name from the UPN.
- The remaining ugly edge is a **suffix** in `displayName`
  (`"Owen Huang (Contractor)"`) when `givenName`/`surname` are *also*
  blank — last word becomes `(Contractor)`. It's rare (those fields are
  usually populated); if your tenant has it, strip the suffix with
  `Substitute`/`Left(... Find("(", ...))` before splitting.

> **Call it once.** `MyProfileV2()` is a network round-trip — fetch it in
> `OnStart`, then read `gblMe` / `currentUser` everywhere. Never put
> `Office365Users.MyProfileV2()` on a control property (it would re-call on
> every render). If you'd rather not slow app launch, move this block to
> `srcHome.OnVisible` behind an `If(IsBlank(currentUser.FullName), …)`
> guard.

---

## Part 3 — Profile photo (with a safe fallback)

### Why this needs thought too

Photos live in Exchange Online and behave inconsistently across tenants:

- **Most corporate users have no photo set** — so this is the common path,
  not the edge case. The fallback (initials bubble) must be solid.
- **Two different failure modes:** depending on connector version and
  tenant, `UserPhotoV2` either **throws an error** *or* **returns blank /
  an empty image** when there's no photo. A guard that only catches the
  error will let a blank image through and you'll get an empty avatar
  covering your initials.

So we guard for **both**: wrap in `IfError`, *and* set the "has photo" flag
from whether the returned image is actually non-blank.

### The OnStart block

Add right after the `Set(currentUser, …)` block:

```powerfx
IfError(
    With({ _photo: Office365Users.UserPhotoV2(gblMe.id) },
        Set(gblUserPhoto, _photo);
        Set(gblHasPhoto, !IsBlank(_photo))      // false if the call returned an empty image
    ),
    // call errored (no photo / not permitted)
    Set(gblUserPhoto, Blank());
    Set(gblHasPhoto, false)
);
```

- `gblHasPhoto` is **true only when the call succeeded *and* gave a real
  image** — covers both the error path and the blank-return path.
- The avatar in Part 4 shows `gblUserPhoto` when `gblHasPhoto`, else the
  initials bubble.

### Make the photo a circle that fills cleanly

On the `imgUserAv` Image control (Part 4), besides the radius:

- **`ImagePosition = ImagePosition.Fill`** so a rectangular headshot crops
  to fill the round avatar instead of letterboxing with gaps.
- Round it with `RadiusTopLeft/TopRight/BottomLeft/BottomRight = Self.Width / 2`.

> **More precise existence check (optional).** Instead of relying on the
> downloaded image being non-blank, you can ask for **metadata** first:
> `Office365Users.UserPhotoMetadata(gblMe.id)` returns size info and errors
> when no photo exists. Wrapping *that* in `IfError` lets you set
> `gblHasPhoto` without downloading the image, then fetch `UserPhotoV2`
> only when it exists. Overkill for one current-user avatar; useful if you
> ever show photos for many people in a gallery.

> **Performance / galleries.** `UserPhotoV2` is a per-user network call.
> One call for the signed-in user (cached in `gblUserPhoto`) is fine. Never
> call it row-by-row in a gallery — bind the gallery's avatar to a
> pre-built collection or accept initials-only there.

---

## Part 4 — Wire the header controls

In `conHeader` you already have `circUserAv` + `lblUserInit` + `lblWelcome`.
Update their text/data and add one image on top:

**`lblWelcome.Text`** — greet by first name. Use the `FirstName` we already
computed in Part 2 (correct even for "Surname, Given" directories — don't
re-parse `FullName` here):

```powerfx
"Welcome, " & currentUser.FirstName
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
| ImagePosition | `ImagePosition.Fill` (crop-to-fill, no letterboxing) |
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
