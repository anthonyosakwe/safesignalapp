# SafeSignal — Product Requirements Document

**Status:** Shipped (MVP v1.0)
**Owner:** Dr. Anthony Onyema Osakwe — Clinical Product Manager
**Last updated:** May 1, 2026
**Live demo:** [anthonyosakwe.github.io/safesignalapp](https://anthonyosakwe.github.io/safesignalapp/)
**Repo:** [github.com/anthonyosakwe/safesignalapp](https://github.com/anthonyosakwe/safesignalapp)

---

## 1. Summary

SafeSignal is a real-time mobile incident reporting tool that lets citizens report harassment, robbery, violence, domestic incidents, and suspicious activity in under 60 seconds — anonymously by default. Reports sync instantly to a safety coordinator's admin dashboard for triage and action. The app is designed for distressed users on Nigerian mobile networks, where bandwidth is unreliable and every second of friction matters.

This document captures what was built, why it was built that way, what shipped, and what's deferred — written after delivery rather than before, because the build itself surfaced product decisions that wouldn't have been visible from a whiteboard.

---

## 2. Problem statement

### The user problem

Someone witnessing or experiencing an incident — harassment on public transport, an attempted robbery, a domestic dispute they can hear through a wall — typically does one of three things:

1. **Nothing.** The friction of reporting (call who? say what? give my name?) outweighs the perceived benefit.
2. **Tells a friend.** Social processing happens, but the incident never reaches anyone who can act.
3. **Calls an emergency line.** Often only when the situation is already severe; rarely for "suspicious" or "ongoing" patterns.

The gap is between observation and action. Most safety incidents are reportable in principle but not in practice — the reporting tools assume a calm, unhurried user with time to authenticate, type, and explain. Real reporters are stressed, in motion, and afraid of being identified.

### The coordinator problem

Safety coordinators (community leads, hospital security officers, NGO field staff, neighborhood watch organizers) currently rely on WhatsApp groups, phone calls, and informal text threads. Reports arrive unstructured, without timestamps or location, and follow-up is manual. There's no shared view of what's open, what's been actioned, or what patterns are emerging.

### Why now

Mobile penetration in Nigeria crossed 90% in 2024. Smartphones are the dominant computing device. Yet most safety apps are still designed for stable Wi-Fi, English literacy, and an unhurried user — none of which describes the average MTN 4G user reporting a real incident.

---

## 3. Goals and non-goals

### Goals

- **G1.** Sub-60-second submission from app open to confirmation, on 3G.
- **G2.** Anonymous reporting as the default path, not a hidden option.
- **G3.** Real-time delivery to coordinator dashboard (<2 seconds end-to-end).
- **G4.** Zero-install (web app, no app store friction).
- **G5.** Zero-cost to operate at MVP scale (free tier infrastructure only).
- **G6.** Reliable on Nigerian mobile networks specifically — not just "works on Wi-Fi."

### Non-goals (explicitly deferred)

- **NG1.** User authentication or accounts. Anonymous-by-default is the value proposition; adding optional accounts is post-MVP.
- **NG2.** SMS or push notifications to trusted contacts. UI placeholder only.
- **NG3.** Heat maps, trend dashboards, or analytics. The MVP scope is point-in-time reporting and triage, not aggregate intelligence.
- **NG4.** End-to-end encryption beyond TLS-in-transit. The threat model assumes Supabase as a trusted operator, not adversarial.
- **NG5.** Multi-tenancy or per-organization deployment. Single instance for the prototype.
- **NG6.** Native iOS/Android apps. Web only.
- **NG7.** Automated escalation rules ("if status = new for >24h, alert lead").
- **NG8.** Integration with police, emergency services, or third-party CRMs.

---

## 4. Target users

### Primary: Citizen reporter

- **Demographics:** Adult mobile-first user in an urban Nigerian context (Abuja, Lagos, Port Harcourt). Owns a smartphone with a 3G/4G data plan. Comfortable with WhatsApp and Instagram, less so with formal apps.
- **Context of use:** Often on a moving bus, in a market, near home. Distressed or vigilant. Battery may be low. Network may be congested.
- **Goal:** Tell someone who can act — quickly, without identifying themselves, with enough detail that the report is useful.
- **Anti-goal:** Being identified, harassed, or held accountable for a report that turns out to be a false alarm.

### Secondary: Safety coordinator

- **Demographics:** Community lead, hospital security officer, NGO field officer, or neighborhood watch organizer. Mid-30s to 50s. Comfortable with desktop and mobile web.
- **Context of use:** Usually at a desk or station. Checks the dashboard reactively (notifications, group chats prompting them) and proactively (start of shift, end of day).
- **Goal:** Triage incoming reports, mark them through a workflow, identify patterns over time.
- **Anti-goal:** Spending more time managing the tool than acting on the reports.

---

## 5. User journeys

### Journey 1: Anonymous incident report (citizen)

1. User experiences or observes an incident. Opens the SafeSignal URL (bookmarked, shared link, or QR code).
2. Sees the home screen — a single primary CTA: "Report an incident."
3. Taps it. Lands on the report flow.
4. Selects incident type from 6 chips (harassment, robbery, violence, domestic, suspicious, other).
5. Types a brief description (max 500 chars). Optional, but encouraged via subtle character counter.
6. Optionally records up to 30 seconds of audio. Optionally toggles photo/video evidence slots (visual indicators only in MVP).
7. GPS captures location automatically. If GPS fails, manual entry is offered.
8. Submits. Sees a success screen with a reference ID (`SS-XXXXXX`).
9. Total time on a normal connection: 30-50 seconds.

### Journey 2: Coordinator triage (safety officer)

1. Coordinator opens the same URL on desktop or tablet, switches to "Admin Dashboard" view.
2. Sees stats (Incoming reports: X new, Y this week) and a filterable list (All / New / In review / Actioned).
3. New report arrives. List updates in real-time without refresh — new card animates in at top.
4. Coordinator taps the card to open detail modal: full description, timestamp, location with map preview, audio playback if attached, anonymous status.
5. Decides on next action. Updates status: New → In review → Actioned → Closed.
6. Status change persists to database immediately. If multiple coordinators are viewing, their dashboards update too.

### Journey 3: Failed-network recovery (citizen)

1. User starts a report on weak 3G. Composes type, description, location.
2. Submits. App attempts insert to Supabase. Network drops mid-request.
3. App falls back to local-only mode: report is held in browser memory and shown on success screen.
4. (Future enhancement: queue and retry. See section 11.)

---

## 6. Functional requirements

### 6.1 Citizen mobile flow

| ID | Requirement | Status |
|---|---|---|
| F-CR-01 | Single-tap entry from home screen to report flow | ✅ Shipped |
| F-CR-02 | 6 incident type categories with visual chips | ✅ Shipped |
| F-CR-03 | Free-text description, max 500 characters, with live counter | ✅ Shipped |
| F-CR-04 | 30-second audio recording via MediaRecorder API | ✅ Shipped |
| F-CR-05 | 4-slot evidence grid (2 photos, 1 video, 1 audio) | ✅ Shipped (audio functional, photo/video are visual placeholders) |
| F-CR-06 | Automatic GPS capture with reverse geocoding | ✅ Shipped |
| F-CR-07 | Manual location entry as GPS fallback | ✅ Shipped |
| F-CR-08 | Anonymous toggle, ON by default | ✅ Shipped |
| F-CR-09 | Generated reference ID in `SS-XXXXXX` format on submission | ✅ Shipped |
| F-CR-10 | Success screen confirming submission | ✅ Shipped |
| F-CR-11 | Pre-flight permission check for geolocation (handles previously-denied state with clear remediation) | ✅ Shipped |
| F-CR-12 | Progressive enhancement: coordinates shown immediately, address upgraded asynchronously | ✅ Shipped |

### 6.2 Coordinator admin flow

| ID | Requirement | Status |
|---|---|---|
| F-AD-01 | Dashboard with stats (new count, this-week count) | ✅ Shipped |
| F-AD-02 | Real-time report list, sorted newest-first | ✅ Shipped |
| F-AD-03 | Filter tabs (All / New / In review / Actioned) | ✅ Shipped |
| F-AD-04 | Tap-to-open detail modal | ✅ Shipped |
| F-AD-05 | Status workflow: New → In review → Actioned → Closed | ✅ Shipped |
| F-AD-06 | Status changes persist to database | ✅ Shipped |
| F-AD-07 | Audio playback in modal when audio attached | ✅ Shipped |
| F-AD-08 | Cross-client real-time updates (one coordinator's status change appears on others' screens) | ✅ Shipped |
| F-AD-09 | Mock baseline reports (5) preserved as permanent demo content | ✅ Shipped |
| F-AD-10 | Delete report from admin modal | ⏳ Deferred to v1.1 |
| F-AD-11 | Realtime DELETE event handling | ⏳ Deferred to v1.1 |

### 6.3 Cross-cutting

| ID | Requirement | Status |
|---|---|---|
| F-X-01 | View toggle: Mobile App / Admin Dashboard / Side-by-side (for portfolio demos) | ✅ Shipped |
| F-X-02 | Single-page deployment (one HTML file, no build step) | ✅ Shipped |
| F-X-03 | Self-hosted Supabase SDK (no third-party CDN dependency) | ✅ Shipped |
| F-X-04 | Automated weekly SDK updates via GitHub Actions | ✅ Shipped |

---

## 7. Non-functional requirements

| ID | Requirement | Target | Actual |
|---|---|---|---|
| NF-01 | Time to interactive on 3G | <3s | ~2s (single HTML, same-origin SDK) |
| NF-02 | End-to-end submission time | <60s | 30-50s observed |
| NF-03 | Real-time sync latency | <2s | <1s observed (Supabase Realtime) |
| NF-04 | First-meaningful-paint on cold load | <2s | ~1.5s |
| NF-05 | App functional with JS-only (no bundler, no transpiler) | Required | ✅ |
| NF-06 | Resilient to CDN failures | Required after MVP testing exposed the issue | ✅ Self-hosted SDK |
| NF-07 | Audio file size per recording | <500KB | ~300KB (30s WebM) |
| NF-08 | Free-tier infrastructure only | Required | ✅ Supabase free tier, GitHub Pages |

---

## 8. Architecture and technical decisions

### 8.1 Stack

- **Frontend:** Vanilla HTML, CSS, JavaScript. No framework, no bundler, no build step.
- **Backend:** Supabase (managed PostgreSQL + Realtime + REST API).
- **Hosting:** GitHub Pages (static).
- **CI/CD:** GitHub Actions for automated SDK updates.

### 8.2 Why this stack

The decision tree:

- **Vanilla JS over React:** The app has 6 screens, ~5 stateful interactions, and no need for component reuse or virtual DOM optimization. React would have added a build step, ~50KB of runtime, and zero functional benefit. Vanilla JS keeps the surface area small and the deploy artifact a single HTML file.
- **Supabase over custom backend:** The MVP needs PostgreSQL, REST inserts, real-time pubsub, and TLS — all of which Supabase provides on a free tier with zero ops burden. Building a custom Node/Express + Postgres + WebSocket stack would have been weeks of work for the same outcome.
- **GitHub Pages over Netlify/Vercel:** Pages is free, ties deployment to git push, and integrates with the same account as the repo. No vendor lock-in for a static site.

### 8.3 The CDN incident — and the decision it forced

During testing on Nigerian mobile networks (MTN 4G), the Supabase JavaScript client failed to load from `cdn.jsdelivr.net`. Browser console showed only "Script error." (the cross-origin error mask). Falling back to `unpkg.com` failed similarly. `cdnjs.cloudflare.com` failed intermittently. The app appeared to work — the page rendered, buttons responded — but every "submitted" report silently fell back to local-only mode and never reached the database.

This took several debugging cycles to diagnose because the failure was silent: a defensive stub I'd written to prevent crashes was masking the root cause. Once identified, the fix was structural: **stop relying on infrastructure I don't control on networks that are hostile to it.**

The Supabase SDK is now hosted in the same repo as the HTML file, served from the same GitHub Pages origin. The trade-off — manual SDK updates — is automated via GitHub Actions running weekly.

This is the kind of decision that doesn't appear in a pre-build PRD. It came out of the build itself, and it shaped the final architecture more than any pre-flight requirement did.

### 8.4 Realtime as the single source of truth

When a citizen submits a report, the local UI does **not** optimistically insert into the dashboard list. Instead, the row goes to Supabase, and the realtime channel echoes it back. Both the citizen's local view (if they had one) and every connected coordinator dashboard receive the same broadcast.

This avoids the duplicate-row bug that occurs when both optimistic insert and realtime echo run on the same client — a subtle issue that broke the UI in early testing.

### 8.5 Audio as base64 in a text column

Audio recordings are encoded as base64 strings and stored in a `text` column on the `reports` table, rather than uploaded to Supabase Storage. For 30-second WebM clips at typical bitrates, this is ~300KB per row.

The trade-off:
- **Pro:** No second API call, no signed URLs, no bucket policies, single insert. Aligns with the speed goal.
- **Con:** Doesn't scale past prototype use. Production would migrate to Storage with signed-URL playback.

This is documented in the codebase and called out in the README as a deliberate prototype choice.

---

## 9. Database schema

```sql
create table reports (
  id              bigserial primary key,
  created_at      timestamptz default now(),
  report_ref      text,                        -- SS-XXXXXX format
  incident_type   text,                        -- harassment / robbery / violence / domestic / suspicious / other
  description     text,                        -- max 500 chars (enforced client-side)
  location_name   text,                        -- reverse-geocoded place
  coordinates     text,                        -- display string e.g. "9.0578° N, 7.4951° E"
  latitude        float8,                      -- numeric for queries
  longitude       float8,
  is_anonymous    bool default true,
  media_count     int2 default 0,
  audio           text,                        -- base64 WebM (optional)
  status          text default 'new'           -- new / review / actioned / closed
);
```

Row-Level Security is **disabled** in this prototype. Anyone with the publishable anon key can insert. For production, RLS policies would:
- Allow `INSERT` from anonymous role (citizens reporting)
- Restrict `SELECT` and `UPDATE` to authenticated coordinators
- Prevent `DELETE` except by admin role

---

## 10. Privacy, security, and threat model

### What is protected
- TLS 1.3 in transit (GitHub Pages and Supabase both enforce HTTPS).
- Anonymous-by-default — no IP logging in the app, no cookies, no fingerprinting.
- Reference IDs are non-sequential random alphanumeric, not enumerable from the URL.

### What is not protected (explicitly out of scope)
- Reports are visible to anyone with the Supabase anon key (which is in the client-side code, by design — that's how Supabase publishable keys work). This is acceptable for a public reporting prototype but inappropriate for sensitive deployments.
- No end-to-end encryption. Supabase is treated as a trusted operator.
- No protection against adversarial Supabase access (state-level threat actors, subpoenas, etc).
- The Supabase `anon` role can read all reports. In production, RLS would prevent this; in the prototype, it's an explicit acceptance.

### What's risky
- The base64 audio in the database is functionally a permanent recording of a possibly distressed reporter's voice. There's no automatic expiration. For production, recordings should expire after coordinator action or fixed TTL.
- Coordinates are precise to ~10m. For a domestic violence report, this could deanonymize the reporter. Consider geographic fuzzing (round to ~500m) for sensitive incident types.

These risks are documented and accepted at MVP scope. They become blocking before any real deployment.

---

## 11. What's deferred — and the reasoning

### v1.1 (next iteration, ~1 week of work)

- **Delete report from admin modal** with confirmation. Coordinators need to remove false reports, duplicates, and test data without leaving the app.
- **Realtime DELETE event handling** so deletions sync across dashboards.
- **Image and video evidence upload** wired through to Supabase Storage. UI slots already exist; backend is the missing piece.
- **Trusted contacts feature** (SMS alert to a saved phone number on submission).
- **Safety tips content section.**

### v1.2 (further out)

- **Heat map view** showing incident clustering by neighborhood. Deprioritized in RICE scoring because trend analysis isn't useful until volume is meaningful (~100+ reports), which the prototype won't reach.
- **Coordinator authentication** with role-based access. Required before any public deployment.
- **Multi-tenancy** so different organizations can run their own SafeSignal instance with isolated data.
- **Offline-first PWA with queue/retry.** Service worker that holds reports in IndexedDB when network is unavailable, syncs when it returns. This is non-trivial and needs its own design doc.
- **Localization** for Yoruba, Hausa, Igbo at minimum. Pidgin English would be high-impact.

### Permanently out of scope

- Native iOS/Android apps. Web is the right platform for the user — zero-install matters more than platform features.
- Integration with formal emergency services (police, fire, NEMA). The legal and operational complexity is far beyond what this tool's scope can support, and a wrong implementation could create dangerous expectations.

---

## 12. Success metrics

For an MVP / portfolio piece:

- ✅ Submission flow works end-to-end on a real Nigerian mobile network
- ✅ Real-time sync demonstrably functional (visible in side-by-side demo view)
- ✅ Time-to-first-report under 60 seconds on cold load
- ✅ Zero infrastructure cost (verified — entire stack runs on free tiers)
- ✅ Self-maintaining (SDK auto-updates, no manual intervention needed)

For a hypothetical pilot deployment:

- **Adoption:** ≥ 50 submitted reports in the first 30 days from a single neighborhood pilot
- **Triage:** ≥ 80% of reports moved out of "New" status within 24 hours
- **Reporter experience:** Median submission time ≤ 60 seconds (measured client-side)
- **Coordinator experience:** Time-to-first-status-change ≤ 4 hours during business days
- **Reliability:** ≥ 99% submission success rate (defined as: report visible in coordinator dashboard within 5 seconds of submit)

These pilot metrics aren't claimed for the current build — they're what would be measured in a real deployment.

---

## 13. Risks and mitigations

| Risk | Likelihood | Impact | Mitigation |
|---|---|---|---|
| Supabase free tier limits hit (500MB DB, 2GB bandwidth/month) | Low at MVP scale | Medium | Monitor in dashboard; migrate audio to Storage before hitting limits |
| CDN dependencies break on user's network | High (already happened) | High | ✅ Mitigated — SDK self-hosted, auto-updated weekly |
| Supabase service outage | Low | High | Document fallback (local-only mode degrades gracefully); accept for MVP |
| Malicious bulk submissions (spam, DoS) | Medium | Medium | Currently unmitigated; would need rate limiting via Supabase Edge Functions in v1.1 |
| Real reporter de-anonymization via coordinates | Medium | Critical for sensitive incidents | Document risk; recommend geographic fuzzing for v1.1 |
| Reviewer dismisses project as "vibe-coded prototype" | Medium | Medium | This PRD and the README explicitly document architectural choices and trade-offs to demonstrate intentionality |

---

## 14. Decisions log

Significant product decisions made during the build, with rationale:

| Decision | Date | Reasoning |
|---|---|---|
| Anonymous-by-default with opt-in identification | Pre-build | Removes the largest source of user friction (identity exposure) without removing the option |
| 4 evidence slots: 2 photos, 1 video, 1 audio | Pre-build | Visually communicates "you can attach evidence" without overwhelming; matches user mental model |
| 6 incident types with chip selection | Pre-build | Exhaustive enough to cover most real reports; small enough to be visually scannable in 2 seconds |
| Status workflow with 4 states (New → In review → Actioned → Closed) | Pre-build | Mirrors operational reality without becoming a project management tool |
| Vanilla JS, no framework | Pre-build | Project size doesn't justify framework overhead; deploy simplicity is a feature |
| Supabase as backend | Pre-build | Realtime + REST + auth-ready, zero ops, free tier |
| Self-host Supabase SDK | Mid-build | Forced by CDN failures on Nigerian mobile networks |
| Realtime as single source of truth (no optimistic insert) | Mid-build | Discovered duplicate-row bug from optimistic insert + realtime echo |
| Audio as base64 in column, not Storage | Pre-build | Single API call vs two; acceptable for prototype |
| Mock baseline reports stay permanently | Mid-build | Portfolio demo needs to look populated even on a fresh database |
| Progressive geolocation (coords first, address second) | Mid-build | Nominatim reverse geocoder was the slowest path; decoupling unblocked the user |

---

## 15. What this project actually demonstrates

For a Clinical Product Manager portfolio reviewer, the relevant signal isn't "can this person build a small web app." Most products at this scope are within reach of any competent engineer. The relevant signals are:

- **Problem framing.** The PRD opens with a user problem rooted in a specific Nigerian context, not a generic "users want safety." The constraints (3G networks, distressed reporters, anonymity) shape every downstream decision.
- **Scope discipline.** The non-goals list is longer than the goals list. Heat maps, authentication, native apps, push notifications — all visible from the start, all deliberately deferred. An MVP that ships is worth more than a product spec that doesn't.
- **Trade-off literacy.** Audio in base64 vs Storage. Vanilla JS vs React. Optimistic insert vs realtime-as-truth. Each is documented with the trade-off explicit and the rationale recorded.
- **Engineering judgment under pressure.** The CDN incident wasn't anticipated. Diagnosing it took several debugging cycles, and the fix — self-hosting the SDK — became the most consequential architectural decision in the project. The build surfaced reality that planning couldn't.
- **Operational thinking.** GitHub Actions for weekly SDK updates with version pinning, sanity checks, and revert paths. Free-tier infrastructure verified end-to-end. A README that explains decisions, not just usage. A deployed app that maintains itself.

Concretely: this project shipped a working, real-time, database-backed application on hostile network conditions, with proper deployment hygiene — auto-updating SDK, version pinning, fallback handling, and no manual maintenance burden. That's an artifact a stakeholder can poke at, not a Figma file.

---

## 16. Appendix

**Tech artifacts:**
- Live app: https://anthonyosakwe.github.io/safesignalapp/
- Repo: https://github.com/anthonyosakwe/safesignalapp
- Database: Supabase project `SafeSignal` (eu-west-1, t4g.nano)

**Document history:**
- 2026-05-01: Initial PRD written post-delivery, capturing actual decisions made during the build.

---

*End of document.*
