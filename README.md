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
   Adds a `Description` column to your existing Menu tab without touching any
   data you already have. Safe to run again.

## What changed this round

### Menu item descriptions + a detail popup for customers
Adding an item in master control now includes a description field. On the
customer app, tapping anywhere on an item card (not the Add button itself)
opens a popup with a larger photo, category, price, and the full
description — a proper look before ordering.

### Search toggle for customers
A toggle above the menu — **Search outlets** / **Search dishes** — switches
what the search box filters: outlet names, or dish names across every
outlet. Filters live as you type, no page reload.

### "Pure Veg" badge
A small green-dot pill reading "Pure Veg" now sits top-right on both the
login screen and the main menu header.

### Master control: full item editing
The ✎ button on any item in the Menu tab now opens a complete edit panel —
name, category, price, description, and a "Change photo" option — plus a
**Delete this item** button (with a confirmation prompt) at the bottom.
Availability toggling still stays with each outlet, unchanged.

### Reports: date range filter
The Reports tab now has From/To date fields. Apply them to filter revenue,
order counts, outlet breakdown, and top items to that window; "Clear
(all-time)" resets to everything. "Orders today" and "Revenue today" always
reflect today regardless of the filter, as a fixed reference point.

### Fixed: uploaded photos not showing for customers
The old photo links (`drive.google.com/uc?export=view&id=...`) don't render
reliably as `<img>` sources for everyone. New uploads now use Drive's
thumbnail endpoint instead, which does. Any photos you already uploaded are
also automatically converted to the working format wherever the app reads
them — you don't need to re-upload anything already in the Sheet.

### Wallet history: now a dropdown
Tapping the wallet balance on the customer app opens a small dropdown right
below it — same style as the notifications bell — instead of a full-screen
sheet.

## Sheet tabs

**Customers** — `MobileNumber | Password | Name | Balance | CreatedAt`
**Outlets** — `OutletName`
**Menu** — `ItemID | Outlet | Category | ItemName | Price | ImageURL | Available | Description`
**Orders** — `OrderID | Timestamp | MobileNumber | CustomerName | Outlet | ItemsJSON | Total | Status | Notes | ClientOrderId`
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
- No delete for outlets or staff users yet — menu items now have full
  edit/delete; outlets and users still only support add.
- Apps Script free tier has usage quotas — fine for a single food court's
  daily volume.

## Next steps worth considering

- OTP login instead of password.
- Push notifications instead of polling.
- Edit/delete for outlets and staff logins.
- Export reports to a downloadable file.
