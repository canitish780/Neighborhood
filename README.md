# The Neighbourhood — setup guide

Your live links:

- **Customer app:** https://canitish780.github.io/Neighborhood/index.html
- **Accounts counter:** https://canitish780.github.io/Neighborhood/accounts.html
- **Outlet dashboard:** https://canitish780.github.io/Neighborhood/counter.html
- **Master control:** https://canitish780.github.io/Neighborhood/master.html
- **Google Sheet:** https://docs.google.com/spreadsheets/d/1QC7Na3J2rvg5gQQ9VokTph1XQgBpE2Jms0b_A6dHqzc/edit
- **Apps Script backend:** https://script.google.com/macros/s/AKfycbyAUESVBaflyYpjkoPHLx7tKeTUtfZbL83oEtB-2wxGDpjHFXEcvcIgsUvMMe2ynhWy-w/exec
- **Menu photos folder:** https://drive.google.com/drive/folders/1OFW-tobhKmCxPk8-NLXnKyS_pdqZ2tXD

All four pages already point at your Apps Script URL above — nothing to edit there.

## One-time setup (do this once)

1. Open your Sheet → **Extensions → Apps Script**.
2. Delete whatever's in `Code.gs`, paste in the `apps-script/Code.gs` from this
   folder.
3. **Deploy → Manage deployments → pencil icon → Version: New version → Deploy.**
   (Important: use *Manage deployments*, not *New deployment* — that keeps
   the same `/exec` URL you already gave me, so the live pages keep working.)
4. Open this link once in your browser:
   `https://script.google.com/macros/s/AKfycbyAUESVBaflyYpjkoPHLx7tKeTUtfZbL83oEtB-2wxGDpjHFXEcvcIgsUvMMe2ynhWy-w/exec?action=setupSeed`
   It creates all 5 tabs with the right headers, and — only the first time —
   seeds 6 demo outlets, ~24 demo menu items, and one demo customer
   (**login 9876543210 / 1234**) so you can test everything end to end.
   Safe to open again later; it won't duplicate data.

That's it — the customer app, accounts counter, outlet dashboard, and master
control are all already live at the links above and talking to your Sheet.

## Sheet tabs (created automatically by setupSeed)

**Customers** — `MobileNumber | Password | Name | Balance | CreatedAt`
Mobile number is the unique key — two customers can share a name, never a number.

**Outlets** — `OutletName`

**Menu** — `ItemID | Outlet | Category | ItemName | Price | ImageURL | Available`

**Orders** — `OrderID | Timestamp | MobileNumber | CustomerName | Outlet | ItemsJSON | Total | Status | Notes`
Status moves through `New → Preparing → Prepared → Delivered`. `Notes` holds
the customer's kitchen note for that outlet's portion of the order.

**Transactions** — `TransactionID | Timestamp | MobileNumber | Name | Type | Amount | BalanceAfter`
Every top-up, reduction, and order deduction logs a row here automatically.

## Who controls what

- **Master control** (`master.html`): adds outlets, adds menu items with a
  photo (saved straight into your Drive folder above), and is the *only*
  place prices get changed — tap the price on any item to edit it.
- **Outlet dashboard** (`counter.html`): two tabs. **Orders** is the existing
  live queue (customer name, mobile number, kitchen note, status buttons).
  **My menu** is new — lists that outlet's own items with an on/off switch,
  so an outlet can 86 something themselves without needing master control.
  Availability toggling now lives here, not in master control.
- **Accounts counter** (`accounts.html`): one search box matches name or
  mobile automatically. "+ New customer" (top right) creates an account —
  mobile number is enforced as the unique ID.

### Locking an outlet's dashboard to just its own orders + menu

Add `?outlet=NAME` to the link, e.g.:
`counter.html?outlet=Sizzling%20Woks`

That device then only ever sees that outlet's orders and menu, no dropdown
to browse others. Leave the `?outlet=` off for a manager device with a
dropdown across all outlets (pick one there to manage its menu too).

## How everything flows

1. Customer tops up at the accounts counter (or master creates their account
   with a starting balance).
2. Customer logs into the app, adds items from any outlets to one cart, adds
   an optional kitchen note per outlet on the checkout ticket, confirms —
   wallet is deducted, one order row is written per outlet.
3. Each outlet's dashboard shows only its own new orders (with the customer's
   name, mobile, and note), auto-refreshing every 10s. Preparing → Prepared →
   Delivered moves it into a collapsed "Delivered today" bucket.
4. The customer's bell icon (top right of the app) polls every 15s and shows
   live status per order.

## Known MVP shortcuts (worth fixing before real launch)

- **Passwords are stored in plain text in Sheets.** Fine to pilot with, but for
  real use switch to OTP-based login instead — happy to build that next.
- None of the four pages have their own login — the link itself is the
  access control. Keep `accounts.html`, `counter.html`, and `master.html`
  unlisted/bookmarked on trusted devices only, or say the word and I'll add
  a lightweight PIN per page.
- Apps Script free tier has usage quotas — fine for a single food court's
  daily volume, worth knowing if you scale to many locations.
- Uploaded photos rely on Drive's "anyone with the link can view" sharing on
  that specific folder — fine for menu photos, just don't reuse that folder
  for anything private.

## Next steps worth considering

- OTP login instead of password.
- Push notifications instead of polling (would need a paid notification
  service — the current 10–15s polling is the free, zero-infra version).
- Edit/delete for existing menu items and outlets (currently: add, plus
  price edit and availability toggle — no delete yet).
