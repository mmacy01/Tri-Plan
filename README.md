# Two to the Line — Sprint Triathlon Plan (Mike & Taylor)

A 12-week beginner sprint-triathlon training app that **syncs live between two phones**. Each of you gets your own tab (Mike / Taylor), checks off your own workouts, and see progress + a race-day countdown. It installs like a real app and works offline.

- **Cost: $0.** GitHub Pages is free for public repos; Firebase runs on the free **Spark** plan, which needs **no payment method on file** — so it literally cannot bill you. Two people ticking off workouts use a tiny fraction of the free daily limits.
- **Sync:** both phones load the same page, which reads/writes one Firestore document. A live listener pushes changes to the other phone within about a second.
- **Offline:** the app caches itself, so it opens with no signal. Your checks save locally and resync when you're back online.

---

## What's in this folder

| File | What it is |
|---|---|
| `index.html` | The app. **This is the only file you edit** (paste your Firebase config near the top). |
| `manifest.webmanifest` | Makes it installable as an app. |
| `sw.js` | Service worker — offline support. |
| `icon-192.png`, `icon-512.png`, `icon-maskable-512.png`, `apple-touch-icon.png` | App icons. |
| `firestore.rules` | The security rules to paste into Firebase. |
| `README.md` | This guide. |

Upload **all** of these to your repo, with `index.html` at the top level.

---

## Part A — Firebase (about 4 minutes)

You can reuse the same Firebase project as one of your World Cup apps — you just need Firestore turned on. Or make a fresh one; steps below cover both.

1. Go to **https://console.firebase.google.com** and sign in.
2. **New project?** Click **Add project**, name it (e.g. `two-to-the-line`), accept defaults, Create. (Skip Google Analytics if it asks — not needed.)
   **Reusing a project?** Just open it.
3. Register a **web app**: on the project overview page, click the **`</>`** (web) icon. Give it a nickname (e.g. `tri-web`). **Do not** check "Firebase Hosting." Click **Register app**.
4. Firebase shows a `firebaseConfig` block that looks like this — **keep this tab open, you'll copy it in Part B:**
   ```js
   const firebaseConfig = {
     apiKey: "AIza...",
     authDomain: "your-project.firebaseapp.com",
     projectId: "your-project",
     storageBucket: "your-project.appspot.com",
     messagingSenderId: "1234567890",
     appId: "1:1234...:web:abcd..."
   };
   ```
   (If you closed it: **Project settings** ⚙️ → scroll to **Your apps** → your web app → **Config**.)
5. Turn on the database: left sidebar → **Build → Firestore Database → Create database**.
   - Choose a location close to Texas (e.g. **nam5 (us-central)**). *This can't be changed later.*
   - Pick **Start in production mode** → **Create**. (Don't worry, you'll set the rules next.)
6. Set the rules: in **Firestore Database → Rules** tab, delete what's there and paste the contents of **`firestore.rules`** (also shown here), then click **Publish**:
   ```
   rules_version = '2';
   service cloud.firestore {
     match /databases/{database}/documents {
       match /plans/{code} {
         allow read, write: if true;
       }
     }
   }
   ```
   > These rules only expose the single shared plan document, gated by knowing the exact sync code (the document id). Keep that code non-obvious and you're fine for a private two-person app.

---

## Part B — Paste your config into the app (1 minute)

1. Open **`index.html`** in any text editor.
2. Near the top of the `<script>` section you'll see:
   ```js
   const FB = {
     apiKey:            "PASTE_API_KEY",
     authDomain:        "PASTE_PROJECT.firebaseapp.com",
     projectId:         "PASTE_PROJECT_ID",
     storageBucket:     "PASTE_PROJECT.appspot.com",
     messagingSenderId: "PASTE_SENDER_ID",
     appId:             "PASTE_APP_ID"
   };
   const SYNC_CODE = "mike-taylor-sprint-2026";
   ```
3. Replace each `PASTE_...` value with the matching value from your `firebaseConfig` (Part A, step 4).
4. **`SYNC_CODE`** — leave it, or change it to something less guessable (e.g. add random characters). Both phones share this one file, so they'll automatically match. **If you ever change it after you've started training, you'll start from a fresh empty plan.**
5. Save the file.

---

## Part C — GitHub Pages (about 3 minutes)

1. Go to **https://github.com/new** (signed in as **mmacy01**).
2. Name the repo, e.g. `tri-plan`. Set it **Public**. Click **Create repository**.
3. On the new repo page: **Add file → Upload files**. Drag in **all the files from this folder** (the individual files — make sure `index.html` lands at the repo root, not inside a subfolder). **Commit changes**.
4. **Settings** (top tab) → **Pages** (left sidebar).
5. Under **Build and deployment → Source**, pick **Deploy from a branch**. Set **Branch: `main`**, folder **`/ (root)`** → **Save**.
6. Wait ~1 minute, refresh the Pages screen, and it'll show your live URL:
   **`https://mmacy01.github.io/tri-plan/`**
7. Open that URL on your computer. The little pill by the logo should turn green and say **Synced**. If it says **This device**, jump to Troubleshooting below.

---

## Part D — Install on both phones

Send Taylor the URL (`https://mmacy01.github.io/tri-plan/`). Then on each phone:

**Android (Chrome):**
1. Open the URL.
2. Tap the **⋮** menu → **Add to Home screen** (or **Install app**).
3. Confirm. You'll get an app icon that launches full-screen — no browser bar.

**iPhone (Safari):**
1. Open the URL.
2. Tap the **Share** button → **Add to Home Screen** → **Add**.

Tip: on your phone, tap your own name tab once — the app remembers which of you is using that phone and opens to your tab next time.

---

## Cost, in one line

Stays on Firebase **Spark (free, no card)** and **GitHub Pages (free)**. Your usage is nowhere near the free daily limits (50,000 reads / 20,000 writes per day). You will not be charged. Keep it running as long as you like.

---

## Troubleshooting

**Pill says "This device," not "Synced."**
- Double-check every `PASTE_...` value was replaced in `index.html` and there are no leftover quotes/typos.
- Make sure Firestore is actually **created** (Part A, step 5) and the **rules were Published** (step 6).
- Re-upload `index.html` to GitHub after editing, then hard-refresh the page.

**Checks aren't showing up on the other phone.**
- Both phones must open the **same deployed URL** (they share the same `SYNC_CODE` automatically). Refresh both.
- If one phone is offline, its checks sync once it's back online.

**I updated a file but the app looks the same.**
- The service worker caches the app. To force everyone onto the new version, open `sw.js`, change `const CACHE = 'ttl-v1';` to `'ttl-v2'`, re-upload, then close and reopen the app.

**Want it locked down harder later?**
- Swap the open rules for Firebase **Anonymous Authentication** + a rule like `allow read, write: if request.auth != null;`. Ask and I'll wire it up.

---

*General fitness guidance for healthy beginners — not medical advice. Worth a quick doctor's OK before starting, especially coming back from surgery or injury. Now go get that finish line. 🏁*
