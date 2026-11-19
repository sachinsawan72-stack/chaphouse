# THE CHAP HOUSE - Firebase Setup Guide

Restaurant:
THE CHAP HOUSE
Add Level 3, SLN Terminus, Jayabheri Enclave, Gachibowli,
Hyderabad, Telangana 500032

This guide is for the Firebase-ready HTML/CSS/JavaScript restaurant dashboard.

## Included project features

- Admin / Waiter / Chef sign-in using Firebase Email/Password Authentication
- Firestore menu management: add, edit, delete, price, category, image URL and availability
- Staff profiles and active/disable controls
- Secure staff account creation through a Firebase Callable Cloud Function
- 50 table records and QR-code generation
- Live Firestore order listener
- Waiter accept/reject workflow
- KOT print dialog opens only after waiter accepts an order
- Chef workflow: Accepted -> Preparing -> Ready
- Waiter workflow: Ready -> Served

## 1. Register the website in Firebase

1. Open https://console.firebase.google.com/
2. Select the project `the-chap-house`.
3. Open Project settings (gear icon) -> General.
4. Under Your apps, select the Web app and copy its Firebase config.
5. Open `index.html` in VS Code.
6. Find `firebaseConfig` and replace the placeholder values with your project's values.
7. Save the file.

The browser config (apiKey, authDomain, projectId, appId, etc.) is designed to be used in the web app. Never put a Firebase Admin SDK service-account private key in HTML or browser JavaScript.

## 2. Enable Email/Password Authentication

1. Firebase Console -> Build -> Authentication.
2. Click Get started if prompted.
3. Open Sign-in method.
4. Select Email/Password.
5. Enable Email/Password and click Save.

## 3. Create the first Admin login

1. Authentication -> Users -> Add user.
2. Create the restaurant admin email and password.
3. Copy the new user's UID.
4. Open Firestore Database -> Data.
5. Create collection `users`.
6. Set the document ID to the Admin UID you copied.
7. Add these fields:

   - `name` (string): `Restaurant Admin`
   - `email` (string): the admin email you created
   - `role` (string): `admin`
   - `active` (boolean): `true`

8. Save the document.

Do not store the password in Firestore. Firebase Authentication manages passwords.

## 4. Create Firestore Database and publish rules

1. Firebase Console -> Build -> Firestore Database.
2. Create the database if you have not already done so.
3. Open the Rules tab.
4. Replace the editor contents with the complete contents of `firestore.rules`.
5. Click Publish.

Important:
- Menu and table documents are publicly readable so the future customer QR menu can display them.
- Only an active Admin can create/edit/delete menu and table records.
- Only active staff can read orders.
- Customer order creation is intentionally blocked (`allow create: if false`) in this dashboard foundation. Do not change it to `allow create: if true`. A secure customer checkout flow with validation must be implemented before enabling customer order creation.
- Waiter can accept/reject pending orders and mark ready orders as served.
- Chef can move accepted orders to preparing and preparing orders to ready.

## 5. Deploy the staff-creation Cloud Function

Staff creation from the Admin dashboard uses a trusted server-side Cloud Function. Do not create staff accounts by putting Admin SDK credentials in the browser.

Requirements:
- Install Node.js (use a currently supported Node.js LTS version).
- Have access to the Firebase project.
- Cloud Functions deployment may require the Firebase project to be on the Blaze billing plan. Check Firebase's current plan requirements before enabling billing.

From the project root folder in the VS Code terminal:

1. Install Firebase CLI:

   npm install -g firebase-tools

2. Sign in:

   firebase login

3. Select the existing Firebase project:

   firebase use --add

   Choose `the-chap-house` when prompted.

4. Initialize Functions if the project does not already have a `functions` setup:

   firebase init functions

   Choose JavaScript and the existing project. If a `functions` folder already exists, do not overwrite its `index.js`.

5. Open the `functions` folder and install dependencies:

   npm install firebase-functions firebase-admin

6. Make sure the included `functions/index.js` is in the project's `functions` folder.

7. Deploy the staff creation function:

   firebase deploy --only functions:createStaffAccount

The function must check that the caller is an active Admin before creating a Waiter or Chef account. Keep that check in place.

## 6. Run the dashboard locally

1. Open the project folder in VS Code.
2. Start the Live Server extension on `index.html`.
3. In Firebase Console -> Authentication -> Settings -> Authorized domains, add `localhost` if it is not already listed.
4. Sign in with the Admin email and password created in Step 3.

If sign-in fails, check:
- Email/Password provider is enabled.
- Firebase config values in `index.html` are correct.
- The admin Authentication user exists.
- Firestore `users/{ADMIN_UID}` document ID exactly matches the Authentication UID.
- The profile has `role: admin` and `active: true`.
- Browser console for the exact error.

## 7. QR code and public website

QR codes must point to a public HTTPS website URL. A QR code containing `localhost` will not work on customers' phones. Deploy the website to Firebase Hosting or another HTTPS host, then set that public URL in the dashboard before generating table QR codes.

## Current limitations

- This project is currently the Admin/Waiter/Chef dashboard foundation. It does not yet include the customer-facing QR menu checkout flow that creates orders.
- Firestore order creation is blocked intentionally until customer checkout is implemented securely.
- KOT printing currently opens the browser print dialog after waiter acceptance. Silent automatic printing to an 80mm thermal printer requires a local print bridge/service and printer configuration.
- Food images currently use an image URL. Direct upload to Firebase Storage is not included yet.
- Before real restaurant use, add order validation, audit logs, monitoring, and end-to-end tests.
