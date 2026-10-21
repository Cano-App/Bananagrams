---
name: new-app
description: Joe's one-step setup for a new web app. Use whenever Joe says "new app", "make an app", "start an app", "build me a ... app", or wants something new put online. It makes a single-page app, puts it on GitHub Pages, connects his Firebase for shared data when needed, tests it, and gives him the live link plus any taps he has to do himself.
---

# New app for Joe

Joe works from his phone. He wants to say what the app does and get a working live link,
with as few steps on his side as possible. Keep messages short and in plain words. When he
has to do something himself, give numbered, phone tap-by-tap steps.

## 1. Find out what to build (one question at most)

If he hasn't said what the app is for, ask one short question. Otherwise pick sensible
defaults and go: a name, a phone-first layout, light and dark mode.

## 2. Make the repo

- Repo name: short, in PascalCase or kebab-case. The owner is Joe's GitHub account or
  org (currently the user account `Cano-App`; the API gets a 403 creating repos, so expect
  to give Joe the steps below).
- If a GitHub tool can create the repo (for example `mcp__github__create_repository`),
  create it as public, with a README.
- Otherwise give Joe these steps, then carry on once he says it's done:
  github.com → **+** → **New repository** → name → **Public** → tick **Add a README**
  → **Create repository**. If this session can't reach the new repo, he starts a new
  Claude Code session with that repo selected and types "new app" again.

## 3. Build it

- One file: `index.html` in the repo root, with all CSS and JS inline. Libraries load
  only from CDNs, pinned to exact versions.
- Phone-first, at least 16px side padding, no sideways scrolling, light and dark mode.
- Commit to `main` and push. Pages serves `main`, so every later push goes live by itself.

## 4. Shared data (only if users need to see each other's data)

Scores, leaderboards, multiplayer, sign-up lists, shared lists and the like.

- Use Joe's existing Firebase project **`bananagrams-4b5f0`** (Firestore, free plan).
  Never ask him to sign up for anything.
- Give the app its own top-level collection, named after the app. Everyone opens the
  GitHub link with no sign-in.
- Load the SDK with dynamic `import()` from
  `https://www.gstatic.com/firebasejs/10.12.2/firebase-app.js` and `firebase-firestore.js`.
  Use `onSnapshot` for live updates, and `runTransaction` when several people write to the
  same document.
- Firestore rules are **one shared file for all of Joe's apps**, and publishing replaces
  the whole file. So always give him the **complete** rules: every existing app's block
  below, plus the new one. Then add the new block to the list below, in this skill and
  in your reply, so the next app keeps it. Steps for him:
  console.firebase.google.com → **Bananagrams** → **Firestore Database** → **Rules** →
  delete everything → paste → **Publish**.
- Write each block as tightly as the app allows: `hasOnly` on the field names, size
  limits, and `allow delete: if false` unless deleting is needed.

```js
// Web config (meant to be public; the rules protect the data)
{ apiKey:"AIzaSyAZHx61X7Eh9rVL62N9juocoQRZbFPS4DU", authDomain:"bananagrams-4b5f0.firebaseapp.com",
  projectId:"bananagrams-4b5f0", storageBucket:"bananagrams-4b5f0.firebasestorage.app",
  messagingSenderId:"665533055457", appId:"1:665533055457:web:d093f201a5f95c7efad2a0" }
```

Current rules: every app's block goes inside the one `documents` match.

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    // Bananagrams (cano-app.github.io/Bananagrams)
    match /games/{gameId} {
      allow read: if true;
      allow create, update: if request.resource.data.keys().hasOnly(
        ['state','names','totals','rounds','target','winner','saved','best'])
        && request.resource.data.state is string
        && request.resource.data.state.size() < 250000;
      allow delete: if false;
    }
    // FlightDeals (cano-app.github.io/FlightDeals): watches people add; the checker writes prices
    match /flightwatches/{id} {
      allow read: if true;
      allow create, update: if request.resource.data.keys().hasOnly(
        ['from','to','depart','ret','flex','any','nights','cabin','airlines','stops','target','adults','by','created',
         'checked','price','low','hist','best','byAir','cal','deal','dealAt','err','alerted'])
        && request.resource.data.from is string && request.resource.data.from.size() == 3
        && request.resource.data.to is string && request.resource.data.to.size() == 3
        && request.resource.data.depart is string && request.resource.data.depart.size() <= 10
        && request.resource.data.cabin in ['economy','premium-economy','business','first']
        && (!('airlines' in request.resource.data) || request.resource.data.airlines.size() <= 30)
        && (!('hist' in request.resource.data) || request.resource.data.hist.size() <= 60)
        && (!('cal' in request.resource.data) || request.resource.data.cal.size() <= 20)
        && (!('by' in request.resource.data) || request.resource.data.by.size() <= 30);
      allow delete: if true;
    }
    // FlightDeals: deal-site headlines (written once, never changed)
    match /flightnews/{id} {
      allow read: if true;
      allow create: if request.resource.data.keys().hasOnly(['title','link','src','t','tags'])
        && request.resource.data.title is string && request.resource.data.title.size() <= 300
        && request.resource.data.link is string && request.resource.data.link.size() <= 500;
      allow update, delete: if false;
    }
  }
}
```

## 5. Turn on GitHub Pages

- The session usually can't turn Pages on through the API (403). After pushing, look for
  a "pages build and deployment" run with the GitHub Actions tools.
- If there isn't one, Joe does this once per repo: repo → **Settings** → **Pages** →
  Source **Deploy from a branch** → Branch **main**, **/ (root)** → **Save**.
- The live URL is `https://<owner-lowercase>.github.io/<RepoName>/`. Confirm the
  deployment run succeeded before saying it's live.

## 6. Test before saying it's done

- Use Playwright with the preinstalled Chromium (`/opt/pw-browsers`) at phone size
  (390×844). Check that there are no console errors, the main flow works, and nothing
  scrolls sideways.
- For shared data, simulate two or three users, each in its own browser context, against
  a mock Firestore served through `page.route`. Then do one real write to the actual
  project (if the sandbox blocks gstatic, use `npm pack firebase@10.12.2` files).
- Say plainly what was tested and what wasn't (for example, real phones or real photos).

## 7. What to tell Joe at the end (short)

1. The live link.
2. Any taps he still has to do (repo, Pages, Firebase rules), as numbered steps.
3. One or two lines on how to use the app, and what was tested.
