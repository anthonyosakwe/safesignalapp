# SafeSignal — Deploy Instructions

This version of the app loads the Supabase SDK from your own repo instead of a CDN. This makes it **reliable on Nigerian mobile networks** where CDNs (jsDelivr, unpkg, cdnjs) are sometimes blocked or slow.

You only have to do this **once.** After that, deploys are just `git push`.

---

## One-time setup

### Step 1: Download the Supabase SDK

On your phone or any device with internet, open this URL in a browser:

**https://cdn.jsdelivr.net/npm/@supabase/supabase-js@2/dist/umd/supabase.js**

The browser will display a wall of JavaScript code. Save the page:
- **Chrome on Android:** menu (⋮) → Download (↓)
- **Desktop browsers:** Right-click → Save Page As → save as `supabase.js`

The file should be ~140KB.

If that URL is blocked on your network, try one of these alternates:
- https://unpkg.com/@supabase/supabase-js@2/dist/umd/supabase.js
- https://cdnjs.cloudflare.com/ajax/libs/supabase-js/2.39.0/supabase.min.js

### Step 2: Add it to your repo

In your `safesignalapp` GitHub repo, upload the downloaded `supabase.js` file to the **root** of the repo (same folder as `index.html`).

Your repo root should look like:

```
safesignalapp/
├── index.html       ← (the new file from this chat)
├── supabase.js      ← (downloaded in Step 1)
└── README.md
```

### Step 3: Push and test

After committing both files, GitHub Pages will redeploy automatically (1–2 minutes).

Open https://anthonyosakwe.github.io/safesignalapp/ on your phone, submit a test report, then check the Supabase **Table Editor → reports** — your test row should be there.

---

## Why this matters

Loading the SDK from `cdn.jsdelivr.net` (or any third-party CDN) means your app's reliability is at the mercy of:
- The user's mobile carrier (MTN, Glo, 9mobile each route differently)
- DNS quirks at peak hours
- Cross-border traffic to CDN edges that aren't in Africa

Loading the SDK from your own GitHub Pages domain means: **if your page loads, the SDK loads.** Same network conditions, same origin.

The trade-off: when Supabase releases a new SDK version, you'd manually re-download `supabase.js` and re-commit it. For an MVP demo, that's a fine trade.

---

## Going to production later

When you're ready to ship this beyond the demo:
- Move to a proper bundler (Vite, esbuild) with versioned dependencies
- Add a service worker for offline-first behaviour (queue reports locally, sync when online)
- Consider Supabase Edge Functions for the audio upload (rather than base64 in a text column)

For the prototype/portfolio version, this setup is exactly right.
