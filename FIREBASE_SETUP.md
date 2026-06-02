# Firebase Setup Guide — Live Endorsements for HaChipush

This guide walks you through connecting your landing page to Firebase so readers
can log in with Google and submit endorsements in real time.

---

## What You're Setting Up

| Feature | Tool |
|---|---|
| Google login | Firebase Authentication |
| Storing endorsements | Firebase Firestore |
| Approving endorsements | Firestore Console (manual) |

Everything is **free** on Firebase's Spark plan (up to 50,000 reads/day).

---

## Step 1 — Create a Firebase Project

1. Go to [https://console.firebase.google.com](https://console.firebase.google.com)
2. Click **"Add project"**
3. Name it `hachipush` (or anything you like)
4. Disable Google Analytics if you don't need it → click **"Create project"**
5. Wait ~30 seconds for setup to finish → click **"Continue"**

---

## Step 2 — Register Your Web App

1. On the project overview page, click the **`</>`** (Web) icon
2. Give it a nickname: `hachipush-web`
3. Leave "Firebase Hosting" unchecked (unless you want to host there)
4. Click **"Register app"**
5. You'll see a code block like this — **copy it somewhere safe**:

```js
const firebaseConfig = {
  apiKey: "AIzaSy...",
  authDomain: "hachipush.firebaseapp.com",
  projectId: "hachipush",
  storageBucket: "hachipush.appspot.com",
  messagingSenderId: "123456789",
  appId: "1:123456789:web:abc123"
};
```

6. Click **"Continue to console"**

---

## Step 3 — Enable Google Sign-In

1. In the left sidebar → **Build → Authentication**
2. Click **"Get started"**
3. Under **"Sign-in method"** tab, click **"Google"**
4. Toggle **"Enable"** to ON
5. Fill in **"Project support email"** (use your email)
6. Click **"Save"**

---

## Step 4 — Create Firestore Database

1. In the left sidebar → **Build → Firestore Database**
2. Click **"Create database"**
3. Choose **"Start in test mode"** → click **"Next"**
   > ⚠️ Test mode expires after 30 days. See Step 7 to set permanent rules.
4. Choose the closest server location (e.g., `europe-west1` for Israel)
5. Click **"Enable"**

---

## Step 5 — Paste Your Config into the HTML

1. Open `hachipush_landing.html` in a text editor (Notepad, VS Code, etc.)
2. Press **Ctrl+F** and search for: `YOUR_API_KEY`
3. You'll find this block near the bottom of the file:

```js
const firebaseConfig = {
  apiKey:            "YOUR_API_KEY",
  authDomain:        "YOUR_PROJECT_ID.firebaseapp.com",
  projectId:         "YOUR_PROJECT_ID",
  storageBucket:     "YOUR_PROJECT_ID.appspot.com",
  messagingSenderId: "YOUR_SENDER_ID",
  appId:             "YOUR_APP_ID"
};
```

4. Replace each `"YOUR_..."` value with the real values from Step 2
5. **Save the file**

The yellow warning banner on the page will disappear once the config is correct.

---

## Step 6 — Add Your Domain to Authorized Domains

If your site is hosted at a custom domain (e.g., `hachipush.co.il`):

1. Firebase Console → **Authentication → Settings → Authorized domains**
2. Click **"Add domain"**
3. Enter your domain: `hachipush.co.il`
4. Click **"Add"**

> If you're just opening the HTML file locally (file://...) this step isn't needed yet.

---

## Step 7 — Set Firestore Security Rules (Important!)

After 30 days, test mode expires. Set these rules now to be safe:

1. Firestore → **Rules** tab
2. Replace the default rules with:

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /endorsements/{doc} {

      // Anyone can read approved endorsements
      allow read: if resource.data.approved == true;

      // Logged-in users can create one endorsement (can't overwrite others)
      allow create: if request.auth != null
                    && request.resource.data.uid == request.auth.uid
                    && request.resource.data.text.size() > 0
                    && request.resource.data.text.size() <= 500;

      // Only you (admin) can update/delete — set your UID below
      // allow update, delete: if request.auth.uid == "YOUR_ADMIN_UID";
    }
  }
}
```

3. Click **"Publish"**

> To get your admin UID: log into your site with Google, then check
> Firestore Console → endorsements collection → your document → `uid` field.

---

## Step 8 — Approving Endorsements

When a reader submits an endorsement, it's saved with `approved: false`.
To approve it and make it visible on the site:

1. Go to Firestore Console → **endorsements** collection
2. Click on the pending document
3. Click the `approved` field → change `false` to `true`
4. Click **"Update"** — it appears on the site **instantly** (no reload needed)

---

## Testing Checklist

- [ ] Open `hachipush_landing.html` in a browser
- [ ] Yellow warning banner is **gone**
- [ ] Click "✍️ כתוב המלצה משלך"
- [ ] Google login popup appears and works
- [ ] Submit a test endorsement
- [ ] Check Firestore Console — new document appears with `approved: false`
- [ ] Approve it in Firestore — it appears on the page live

---

## Troubleshooting

| Problem | Fix |
|---|---|
| Yellow banner still showing | Config values weren't saved correctly |
| Google popup blocked | Allow popups for your domain in browser settings |
| "Permission denied" error | Check Firestore security rules (Step 7) |
| Endorsements not loading | Make sure the `approved` field is boolean `true`, not string `"true"` |

---

## File Structure

```
hachipush-project/
├── hachipush_landing.html   ← your landing page (edit this)
├── book_cover.jpg           ← place your book cover image here
└── FIREBASE_SETUP.md        ← this guide
```

---

*Guide prepared for israeld@fintek.co.il — June 2026*
