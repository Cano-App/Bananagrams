# Working with Joe on apps

Joe builds small web apps from his phone. He wants each one live and linked up with as
little effort on his side as possible. Keep explanations short and in plain words, and
give phone tap-by-tap steps when he has to do something himself.

## Default setup for every new app

1. **One file:** the whole app is a single `index.html` in the repo root.
2. **Hosting:** commit to `main` and serve it with GitHub Pages from `main` / `(root)`.
   - Live URL: `https://<owner-lowercase>.github.io/<RepoName>/`
   - Every push to `main` goes live in about a minute, so code changes deploy themselves.
   - The cloud session can't turn Pages on through the API (403). Check the
     "pages build and deployment" workflow runs; if there are none, give Joe the taps:
     Settings → Pages → Deploy from a branch → `main` → `/ (root)` → Save.
3. **Shared data:** if players or users need to see each other's data, use Joe's Firebase
   project **`bananagrams-4b5f0`** (Firestore; the free plan is enough). Don't ask him to
   sign up for anything new.
   - Give each app its own top-level collection. Firestore rules then need a `match`
     block for it: write the rules out and tell Joe where to paste them
     (Firestore → Rules → Publish).
   - The web config below is meant to be public. Security comes from the rules.
   - Load the SDK from `https://www.gstatic.com/firebasejs/10.12.2/` with dynamic
     `import()`, and use transactions when several phones write to the same document.
   - People open the GitHub link with no sign-in, so don't rely on Claude-only features.
4. **Test before saying it's done:** run Playwright (Chromium at `/opt/pw-browsers`).
   Simulate several players by giving each its own browser context and a mock Firestore,
   and run one real save to Firebase. Then report what was and wasn't tested.

```js
// Joe's Firebase web config (project bananagrams-4b5f0)
{ apiKey:"AIzaSyAZHx61X7Eh9rVL62N9juocoQRZbFPS4DU", authDomain:"bananagrams-4b5f0.firebaseapp.com",
  projectId:"bananagrams-4b5f0", storageBucket:"bananagrams-4b5f0.firebasestorage.app",
  messagingSenderId:"665533055457", appId:"1:665533055457:web:d093f201a5f95c7efad2a0" }
```

## Bananagrams specifics

- Live at https://cano-app.github.io/Bananagrams/ with games stored under `games/<code>`.
  The current Firestore rules only allow the `games` collection, with fixed fields.
- The photo reader (`BGReader`) runs on the phone itself. Test reader changes against
  real photos, because computer-drawn boards missed real problems such as paint specks
  on the table.
