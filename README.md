# RFP / Enquiry Tracker — Deployment Guide

This folder has one file that matters: **`index.html`**. It's the whole app — no build step, no npm install. It talks to a free Firebase database instead of Claude's storage, so it works as a normal website on GitHub Pages.

## Part 1 — Create your Firebase project (5–10 minutes, no cost)

1. Go to https://console.firebase.google.com and sign in with any Google account (a personal one is fine — this doesn't need to be tied to ESDS's Google Workspace).
2. Click **Add project**, name it something like `esds-rfp-tracker`, and finish the wizard (you can skip Google Analytics).
3. In the left sidebar, click **Build → Firestore Database**, then **Create database**. Choose **Start in test mode** for now — pick the region closest to your team (e.g. `asia-south1 (Mumbai)`).
4. In the left sidebar, click the **gear icon → Project settings**. Under "Your apps," click the **`</>`** (Web) icon to register a new web app. Name it anything (e.g. "tracker"). You don't need Firebase Hosting — skip that checkbox.
5. Firebase will show you a `firebaseConfig` object that looks like this:
   ```js
   const firebaseConfig = {
     apiKey: "AIza...",
     authDomain: "esds-rfp-tracker.firebaseapp.com",
     projectId: "esds-rfp-tracker",
     storageBucket: "esds-rfp-tracker.appspot.com",
     messagingSenderId: "123456789",
     appId: "1:123456789:web:abcdef"
   };
   ```
   Copy this whole block.

## Part 2 — Drop your config into the app

1. Open `index.html` in any text editor.
2. Near the top (inside the `<head>`), find this block:
   ```js
   const firebaseConfig = {
     apiKey: "YOUR_API_KEY",
     authDomain: "YOUR_PROJECT_ID.firebaseapp.com",
     ...
   };
   ```
3. Replace it entirely with the real config block you copied from Firebase.
4. Save the file.

## Part 3 — Set Firestore security rules

Test mode rules expire after 30 days, so switch them to something permanent before that happens.

1. In Firebase Console → **Firestore Database → Rules**, replace the contents with:
   ```
   rules_version = '2';
   service cloud.firestore {
     match /databases/{database}/documents {
       match /opportunities/{doc} {
         allow read, write: if true;
       }
     }
   }
   ```
2. Click **Publish**.

**Worth knowing:** this makes the tracker's data open to anyone who has the site URL and knows to look at the Firestore API — there's no login screen. That's a reasonable tradeoff for an internal team tool with no highly sensitive data, but if you want a lock on the door later (e.g. requiring sign-in before read/write), say the word and I'll wire up Firebase Authentication — it's a small addition on top of this.

## Part 4 — Push to GitHub Pages

1. Create a new **public** repository on GitHub (e.g. `rfp-tracker`).
2. Upload `index.html` to the root of that repository (GitHub's web UI lets you drag-and-drop — no need for git command line if you don't want it).
3. Go to the repo's **Settings → Pages**.
4. Under "Build and deployment," set **Source** to **Deploy from a branch**, branch = `main`, folder = `/ (root)`. Save.
5. GitHub will give you a URL like `https://yourusername.github.io/rfp-tracker/` within a minute or two.
6. Open that URL — the tracker should load and seed itself with the original 75 opportunities on first visit.

## Part 5 — Share it with the team

Send the GitHub Pages URL to Yogesh's team. Everyone who opens it:
- Types their name once (remembered on their device from then on)
- Sees the same live board/table — additions, edits, and drag-and-drop status changes appear for everyone within a second or two, since it's now backed by Firestore's real-time sync (this is actually smoother than the Claude-artifact version was)

## Optional: custom domain

If ESDS wants `tracker.esds.co.in` instead of the `github.io` link, that's a DNS change your IT team makes (a CNAME record pointing at your GitHub Pages URL) plus one settings field in the repo's Pages settings — a 10-minute job for whoever manages ESDS's DNS, no Azure/app-registration complexity involved.
