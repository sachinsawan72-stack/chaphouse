# THE CHAP HOUSE - Restaurant Management (Full Security Update)

## Folder
- `public/index.html`  -> poora website (yehi Vercel/Hosting par deploy hoga)
- `firestore.rules`    -> **hardened** Firebase Firestore security rules
- `firebase.json`      -> Hosting + rules config

---

## Is update me kya-kya hardened kiya gaya hai

### 1. Staff profiles (`users`)
- Koi bhi non-admin, kabhi bhi, kisi ko bhi `role: 'admin'` nahi bana sakta — chahe create ho ya update,
  app se ho ya direct Firestore API call se.
- Ek non-admin **Manager** kisi ko `manager` bhi promote/ create nahi kar sakta (sirf Admin kar sakta hai).
  Isse ek rogue Manager apne jaise aur Manager nahi bana sakta (privilege-escalation band).
- Kisi bhi existing **Admin** ka document Manager edit/delete nahi kar sakta.
- **Sirf Admin** staff account **delete** kar sakta hai — na khud ka, na kisi doosre Admin ka.
- `create`/`update` me sirf tay kiye gaye fields (`name,email,role,active,...`) hi allow hain — koi extra/
  unexpected field inject nahi ho sakta.

### 2. Orders — customer (QR) se aane wale order
- Sirf exact fields allow hain jo customer menu page bhejta hai — koi extra field (jaise fake
  `captainId` ya `acceptedBy`) inject nahi ho sakta.
- Order hamesha `status: 'Pending'` se hi shuru ho sakta hai — koi customer seedha "Accepted"/"Served"
  order nahi bana sakta.
- `items` array ka har item check hota hai: naam non-empty string, price 0 se 1,00,000 ke beech,
  quantity 1 se 50 ke beech, aur max 20 alag items per order.
- Bill hamesha default (`no discount, S.C. applied`) se hi start hota hai — customer khud discount
  lagakar order nahi bhej sakta.
- `captainId`/`waiterId` khali aur `takenByRole: 'customer'` hi ho sakta hai — koi customer khud ko
  staff member banakar order nahi daal sakta.

### 3. Orders — staff dwara seedha liya gaya order ("+ New order")
- Sirf non-chef staff order bana sakta hai, aur `takenById` / `acceptedBy` hamesha apna hi UID hona
  zaroori hai (koi doosre staff ke naam se order nahi daal sakta).
- Discount/S.C. hatana sirf **Admin / Manager / Cashier** kar sakte hain, bina default bill ke
  Captain/Waiter order save nahi kar paayenge.

### 4. Orders — billing (discount, GST/S.C., bill email) sirf Cashier/Manager/Admin ke liye
Aapki original role-table yahi kehti hai: **"Cashier -> Billing and orders", "Captain/Waiter -> apne
orders"**. Ab ye sirf UI me nahi, Firestore rules me bhi lock hai:
- `bill`, `total`, `subtotal`, `customerEmail` fields sirf Admin/Manager/Cashier change kar sakte hain.
- Captain/Waiter order ko Accept/Reject/Preparing/Ready/Served tak move kar sakte hain, note add kar
  sakte hain, khud ko Captain/Waiter assign kar sakte hain — par Bill modal me Save/Print/Email karne
  par unhe "permission denied" milega (jaisa role-table me chaha gaya hai).
- `billEmailedAt` field is se bahar rakha gaya hai, kyunki jab koi bhi floor staff order "Served" karta
  hai to app khud-ba-khud bill email bhejkar us par timestamp save karta hai — isse wo flow nahi tootega.

### 5. Chef
- Chef sirf `status` ko `Preparing`/`Ready` me badal sakta hai, aur kuch nahi — na dusre orders ki
  list access, na billing, na staff/menu/tables.

### 6. Menu & Tables
- Menu sabke liye readable hai (QR customer ke liye zaroori), par sirf Admin/Manager likh sakte hain.
- Tables sirf staff padh sakta hai, sirf Admin/Manager likh sakte hain.

### 7. Default-deny
- File ke bilkul end me ek catch-all rule hai jo kisi bhi aisi collection/path ko band kar deta hai jo
  upar explicitly allow nahi ki gayi — isse galti se koi naya open collection ban jaye to bhi wo safe
  rehta hai.

---

## Ek zaroori limitation (jaan-boojh kar accept ki gayi)
Firestore Rules me hum har order item ki price ko live Menu document se match **nahi** karwa rahe
(ye Rules language me possible hai par bahut complex aur fragile ho jaata hai). Iska matlab: theory
me koi QR customer browser console se manually ek dish ki price ghata kar bhej sakta hai.

**Isliye asli safety net ye hai:** har QR order "Pending" hi rehta hai jab tak koi Waiter/Captain use
"Accept & print KOT" na kare. Staff ko accept karne se pehle order items aur price screen par dikhte
hain — wahi insaan check hai jo galat/chhoti price ko pakad sakta hai. Agar aap chahen to future me
ek Cloud Function add karwa sakte hain jo har naye order ki price ko Menu se cross-check karke galat
order ko turant reject kar de — ye zero-trust level ki extra suraksha hai, abhi ke rules me nahi hai.

---

## Deploy karne se pehle — ZAROORI testing steps
Ye rules pehle se zyada sakht hain. Agar app ke code aur rules ke field-names me koi mismatch reh gaya,
to kuch cheez (jaise order place karna) turant "Missing or insufficient permissions" error dega.

1. Firebase Console -> Firestore Database -> **Rules** tab -> upar right me **"Rules Playground"**
   button se kuch sample requests (jaise ek customer order create, ek staff order accept) simulate
   karke dekh lein ki "Allow" aa raha hai.
2. Pehle ek **test/staging Firebase project** par publish karke poori app chala kar dekh lein:
   - QR se order place karna
   - Waiter/Captain se Accept karna
   - Kitchen se Preparing/Ready karna
   - Cashier se Bill save/print/email karna
   - Admin se naya staff banana, disable karna, delete karna
3. Sab sahi chalne ke baad hi production project par **Publish** karein.

## Rules kaise lagayein
**Console se:** Firebase Console -> Firestore Database -> Rules -> `firestore.rules` ka poora text
paste karke Publish karein.

**CLI se:**
```
npm i -g firebase-tools
firebase login
firebase use the-chap-house
firebase deploy --only firestore:rules,hosting
```

## Zaroori
- Pehla Admin Firebase Console se manually banayein, Firestore `users/<UID>` me:
  `{ name: "Admin", email: "...", role: "admin", active: true }`
- Naye staff Admin/Manager ke "Staff Accounts" page se banenge.
- Staff ka `users` document hona zaroori hai, warna login nahi hoga.
- Purane (is update se pehle ke) orders me Captain/Waiter naam khali dikhega.
- (Optional UX improvement, zaroori nahi): Captain/Waiter ko "Bill" button hi na dikhayein UI me,
  taaki unhe permission-denied error kabhi na aaye — abhi button dikhta hai par click karne par
  rules usse rok dengi.
