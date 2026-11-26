# Claude Code prompt: complete all 14 screens, make every control work, and ship a beta APK to friends

Paste everything below the line into Claude Code, from the project root.

---

## Goal

Finish Maitri so that **every screen in the two design boards works end to end**, then produce a **tested, signed beta build** that I can send to about 10 friends. When you're done, nothing in `MAITRI_AUDIT_V2.md` may be marked `fake` or `missing`. Anything still `partial` or `blocked` must show an honest message in the app and be listed in the tester notes.

## Fixed constraints (do not change without asking me)

- **Zero cost.** The backend is Render Free (API) + Neon Free (Postgres/PostGIS) + Cloudflare R2 (private files) + cron-job.org keep-alive, as in `DEPLOYMENT.md`. Firebase stays on the **Spark** plan. Don't add any paid service or anything that needs billing turned on.
- **Design source of truth:**
  - `design-reference/maitri · Light theme · 01–14.svg` and `… Dark theme …`
  - per-screen crops in `design-reference/crops/`
  - Follow them exactly in both themes. Where reality needs a change (e.g. no SMS provider), keep the same layout and style and change only the text.
- **Never show fake data as real.** Maya, Arjun, Asha, Ravi, Rosewood Park, "8 min", "650 m" are placeholders. Every value must come from real data, with honest empty, loading and error states.
- **Ask, don't guess, for accounts, keys, payments or console actions.** Give me exact click-by-click steps. Never ask me to paste secrets into chat. Secrets go in `local.properties` (app) or Render → Environment (server).
- **Work in small commits** on a new branch `beta-complete`. Run the Android unit tests and the backend tests after each phase. Open a PR into `master` at the end. Don't deploy to Render or send builds to anyone without asking me first.

## Step 0: Re-audit (no code changes)

1. Re-read `MAITRI_AUDIT_V2.md` and the current code.
2. For each of the 14 screens × 2 themes, list every visible control with its current status, then compare a fresh emulator screenshot against the crop PNG.
3. Update the audit file, then show me the plan with the order of work. Wait for my OK.

## Step 1: Decisions already made (apply them)

1. **Maps:** replace Esri with **MapLibre + OpenFreeMap** (`https://tiles.openfreemap.org/styles/liberty`). No key, free, commercial use allowed.
   - Dark mode: use a bundled dark style derived from liberty, in the Maitri dark palette.
   - Keep the "© OpenStreetMap contributors" / OpenFreeMap attribution visible.
   - Remove `maitriEsriApiKey` and every Esri endpoint.
   - Distance and ETA: straight-line distance plus a walking estimate, labelled "approx.". "Open in Maps" hands navigation to the phone's maps app.
2. **Screen 03 phone login:** Firebase Phone Auth SMS needs the paid Blaze plan, so on Spark it fails for real users.
   - Replace "Continue with phone" with **"Continue with Google"**, in the same button position, size and style.
   - Keep the phone-number field only as an optional profile field (screen 14).
   - Keep the server's `phone_verified` support in place for later.
   - Email + password stays as drawn.
3. **Circle alerts without a paid SMS provider (screens 02, 05, 08):**
   - **Circle members who use Maitri:** match them by phone number or pairing code and send a high-priority FCM push with a live-tracking link.
   - **Contacts without the app:** when SOS starts, open the phone's SMS app (and offer WhatsApp via a share intent), prefilled with "I need help" + a live-location web link to all SOS-enabled contacts. The user taps Send. Record it as "Sent from your phone" only after the composer returns. Don't claim delivery.
   - **Live-location web link:** a small read-only page served by the API (`/l/<token>`). It shows the last location on an OpenFreeMap map. The token is random, expires when the SOS ends, and the page has no other personal data.
   - **Screen 08 "Circle notified":** show the real per-contact status: Push delivered / Opened, SMS prepared, Not reached.
4. **Volunteers (screens 04, 06):**
   - Only **approved** volunteers who are opted-in and available get nearby SOS requests (fixes R5).
   - Build an **admin review panel**: server-rendered pages at `/admin` on the API.
     - Access is limited to Firebase users with an `admin` custom claim. Give me a one-time script that sets the claim on my own UID.
     - The panel lists applications, opens documents via 5-minute R2 links, and lets me approve or reject with a note. Every action goes in the audit log.
   - **Uploads:** the 5 documents go to R2 via presigned URLs, and the "x of 5 uploaded" count reflects real uploads.
   - **"Download template":** generate a simple volunteer declaration PDF in the app's style, bundled with the app.
