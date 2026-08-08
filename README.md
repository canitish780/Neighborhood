# The Neighbourhood

A digital wallet + ordering system for a multi-outlet food court. A customer
tops up once at the accounts counter, then orders across any of the outlets
from their phone, paying out of that one balance — no more paper coupons.

Built as two static web pages (installable as a PWA) backed by a Google
Sheet, with Google Apps Script as the API layer in between. No paid hosting,
no database to manage — everything lives in a Sheet you can open and read
directly.

---

## The two webpages

### 1. `index.html` — Customer app
- Log in with mobile number + password (set up by the accounts counter).
- The device stays logged in after the first login — no repeat password
  entry on the same phone. A "Log out of this device" link is in the
  notifications panel for shared devices.
- Browse the full menu, grouped by outlet. One search box filters by either
  outlet name or dish name at once.
- Tap any dish to see a larger photo and its full description before
  ordering; add to cart from there or straight from the list.
- Cart can span multiple outlets in one order. Before confirming, add an
  optional kitchen note per outlet (e.g. "less spicy" for one stall, nothing
  for another).
- Orders sync in the background from local storage, so a dropped connection
  never loses or double-charges an order (see "Reliability" below).
- Wallet balance pill (top right) opens a dropdown with recent top-ups,
  orders, and adjustments.
- Bell icon shows live status per order (New → Preparing → Prepared →
  Delivered), auto-refreshing; delivered orders collapse into a "Past
  orders" section.
- "Pure Veg" badge on login and the main header — all outlets are veg-only.

### 2. `management.html` — Staff app (one login, three roles)
A single login screen with username, password, and a "log in as" dropdown:

- **KMP (master control)** — full back-office:
  - *Outlets* — add outlets.
  - *Menu* — add items (name, category, price, description, photo);
    tap ✎ on any item for full edit (including delete) — price is
    master's call, not the outlet's.
  - *Orders* — oversight across every outlet, filterable by outlet, with
    delivered orders collapsed out of the way.
  - *Users* — create Accounts and Outlet logins. An Outlet login is tied to
    exactly one outlet at creation, so that person only ever sees their own
    outlet's dashboard.
  - *Reports* — orders/revenue today (fixed) plus a From/To date filter for
    revenue by outlet, top-selling items, and top-up/spend totals; total
    customers and total wallet liability (money owed in stored balances) are
    always shown as current snapshots.

- **Accounts counter** — one search box (name or mobile, whichever matches)
  over the full customer list; tap anyone to add or reduce their balance;
  "+ New customer" creates an account (mobile number is the enforced unique
  ID, so two customers can share a name but never a number).

- **Outlet** — locked automatically to whichever outlet the login is
  assigned to, no dropdown or URL parameter needed. Two tabs:
  - *Orders* — live queue with customer name, mobile, kitchen note, and
    status buttons. **Status can only move forward** (New → Preparing →
    Prepared → Delivered) — once a stage is marked, going back to an
    earlier one is disabled on the button itself and rejected by the
    server if attempted anyway. Delivered orders collapse into their own
    section.
  - *My menu* — on/off toggle for that outlet's own items (what shows on
    the live customer menu). Price stays master's to change.

Staff logins also stay signed in on the device after first login, with a
"Log out" button in the header.

---

## Architecture

```
GitHub Pages (index.html, management.html)
        │  fetch() POST/GET, JSON
        ▼
Google Apps Script Web App  ──────►  Google Sheet
   (Code.gs — the only backend)        (Customers, Outlets, Menu, Orders,
                                         Transactions, Users tabs)
Google Drive folder ◄── menu photo uploads (via Apps Script)
```

No server to host, no database to run — Apps Script's web app deployment
*is* the API, and the Sheet *is* the database. This keeps hosting free but
does mean Apps Script's execution limits and quotas apply (see Limitations).

---

## Reliability / correctness details worth knowing

- **Concurrency locking.** Every operation that reads-then-writes a balance
  or checks a uniqueness rule (placing an order, adding/reducing money,
  creating a customer/outlet/user, changing order status) runs inside a
  script lock on the backend — two devices hitting the same operation at
  once queue up instead of racing and silently corrupting a balance.
- **Idempotent orders.** Each order carries a client-generated ID. If a
  request is retried after a dropped connection, the backend recognizes the
  duplicate and never charges the wallet twice.
- **Offline-safe ordering.** Placing an order writes it to the phone's local
  storage first and shows the confirmation screen immediately; a background
  sync (every 15s, and on next app open) pushes it to the Sheet, retrying
  until it succeeds.
- **Forward-only order status**, enforced both in the UI (buttons for
  earlier stages are disabled) and on the server (a downgrade request is
  rejected outright) — belt and suspenders.
- **Wallet balance loading state.** The balance pill visibly pulses while
  unconfirmed instead of silently showing a number that might be stale.

---

## One-time setup

