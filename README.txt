# THE CHAP HOUSE – Restaurant Management & QR Ordering

A single-file web app (`index.html`) built on Firebase. It has two parts:

- **Staff dashboard** (admin, waiter, chef) – login, menu management, orders, kitchen display, KOT and bill printing, staff accounts, table QR codes.
- **Customer menu** – opened by scanning a table QR code (`/?table=5`). No login needed. Customers order, enter their email, and track the order live.

## Files

| File | Purpose |
|------|---------|
| `index.html` | The complete app (dashboard + customer menu) |
| `firestore.rules` | Firestore security rules |
| `README.md` | This guide |

## How it works

1. Customer scans the table QR and sees the menu.
2. Customer adds items, enters their email, and places the order (status **Pending**).
3. Waiter gets a "New order" alert and clicks **Accept & print KOT**. The order moves to **Accepted** and the KOT prints.
4. Chef sees only accepted orders in **Kitchen** and marks them **Preparing** then **Ready**.
5. Waiter marks the order **Served**. The bill is emailed to the customer automatically.

Order status flow: `Pending → Accepted → Preparing → Ready → Served` (or `Rejected`).

## Setup

### 1. Firebase project
1. Create a project at [console.firebase.google.com](https://console.firebase.google.com).
2. Add a **Web app** and copy the config into `firebaseConfig` inside `index.html`.
3. **Authentication → Sign-in method** → enable **Email/Password**.
4. **Firestore Database** → create a database (production mode).

### 2. Security rules
1. Open **Firestore Database → Rules**.
2. Paste the contents of `firestore.rules`.
3. Click **Publish**.

### 3. Create the first admin
The first admin must be created manually (the app cannot create it, by design).

1. **Authentication → Users → Add user** – enter the admin email and password. Copy the **User UID**.
2. **Firestore → Start collection** `users` → document ID = that **UID**, with fields:

| Field | Type | Value |
|-------|------|-------|
| `name` | string | Your name |
| `email` | string | Admin email |
| `role` | string | `admin` |
| `active` | boolean | `true` |

3. Log in to the dashboard with that email and password.

### 4. Deploy online (required for QR codes)
QR codes only work on customer phones if the site has a public URL.

```bash
npm install -g firebase-tools
firebase login
firebase init hosting     # public directory: the folder containing index.html
firebase deploy
```

Your site will be at `https://<project-id>.web.app`.

### 5. Generate table QR codes
1. Log in as admin → **Tables & QR**.
2. Click **Generate 50 tables**.
3. Enter your public URL (for example `https://the-chap-house.web.app`).
4. Click **Generate QR codes**, then **Print QR sheet**.

### 6. Email bills (EmailJS)
Bills are emailed using [EmailJS](https://www.emailjs.com) (free plan: 200 emails/month).

1. Create an EmailJS account and add an **Email Service** (for example Gmail).
2. Create an **Email Template**:
   - To Email: `{{to_email}}`
   - Subject: `{{subject}}`
   - Content (HTML): `{{{message_html}}}` (three braces)
3. In `index.html`, fill in the `EMAILJS` constant:
   ```js
   const EMAILJS={publicKey:"...",serviceId:"...",templateId:"..."};
   ```
4. In the EmailJS dashboard, restrict allowed domains to your website.
5. Your GSTIN is set in the `RESTAURANT` constant (change it there if needed).

Until EmailJS is configured, "Email bill" shows an error. Printing still works.

## Roles

| Feature | Admin | Waiter | Chef |
|---------|:-----:|:------:|:----:|
| Dashboard | ✅ | ✅ | ✅ |
| Menu (add / edit / delete) | ✅ | view | view |
| Staff accounts | ✅ | – | – |
| Tables & QR | ✅ | – | – |
| Reports | ✅ | – | – |
| Accept / reject orders, print KOT | ✅ | ✅ | – |
| Bill, print, email | ✅ | ✅ | – |
| Kitchen (Preparing / Ready) | ✅ | ✅ | ✅ |

## Staff accounts
Admin → **Staff Accounts → Create staff account**. Enter name, email, temporary password and role. The account is created in Firebase Authentication, and a profile is saved in Firestore. Share the credentials privately.

- **Disable** blocks the user from logging in to the dashboard.
- Staff can use **Forgot password?** on the login screen to receive a reset email.

## Menu and photos
Admin → **Food Menu → Add food item**. You can upload a photo (it is resized automatically and stored with the item) or paste an image URL. Use **Disable** to hide an item from customers without deleting it.

## Billing format

```
Total Qty
Sub Total
Discount            (optional)
S.C Charge @ 10%    (can be removed)
GST on Sub Total @ 5%
   CGST @ 2.5%
   SGST @ 2.5%
Round Off
GRAND TOTAL
```

- GST (5%) is calculated on the **Sub Total** (after discount, if any). It is **not** charged on the S.C amount.
- S.C (service charge) is 10% of the subtotal (after discount, if any). Use **Bill → Remove S.C charge** to remove it.
- Grand Total = Sub Total - Discount + S.C + CGST + SGST, rounded to the nearest rupee.
- Discount can be a percentage or a flat amount.
- The GSTIN is printed on every bill. Change it in the `RESTAURANT` constant in `index.html`.
- To change the rates, edit `SC_RATE` and `GST_RATE` in `index.html`.

## Reports (Admin only)
Open **Reports** in the sidebar.

1. Choose a date range (From / To) or use the quick buttons: Today, Yesterday, Last 7 days, This month, All time.
2. Choose which orders to include: **All accepted orders** or **Served orders only**. Pending and rejected orders are never counted.
3. Pick a report:
   - **Day-wise sales** – one row per day with orders, qty, sub total, discount, S.C, CGST, SGST, round off and net sales.
   - **Bill-wise sales** – every bill with its full breakup.
   - **Discount report** – only the bills where a discount was given (percent or flat amount) and the total discount.
   - **Item-wise sales** – quantity sold and sales amount for each menu item.
4. Use **Download CSV** (opens in Excel / Google Sheets) or **Print**.

The summary cards at the top (Orders, Qty, Sub Total, Discount, S.C, GST, Round Off, Net Sales) always match the selected date range.

## Troubleshooting

| Problem | Fix |
|---------|-----|
| QR opens the login page | The link must contain `?table=NUMBER`. Regenerate the QR codes using your public URL. |
| QR doesn't open on phones | You used `localhost`. Deploy the site and use the public URL. |
| Cannot create staff accounts | Enable Email/Password sign-in and publish `firestore.rules`. Make sure the admin has a `users/{uid}` document with `role: "admin"`. |
| "Missing or insufficient permissions" | Publish `firestore.rules`. Check the user's `role` and `active` fields. |
| Customer sees an empty menu | Add menu items and make sure at least one is **Available**. |
| KOT or bill doesn't print | Allow pop-ups for the site in your browser. |
| Bill email fails | Check the EmailJS keys, template variables, and the monthly limit. |
| Photo upload fails | Use a valid image file (JPG, PNG, WebP). |

## Notes
- The customer's order total is calculated on their phone. Waiters review each order before accepting, so tampering is easy to spot. For strict enforcement, validate totals in a Cloud Function.
- Customer emails are stored in the order document. Anyone with a full order ID can read that order. IDs are long and random, but treat them as private.
- Never share admin credentials. Use separate waiter and chef accounts.
