# The Neighbourhood — setup guide

Now just **two** pages:

- `index.html` — customer app
- `management.html` — one login for all staff (KMP / Accounts / Outlet), routes to the right dashboard

Live links:
- Customer app: https://canitish780.github.io/Neighborhood/index.html
- Management: https://canitish780.github.io/Neighborhood/management.html
- Sheet: https://docs.google.com/spreadsheets/d/1QC7Na3J2rvg5gQQ9VokTph1XQgBpE2Jms0b_A6dHqzc/edit
- Apps Script: https://script.google.com/macros/s/AKfycbyAUESVBaflyYpjkoPHLx7tKeTUtfZbL83oEtB-2wxGDpjHFXEcvcIgsUvMMe2ynhWy-w/exec
- Menu photos folder: https://drive.google.com/drive/folders/1OFW-tobhKmCxPk8-NLXnKyS_pdqZ2tXD

`accounts.html`, `counter.html`, and `master.html` are gone — their functionality now lives inside `management.html`, split by role after login.

## One-time setup (do this again — the backend changed)

1. Sheet → **Extensions → Apps Script** → replace everything with `apps-script/Code.gs` from this folder.
2. **Deploy → Manage deployments → pencil icon → Version: New version → Deploy.** (Keeps the same `/exec` URL.)
3. Open this once in a browser:
   `https://script.google.com/macros/s/AKfycbyAUESVBaflyYpjkoPHLx7tKeTUtfZbL83oEtB-2wxGDpjHFXEcvcIgsUvMMe2ynhWy-w/exec?action=setupSeed`
   Creates any missing tabs (including the new **Users** tab) and, only if empty, seeds demo data — safe to re-run.

### Demo staff logins (created by setupSeed)

| Username | Password | Role | Notes |
|---|---|---|---|
| `admin` | `admin123` | Master control (KMP) | full access |
| `accounts1` | `acc123` | Accounts counter | |
| `outlet1` | `out123` | Outlet | tied to "Sizzling Woks" |

Demo customer login for the app: `9876543210` / `1234`.

Change these passwords (or delete the rows) once you've tried it out — replace them with real staff via the **Users** tab inside `management.html`.

## What changed

**One login, three destinations.** `management.html` asks for username, password,
and a "log in as" dropdown (Master control / Accounts / Outlet). It routes to the
matching dashboard automatically and remembers the device (like the customer app)
so staff don't have to log in every single time — there's a "Log out" button in
the header when they need to switch.

**KMP (master control)** now has four tabs:
- **Outlets** — add outlets.
- **Menu** — add items with a photo, tap any price to edit it.
- **Orders** — oversight view across every outlet, with a working outlet filter.
- **Users** — create Accounts and Outlet logins. Outlet logins are tied to one
  specific outlet at creation time, so that person only ever sees their own
  outlet's dashboard — no picking or mixing up outlets.

**Accounts counter** now shows the full customer list on load (not just search
results) — search narrows it, clearing the box shows everyone again.

**Outlet dashboard** is now auto-locked to whichever outlet the logged-in
user was assigned — no dropdown, no URL parameter needed. Two tabs: **Orders**
(unchanged — status buttons, mobile number, kitchen notes) and **My menu**
(the on/off toggle for that outlet's own items, unchanged from before).

**Customer app remembers the device.** After first login, the phone stays
signed in — no password needed next visit. There's a "Log out of this device"
link inside the bell/notifications panel for shared devices.

**Faster loading.** The menu is cached on-device: it renders instantly from
the last-seen copy while a fresh copy loads quietly in the background, so
reopening the app feels immediate instead of waiting on a network round trip
every time.

## Bug fix

The outlet dropdown showing only 2 outlets was because the old `counter.html`
built its dropdown from outlets *that already had orders*, not from the
actual Outlets list — new outlets with no orders yet were invisible. Now
every outlet dropdown (KMP's Orders tab, menu item outlet picker) reads
directly from the Outlets sheet via `getOutlets`, so it always shows all of
them immediately after you add one.

## Sheet tabs

**Customers** — `MobileNumber | Password | Name | Balance | CreatedAt`
**Outlets** — `OutletName`
**Menu** — `ItemID | Outlet | Category | ItemName | Price | ImageURL | Available`
**Orders** — `OrderID | Timestamp | MobileNumber | CustomerName | Outlet | ItemsJSON | Total | Status | Notes`
**Transactions** — `TransactionID | Timestamp | MobileNumber | Name | Type | Amount | BalanceAfter`
**Users** (new) — `Username | Password | Role | OutletName | Name` — Role is `KMP`, `Accounts`, or `Outlet`; `OutletName` only applies to `Outlet` role.

## Known MVP shortcuts (worth fixing before real launch)

- **Passwords are stored in plain text in Sheets** for both customers and
  staff. Fine to pilot with, not for production — OTP or hashed passwords
  would be the next step.
- Device-remember uses the browser's local storage — clearing browser data
  or switching browsers signs the device out. That's expected, not a bug.
- No delete for outlets, menu items, or staff users yet — only add, plus
  price edit and availability toggle.
- Apps Script free tier has usage quotas — fine for a single food court's
  daily volume.
- Uploaded photos rely on Drive's "anyone with the link can view" sharing on
  that specific folder.

## Next steps worth considering

- OTP login instead of password, for both customers and staff.
- Push notifications instead of polling.
- Edit/delete for outlets, menu items, and staff logins.