1. Open your Sheet → **Extensions → Apps Script** → replace everything with
   `apps-script/Code.gs` from this folder.
2. **Deploy → Manage deployments → pencil icon → Version: New version →
   Deploy.** (Keeps the same `/exec` URL — never use "New deployment" for an
   update, that creates a different URL your pages aren't pointed at.)
3. Open once in a browser: `<your exec URL>?action=setupSeed`
   Creates any missing tabs/columns and — only if empty — seeds demo
   outlets, menu items, a demo customer, and demo staff logins. Safe to
   run again; never touches data that's already there.

Live links for this deployment:
- Customer app: https://canitish780.github.io/Neighborhood/index.html
- Management: https://canitish780.github.io/Neighborhood/management.html
- Sheet: https://docs.google.com/spreadsheets/d/1QC7Na3J2rvg5gQQ9VokTph1XQgBpE2Jms0b_A6dHqzc/edit
- Apps Script: https://script.google.com/macros/s/AKfycbyAUESVBaflyYpjkoPHLx7tKeTUtfZbL83oEtB-2wxGDpjHFXEcvcIgsUvMMe2ynhWy-w/exec
- Menu photos folder: https://drive.google.com/drive/folders/1OFW-tobhKmCxPk8-NLXnKyS_pdqZ2tXD

## Sheet tabs

| Tab | Columns |
|---|---|
| Customers | MobileNumber, Password, Name, Balance, CreatedAt |
| Outlets | OutletName |
| Menu | ItemID, Outlet, Category, ItemName, Price, ImageURL, Available, Description |
| Orders | OrderID, Timestamp, MobileNumber, CustomerName, Outlet, ItemsJSON, Total, Status, Notes, ClientOrderId |
| Transactions | TransactionID, Timestamp, MobileNumber, Name, Type, Amount, BalanceAfter |
| Users | Username, Password, Role, OutletName, Name |

`Status` moves through `New → Preparing → Prepared → Delivered`. `Role` is
one of `KMP`, `Accounts`, `Outlet`.

## Demo logins (created by setupSeed)

| Where | Username/Mobile | Password | Role |
|---|---|---|---|
| Management | `admin` | `admin123` | KMP |
| Management | `accounts1` | `acc123` | Accounts |
| Management | `outlet1` | `out123` | Outlet — Sizzling Woks |
| Customer app | `9876543210` | `1234` | — |

---

## Known limitations

- **Passwords are plain text** in the Sheet, for both customers and staff.
  Fine for a pilot; not acceptable for a real production launch.
- **No real authentication layer** — the Apps Script web app is deployed
  "Anyone can access," and anyone with the URL could call its API directly
  with the right parameters. There's no per-request auth token; the login
  screens are a UX gate, not a security boundary.
- **Device-remember uses browser local storage** — clearing site data or
  switching browsers signs the device out. Expected, not a bug.
- **No delete for outlets or staff users** — menu items now support full
  edit/delete; outlets and staff logins currently only support add.
- **Apps Script quotas** apply (execution time, requests/day, Drive
  storage) — comfortable for a single food court's daily volume, worth
  checking Google's current limits before scaling to many locations.
- **No image compression** — photos upload at whatever size the phone
  camera produces; large photos will be slow to upload and to load in the
  customer app.
- **Reports are computed on read**, scanning the full Orders/Transactions
  sheets each time — fine at pilot scale, would need optimizing (e.g.
  cached summaries) at high order volumes.
- **Single currency, single tax model** — no GST/tax breakdown, no
  multi-currency support.
- **No printer integration** — outlet staff read orders off the dashboard
  screen; nothing prints automatically to a kitchen printer.

## Suggested future upgrades

- **OTP-based login** instead of passwords, for customers and staff both.
- **Push notifications** instead of polling, once order status changes —
  would need a paid push service (Apps Script can't push on its own).
- **Real auth tokens** on the Apps Script API (e.g. a signed session token
  per login) instead of trusting whatever the client sends.
- **Edit/delete for outlets and staff users**, matching what menu items
  already have.
- **Downloadable/exportable reports** (CSV or PDF) instead of on-screen only.
- **Order printing** — auto-print new orders at each outlet via a connected
  receipt printer.
- **Multi-location support** — right now this Sheet/script pair serves one
  food court; running several would mean either separate deployments or
  adding a "Location" dimension throughout.
- **Refunds / order cancellation** — there's currently no way to reverse a
  placed order and credit the wallet back.
- **Analytics on customer behavior** — repeat-visit rate, average basket
  size, etc., beyond the current sales-focused reports.

## Known bugs worth watching for

- None currently open as of this write-up — the last few rounds fixed the
  outlet-dropdown-showing-only-2-outlets bug, the balance race condition
  across devices, and photos not rendering for customers. If something new
  turns up, the Sheet is the source of truth — check there first to see
  whether it's a data issue or a display issue.
