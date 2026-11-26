# THE CHAP HOUSE — QR Ordering Fix

## What was wrong

1. QR menu and QR order submission run in the browser without a customer login.
   Firestore's default/locked rules deny unauthenticated reads and writes. If public
   menu reads and the narrowly-scoped customer order create rule are not published,
   customers see an empty/error menu or "Missing or insufficient permissions".
2. The kitchen screen intentionally displays only `Accepted` and `Preparing` orders.
   A QR order is first saved as `Pending`; a captain/waiter must click **Accept & print KOT**.
   It will then appear in Kitchen. It is not sent directly to Kitchen by design.
3. QR generation previously built `base + "/?table=N"`, which breaks when the
   entered public URL already contains a page path or query string. It now uses
   the URL API and sets/replaces the `table` query parameter safely.

## Files

- `index.html` — updated website.
- `firestore.rules` — Firestore rules required for public menu read and customer order create.

## Required Firebase setup (do this before testing)

1. Open Firebase Console → **the-chap-house** → Firestore Database → **Rules**.
2. Back up your current rules, then review and publish the included `firestore.rules`.
   These rules allow:
   - public read of `menu`;
   - QR customers to create only `Pending` orders with basic field checks;
   - active signed-in staff (a user document exists and `active != false`) to read/manage
     orders and maintain menu/tables.
3. Confirm Firebase Authentication staff users have matching `/users/{uid}` documents.
4. Deploy `index.html` to your public HTTPS hosting (Firebase Hosting recommended).
5. In Dashboard → Tables & QR, enter the public URL, click **Generate QR codes**,
   print the sheet, and scan a QR using a phone on mobile data (not localhost).

## Correct order flow

Customer scans QR → menu loads → adds items → enters email → Place order →
Firestore `/orders` document is created with `status: Pending` → Captain/Waiter
opens Order Management and clicks **Accept & print KOT** → status becomes `Accepted`
and the order appears on Kitchen Display → Chef starts preparation → marks Ready.

## Important security note

The included rules are a practical starter for this existing client-side design, not
a complete anti-abuse system. Public unauthenticated order creation can be abused
by automated clients. For production, add Firebase App Check and ideally move order
creation to a trusted Cloud Function that validates table numbers, menu item IDs,
prices, quantities and totals on the server. Review these rules against any custom
rules you already use before publishing.

## Troubleshooting

- `Missing or insufficient permissions`: publish the rules and verify the Firebase
  project is `the-chap-house`.
- Menu empty: verify menu documents exist and `available` is not `false`.
- Order appears in Orders as Pending but not Kitchen: expected; waiter/captain must
  accept it first.
- QR opens wrong site: regenerate QR using the exact public HTTPS URL where this
  `index.html` is deployed. Do not use `localhost`, `127.0.0.1`, or a private LAN IP.
- QR library error: allow the QRCode CDN (`cdnjs.cloudflare.com`) and reload dashboard.

The app uses the Firebase config already present in the supplied HTML. Do not publish
private service-account keys or admin credentials in the browser.