5. **Beta access instead of the test allowlist (R1):** add `ENVIRONMENT=beta`.
   - Any verified account can sign up.
   - Real SOS goes only to the user's circle and approved volunteers.
   - Limits: max 3 SOS per user per hour, plus a "false alarm" report button for responders.
   - Show a small "Beta" label on Home.
   - Keep production stricter.
   - Tell me exactly which Render env var to change, and when.
6. **Medical notes:** stored on the server, encrypted at rest (key in an env var). Shown only to the owner, and to an **accepted responder during an active SOS**. Deleted with the account. Show a one-line consent note under the field.
7. **Support email:** `<my email>`. Privacy policy, terms and the account-deletion page: generate them under `docs/site/` for GitHub Pages, and give me the steps to publish them.
8. **Crash reporting:** use **Firebase Crashlytics** (free on Spark) for the app, plus Sentry's free developer plan for the backend, but only if it needs no card. Otherwise log to Render.

## Step 2: Make every control work (one commit per screen)

| Screen | Must work end to end |
|---|---|
| 01 Welcome | Get started → 03. Skip when already signed in. |
| 02 Home | **Bell:** a real in-app inbox (SOS requests, circle requests, check-in reminders, alert status) with an unread badge. **SOS:** hold 3 s with cancel; with no PIN set, still start the SOS and prompt for a PIN afterwards. **Share my trip:** live sharing to the circle with auto-expiry. **Check in:** pick a time; a server job alerts the circle if it's missed (R9). The job must run while Render is awake and catch up after a cold start. **Manage** → 05. **+** → add contact. Contact count comes from the server. |
| 03 Log in / Sign up | Tabs, Google, email/password, show/hide password, forgot password. Terms/Privacy are real links. Errors are friendly. Cold-start "Connecting to Maitri…". |
| 04 Volunteer | Yes / Not now, 5 R2 uploads (camera or file, PDF/JPG/PNG, ≤10 MB, retry and remove), template download, consent, real count, Submit. Application status shown (Pending / Approved / Rejected + note). |
| 05 Circle | Server-backed add, edit, remove, primary and SOS toggle; import from phone contacts (system picker); "Choose who gets your SOS"; pairing code so friends with the app link up. |
| 06 Responder request | High-priority notification (full-screen only where Play policy and Android 14 permission rules allow; otherwise a heads-up notification + sound + vibration pattern). Shows approximate locality + distance band only before acceptance. Accept / Decline / expiry. "Available for nearby requests" toggle. |
| 07 On your way | Real MapLibre map with both positions, approx. distance/ETA, **Bring in help**, **Call** (dial the victim's number when they've consented; otherwise hidden), **Contact emergency services** (dial 112), **Open in Maps**. Coordinates wiped on end or revoke. |
| 08 SOS active | Live map with self + accepted responders, live timer, responders list with approx. ETA, real circle statuses. **Voice:** in-app recording (≤60 s) with a consent prompt and visible recording indicator, uploaded to R2. **Camera:** in-app capture uploaded to R2. **End SOS** with PIN. **Call 112** always visible. Works offline-first, with a queue. |
| 09 Profile | Every row works, including Themes (System / Blush-Light / Dark, persisted), Notifications settings, **Privacy & data**, Help & support (FAQ + email), Sign out. **Privacy & data:** export my data as JSON via the share sheet, and delete account (in-app + web link) with PIN/re-auth, deleting DB rows, R2 objects and the Firebase user (R8). |
| 10 Language | Search; 6 languages; Save applies at once without losing an active SOS. **Full-app** translations for every string, marked for human review. |
| 11 Test vibration | Gentle / Steady / Strong, **persisted**, and used for real SOS / responder alerts. Never sends an SOS. |
| 12 Permissions | Real states for Location (incl. background, with prominent disclosure), Notifications, Contacts, Microphone, Camera. Rows request the permission or deep-link to settings. Battery-optimisation guidance, with OEM-specific tips (Samsung, Xiaomi, Oppo/OnePlus, Vivo). |
| 13 Safety PIN | Set, confirm, show, weak-PIN rejection, change, reset via re-auth. |
| 14 Edit profile | Photo (camera/gallery, crop, R2), name, phone, email (verify-before-update), city, medical notes; Save to server, with validation. |

