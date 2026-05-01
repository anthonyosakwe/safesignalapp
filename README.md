# SafeSignal

> Real-time mobile safety incident reporting, built for low-bandwidth networks.

**Live demo:** [anthonyosakwe.github.io/safesignalapp](https://anthonyosakwe.github.io/safesignalapp/)

A portfolio prototype by [Dr. Anthony Onyema Osakwe](https://github.com/anthonyosakwe) — Clinical Product Manager, Abuja.

---

## What it does

SafeSignal lets citizens report harassment, robbery, violence, domestic incidents, or suspicious activity in under 60 seconds — anonymously by default. Reports sync instantly to a safety coordinator's admin dashboard, where status can be tracked through a simple workflow (New → In review → Actioned → Closed).

Designed around three constraints that shape every decision:

- **Speed under stress.** A person reporting an incident is often distressed. The flow is one tap to start, four steps to submit, with sane defaults at every stage.
- **Privacy by default.** Anonymous reporting is on by default. No login, no account, no identifier collection unless the reporter explicitly opts in.
- **Resilient on weak networks.** Built and tested on Nigerian mobile networks (MTN, Glo) where CDN reliability is unpredictable. The app is structured so that if the page loads, every dependency loads.

---

## Features

- 📱 4-step incident report flow — type, description, evidence, location
- 🎙 30-second audio recording with in-browser MediaRecorder
- 📍 GPS capture with progressive enhancement (coordinates first, address second)
- 🔒 Anonymous-by-default with explicit opt-in for identification
- ⚡ Real-time sync to admin dashboard via Supabase Realtime
- 🖥 Coordinator dashboard with stats, filters, and per-report status updates
- 📊 4-step status workflow (New → In review → Actioned → Closed)
- 🌐 Works on 3G/4G — designed for sub-60-second submission

---

## Architecture

**Stack:** Vanilla HTML, CSS, JavaScript. No build step, no framework, no bundler.

**Backend:** [Supabase](https://supabase.com) (managed PostgreSQL + Realtime + REST API).

**Hosting:** GitHub Pages (static).

**Why this stack:**

The whole app is one HTML file plus the Supabase SDK. No npm, no Webpack, no React, no transpilation. This is deliberate — the project is a real piece of software for a clinician in a low-resource environment, not a developer-experience showcase. Fewer moving parts means fewer things that can break, and a single-file architecture deploys to GitHub Pages with zero infrastructure.

```
Browser (Mobile App + Admin Dashboard, same page)
   │
   ├─→ Supabase REST API ──→ PostgreSQL (reports table)
   │
   └─← Supabase Realtime ←─ pg_changes broadcast
```

When a citizen submits a report, the browser inserts directly into the `reports` table via the Supabase REST API. The Realtime channel then broadcasts the new row to every other connected client — including the safety coordinator's admin dashboard — within ~1 second. No backend code, no message queue, no server.

### Self-hosted Supabase SDK

The Supabase JavaScript client is hosted in this repo (`supabase.js`) and loaded same-origin, not from a CDN. This is unusual but intentional.

**Why:** During testing on Nigerian mobile networks, three different CDNs (jsDelivr, unpkg, Cloudflare) each failed unpredictably — sometimes blocked entirely, sometimes throttled to multi-second load times. For a safety reporting app where speed is the entire value proposition, "the SDK loaded slowly" is a product failure.

Loading the SDK from `anthonyosakwe.github.io/safesignalapp/supabase.js` — the same origin as the HTML — means the SDK is subject to exactly the same network conditions as the page itself. If the page loaded, the SDK loaded. Period.

Trade-off: when Supabase ships a new SDK version, the local file needs updating. That's automated (see below).

### Automated SDK updates

A GitHub Actions workflow (`.github/workflows/update-supabase-sdk.yml`) runs every Monday at 04:00 WAT. It:

1. Checks npm for the latest `@supabase/supabase-js` v2.x release
2. Downloads the UMD bundle from jsDelivr (which works fine from GitHub's runners — the issue is mobile-network-specific)
3. Sanity-checks the file (size, contains `createClient` export)
4. Commits the updated file back to this repo, but only if it changed
5. GitHub Pages auto-redeploys

The workflow pins to v2.x — it won't auto-pull a future v3.0 with breaking changes. You can also trigger it manually from the Actions tab.

---

## Project structure

```
safesignalapp/
├── index.html                              # The entire app (~2,500 lines)
├── supabase.js                             # Self-hosted Supabase SDK v2.39.7
├── .github/
│   └── workflows/
│       └── update-supabase-sdk.yml         # Weekly SDK auto-update
├── README.md
└── LICENSE
```

That's it. Three files of code, no `node_modules`, no `dist/`, no build artifacts.

---

## Database schema

The `reports` table has 13 columns:

| Column         | Type        | Notes                                         |
| -------------- | ----------- | --------------------------------------------- |
| id             | int8        | Auto-incrementing primary key                 |
| created_at     | timestamptz | Defaults to `now()`                           |
| report_ref     | text        | Human-readable ID, format `SS-XXXXXX`         |
| incident_type  | text        | harassment / robbery / violence / domestic / suspicious / other |
| description    | text        | Up to 500 characters                          |
| location_name  | text        | Reverse-geocoded place name                   |
| coordinates    | text        | Display string e.g. `9.0578° N, 7.4951° E`    |
| latitude       | float8      | Numeric for sorting/queries                   |
| longitude      | float8      |                                               |
| is_anonymous   | bool        | Defaults to `true`                            |
| media_count    | int2        | Number of attached evidence items             |
| audio          | text        | Base64-encoded WebM audio (optional)          |
| status         | text        | new / review / actioned / closed              |

Row-Level Security is disabled in this prototype — anyone with the publishable key can insert. For production, RLS policies would restrict reads to safety coordinators while keeping inserts open to anonymous citizens.

---

## Design decisions worth calling out

**Audio as base64 in a text column, not Supabase Storage.** Storage is the right answer for production, but for a prototype with 30-second clips, base64-in-column avoids signed URLs, bucket policies, and an extra API roundtrip. Migration path is straightforward when needed.

**Mock baseline reports stay permanently.** The dashboard ships with five pre-populated mock reports (Wuse 2 robbery, Maitama domestic incident, etc.) that never delete. Real reports stack on top. This keeps the portfolio demo looking populated even on a fresh database.

**Realtime as the single source of truth for inserts.** When a report is submitted, the local UI does NOT optimistically insert. Instead, the row goes to Supabase, and the realtime channel echoes it back to populate the dashboard. This avoids the duplicate-row bug that occurs when both optimistic insert and realtime echo run on the same client.

**Progressive geolocation.** GPS capture shows coordinates within 3-8 seconds, then upgrades to a friendly address name when (or if) Nominatim's reverse geocoder responds. The user is never blocked waiting on a slow third-party API.

---

## Built by

Dr. Anthony Onyema Osakwe — Clinical Product Manager.

Background: medical doctor (MBBS, University of Lagos), MDCN-licensed, ACLS/PALS certified. PMP. Currently building [ABC Specialist Hospital Limited](https://github.com/anthonyosakwe) (Abuja) and exploring the intersection of clinical practice and product engineering.

This project was built as a portfolio piece to demonstrate end-to-end product thinking: framing the problem (incident reporting in distressed users), constraining the scope (MVP in days, not months), and shipping working software with real infrastructure (database, realtime sync, automated maintenance).

---

## License

MIT — see [LICENSE](LICENSE).
