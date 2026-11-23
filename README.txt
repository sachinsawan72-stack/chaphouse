# THE CHAP HOUSE - Restaurant Management (Updated)

## Folder
- `public/index.html`  -> poora website (yehi deploy hoga)
- `firestore.rules`    -> Firebase Firestore security rules
- `firebase.json`      -> Hosting + rules config

## Naye features
1. Orders: Date From / To filter, Today, All dates, date-wise total summary, CSV download (har order ke saath Captain/Waiter/Taken-by naam aur role).
2. Staff roles: Manager, Captain, Waiter, Cashier, Chef (Admin ke saath).
3. Captain dashboard: My Orders, Today's Orders, Order Status, Order History.
   Waiter dashboard: My Orders, Accepted Orders, Table Number, Order History.
4. Order tracking: orderId, tableNumber, createdAt, captainName, waiterName, status, takenByName/Role.
6. **Staff delete:** Sirf ADMIN "Staff Accounts" page se staff ka account Delete kar sakta hai (Manager ya koi aur nahi). Delete hote hi staff login nahi kar paata, aur agar logged-in hai to turant logout ho jata hai. Admin apna ya kisi dusre admin ka account delete nahi kar sakta. Rules me bhi yehi lock hai.
5. "+ New order" button: Captain/Waiter/Cashier/Manager khud table ka order le sakte hain (KOT print hota hai).

## Roles aur adhikaar
| Role    | Access |
|---------|--------|
| Admin   | Sab kuch |
| Manager | Poora management (Staff, Tables, Reports, Menu, Orders) |
| Captain | Sirf apne orders + naye pending QR orders accept karna |
| Waiter  | Sirf apne orders + naye pending QR orders accept karna |
| Cashier | Sabhi orders, Bill, Print, Email |
| Chef    | Sirf Kitchen (Preparing / Ready) |

## Rules kaise lagayein
**Tarika 1 (Console):** Firebase Console -> Firestore Database -> Rules -> `firestore.rules` ka poora text paste karein -> Publish.

**Tarika 2 (CLI):**
```
npm i -g firebase-tools
firebase login
firebase use the-chap-house
firebase deploy --only firestore:rules,hosting
```

## Zaroori
- Pehla Admin: Firebase Console me user banayein, phir Firestore `users/<UID>` me document banayein:
  `{ name: "Admin", email: "...", role: "admin", active: true }`
- Naye staff Admin/Manager ke "Staff Accounts" page se banenge.
- Staff ka `users` document hona zaroori hai, warna login nahi hoga.
- Delete se staff ka `users` profile hata hota hai, isse wo system me kaam nahi kar sakta. Firebase Authentication me email entry bachi rehti hai; use poori tarah hatana ho to Firebase Console -> Authentication -> Users se delete karein (ya wahi email dobara use karna ho to pehle wahan se hatayein).
- Purane orders me Captain/Waiter naam khali dikhega; naye orders me bharega.
- Customer QR order me jo Captain/Waiter pehle "Accept" karega, uska naam order me jud jayega.
- Rules me "apne orders" ka filter app me hai; rules orders ko staff ke liye readable rakhte hain
  (Firestore list query me per-user filter ke liye query ko bhi filter karna padta hai).