**Visual pass after the functionality works:** fix every spacing, size, icon and text difference listed in the audit, so each screen matches its crop in both themes. Use the exact SVG icons, not Material icons. Add Paparazzi/Roborazzi screenshot tests for all 28 screen/theme pairs and report the diff per screen.

## Step 3: Backend completion

- Endpoints and migrations for: inbox, missed check-in job, volunteer approval gate, admin panel, account deletion/export, medical-notes encryption, live-location link, beta mode and rate limits.
- Every new route checks the Firebase token and ownership.
- Retention jobs:
  - SOS media deleted after 30 days, matching the R2 lifecycle rules.
  - Locations purged after the SOS plus 7 days.
- Keep Neon able to sleep: no new polling loops that touch the DB while idle (see `DEPLOYMENT.md`).
- Update `API_DOCUMENTATION.md`, `DATABASE_SCHEMA.md` and `DEPLOYMENT.md`.

## Step 4: Final test before the beta (must all pass)

1. **Automated:**
   - `./gradlew app:testDebugUnitTest lintDebug` must pass with zero lint errors.
   - Fix the 8 pre-existing lint errors.
   - Run the full backend pytest suite.
   - Run the screenshot tests.
2. **Release build:** a signed **release APK** with R8 on, using the upload key from `local.properties`. Install it on my phone and check that R8 hasn't broken Firebase, Retrofit/serialization, MapLibre or WorkManager. Add keep rules where needed.
3. **Staging check against the live Render + Neon + R2:** first tell me the Render env vars to set (beta mode, encryption key, etc.) and wait for my "deployed".
4. **Device test matrix.** Write it into `BETA_TEST_REPORT.md` with pass/fail and screenshots:
   - My Samsung **SM-E625F** + one emulator on the lowest supported Android (API 26) + one on API 35/36.
   - Light and dark; English and Hindi.
   - First install → sign up (Google and email) → permissions → PIN → circle → volunteer application.
   - **Three-phone SOS:** victim + circle member + approved volunteer. Check:
     - push received with the screen off
     - accept → on-your-way map
     - location updates
     - voice and photo upload
     - end with PIN
     - all three phones update
   - Cold start: wait 20 min with no traffic, then SOS. The "Connecting…" message and retry must work, and the SOS reaches the server.
   - Offline SOS: airplane mode → SOS → queued → reconnect → synced.
   - Indoors / approximate-location SOS shows a location with an accuracy circle.
   - Check-in missed → circle alerted.
   - Account deletion removes everything (verify DB + R2).
   - App killed, restarted and rotated during an active SOS → state preserved.
5. **Fix everything found,** then run the matrix again.

## Step 5: Ship to friends (free)

- Use **Firebase App Distribution** (free on Spark).
  - Give me the console steps.
  - Add a Gradle task `appDistributionUploadRelease`, with a tester group `friends` and release notes.
  - Alternative: a signed APK file I can share directly. Explain both.
- versionName `0.9.0-beta.1`, versionCode scheme `major*10000 + minor*100 + patch`.
- Write `TESTER_GUIDE.md` (also as a short in-app "Beta info" screen under Help). It covers:
  - how to install
  - that Maitri is a beta and **not a replacement for 112**
  - how to test SOS safely: tell your circle first, end it with the PIN
  - known limitations (cold start, contacts without the app get SMS only when you tap Send, etc.)
  - how to report bugs: in-app "Send feedback" → email with device info + logs, no personal data
- Before sending: a final checklist confirming no test UIDs, debug menus or fake data remain, and that all secrets are out of git (run a secret scan).

## Definition of done

- All 14 screens work end to end in both themes and match the designs.
- `MAITRI_AUDIT_V2.md` has no `fake` or `missing` rows.
- All tests pass, and `BETA_TEST_REPORT.md` is complete.
- The signed beta build is uploaded to App Distribution (after I approve).
- `TESTER_GUIDE.md` is ready to send.

Start with Step 0 and show me the plan. Don't edit code until I approve.
