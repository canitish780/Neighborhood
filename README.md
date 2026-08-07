# The Neighbourhood — setup guide

Two pages:

- `index.html` — customer app
- `management.html` — one login for all staff (KMP / Accounts / Outlet)

Live links:
- Customer app: https://canitish780.github.io/Neighborhood/index.html
- Management: https://canitish780.github.io/Neighborhood/management.html
- Sheet: https://docs.google.com/spreadsheets/d/1QC7Na3J2rvg5gQQ9VokTph1XQgBpE2Jms0b_A6dHqzc/edit
- Apps Script: https://script.google.com/macros/s/AKfycbyAUESVBaflyYpjkoPHLx7tKeTUtfZbL83oEtB-2wxGDpjHFXEcvcIgsUvMMe2ynhWy-w/exec
- Menu photos folder: https://drive.google.com/drive/folders/1OFW-tobhKmCxPk8-NLXnKyS_pdqZ2tXD

## Re-do the one-time setup — the backend changed again

1. Paste the new `apps-script/Code.gs` into your Apps Script editor, replacing everything.
2. **Deploy → Manage deployments → pencil icon → Version: New version → Deploy** (same `/exec` URL).
3. Open once in a browser: `.../exec?action=setupSeed`
   This adds a `ClientOrderId` column to your existing Orders tab (needed for the fix below) without touching any data you already have, plus creates anything still missing. Safe to run again.

## What changed this round

### 1. Fixed: orders clashing across devices

This was a real correctness bug, not cosmetic — two devices placing an order
(or two staff adjusting a balance) at the exact same moment could read the
same starting balance and overwrite each other's update, silently losing
money. Fixed two ways:

- **Server-side locking.** Every operation that reads-then-writes a balance
  or checks uniqueness (placing an order, adding/reducing money, creating a
  customer/outlet/user, changing order status) now runs inside a script
  lock — concurrent requests queue up and execute one at a time instead of
  racing. This is the actual fix; everything else below is about making it
  *feel* fast despite that queuing.
- **Client-side offline queue.** When you place an order, it's saved to the
  phone's local storage first and the confirmation screen shows immediately
  — it doesn't wait on the network. A background sync then sends it to the
  Sheet; if the connection is flaky it keeps retrying (every 15s and on next
  app open) until it succeeds. Each order carries a unique ID generated on
  the phone, so if a retry happens after a request that actually *did* go
  through but the response got lost, the server recognizes the duplicate and
  does **not** charge the wallet twice.

### 2. Wallet balance: loading state instead of guessing

The wallet pill now visibly pulses/dims while a balance is unconfirmed
(right after opening the app, and right after placing an order) instead of
silently showing a number that might be stale. Once the server confirms the
real balance, it settles and stops pulsing. The confirmation screen after
ordering shows "Syncing with the kitchen…" until the background sync
actually completes, then flips to "✓ Confirmed."

### 3. Faster-feeling animations (optimistic UI)

Outlet toggles (My menu on/off) and order status buttons (Preparing →
Prepared → Delivered) now flip instantly on tap — the screen updates from
memory immediately, then syncs to the Sheet in the background. If that
background sync fails, it quietly rolls back and re-fetches the real state.
You should no longer feel a pause when tapping these.

### 4. Wallet transaction history

Tapping the wallet balance pill in the customer app opens a short history:
recent top-ups, orders, and adjustments with amounts and running balance.

### 5. Customer app: past orders collapse like the outlet dashboard

In the bell/notifications panel, delivered orders now move into a
collapsed "Past orders (n)" section underneath the active ones, instead of
staying mixed in with what's still being prepared.

### 6. Master control → Reports tab

New tab on the KMP dashboard with:
- Orders & revenue today, and all-time
- Total customers and **total wallet liability** (sum of everyone's prepaid
  balance — useful for an owner to know how much is owed in stored value)
- Total topped up / total spent
- Revenue by outlet, and top-selling items

## Sheet tabs

**Customers** — `MobileNumber | Password | Name | Balance | CreatedAt`
**Outlets** — `OutletName`
**Menu** — `ItemID | Outlet | Category | ItemName | Price | ImageURL | Available`
**Orders** — `OrderID | Timestamp | MobileNumber | CustomerName | Outlet | ItemsJSON | Total | Status | Notes | ClientOrderId` (last column is new — used for the duplicate-order safety check)
**Transactions** — `TransactionID | Timestamp | MobileNumber | Name | Type | Amount | BalanceAfter`
**Users** — `Username | Password | Role | OutletName | Name`

## Demo logins (from setupSeed)

| Where | Username/Mobile | Password | Role |
|---|---|---|---|
| Management | `admin` | `admin123` | KMP |
| Management | `accounts1` | `acc123` | Accounts |
| Management | `outlet1` | `out123` | Outlet — Sizzling Woks |
| Customer app | `9876543210` | `1234` | — |

## Known MVP shortcuts (worth fixing before real launch)

- Passwords are stored in plain text in Sheets, for both customers and staff.
- Device-remember uses local browser storage — clearing browser data or
  switching browsers signs the device out.
- No delete for outlets, menu items, or staff users yet — only add, plus
  price edit and availability toggle.
- Apps Script's script lock serializes writes safely, but it does mean a
  burst of simultaneous orders processes slightly slower per-order than
  before (milliseconds, not seconds) — a deliberate trade for correctness.
- Apps Script free tier has usage quotas — fine for a single food court's
  daily volume.

## Next steps worth considering

- OTP login instead of password.
- Push notifications instead of polling.
- Edit/delete for outlets, menu items, and staff logins.
- Date-range filters on the Reports tab (currently today + all-time only).
