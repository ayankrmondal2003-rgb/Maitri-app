<div align="center">

# 🩷 maitri
### Here for you. Safety, together.

**A community-powered Android personal-safety application**  
*Designed and developed by Team 4 Loops*

[![Platform](https://img.shields.io/badge/Platform-Android-3DDC84?logo=android&logoColor=white)](app/)
[![Kotlin](https://img.shields.io/badge/Language-Kotlin-7F52FF?logo=kotlin&logoColor=white)](app/src/main/java/in/maitri/app/)
[![Backend](https://img.shields.io/badge/API-FastAPI-009688?logo=fastapi&logoColor=white)](backend/)
[![Status](https://img.shields.io/badge/Status-Beta-orange)](BETA_TEST_REPORT.md)
[![Design](https://img.shields.io/badge/Design-Figma-F24E1E?logo=figma&logoColor=white)](https://www.figma.com/design/Tsvh9SuF1INsZyS7EEnTFr/maitri?node-id=0-1)

[**Explore the Figma design**](https://www.figma.com/design/Tsvh9SuF1INsZyS7EEnTFr/maitri?node-id=0-1) · [**Architecture**](PROJECT_HANDBOOK.md) · [**Setup**](SETUP.md) · [**Beta test report**](BETA_TEST_REPORT.md) · [**Privacy**](PRIVACY.md)

</div>

> [!IMPORTANT]
> **Beta / safety disclaimer:** Maitri is an experimental personal-safety and community-response platform, **not** a certified emergency-response system or a substitute for police, ambulance, or other official emergency services. It cannot guarantee network availability, responder availability, notification delivery, rescue, or continuous GPS operation. **In India, dial 112 for an emergency.** Current beta and deployment features must be interpreted alongside [known limitations](KNOWN_LIMITATIONS.md) and [beta testing evidence](BETA_TEST_REPORT.md).

## Short description

**maitri** is a native Android app built around a three-second SOS action, a moving **1 km** nearby-responder discovery radius, an opt-in trusted circle, and privacy-controlled location sharing. It combines Firebase identity and push notifications, a FastAPI backend, PostGIS proximity matching, and WebSocket-based incident updates with a carefully designed Light and Dark interface.

## Team 4 Loops

**Team name:** 4 Loops  
**Project:** maitri — Android personal-safety and community-response application  
**Repository:** [ayankrmondal2003-rgb/Maitri-app](https://github.com/ayankrmondal2003-rgb/Maitri-app)  
**GitHub maintainer:** [@ayankrmondal2003-rgb](https://github.com/ayankrmondal2003-rgb)

Team member names and individual responsibilities have not been listed here because they have not been provided. Add them once approved by the team.

## Contents

1. [Objective and problem](#objective-and-problem)
2. [Our differentiator](#our-differentiator)
3. [Current feature guide](#current-feature-guide)
4. [Architecture](#architecture)
5. [System flow](#system-flow)
6. [Technology stack](#technology-stack)
7. [Limitations](#limitations)
8. [Screenshot walkthrough](#screenshot-walkthrough)
9. [Run locally](#run-locally)
10. [Deployment](#deployment)
11. [Android development](#android-development)
12. [Data preparation](#data-preparation)
13. [Testing](#testing)
14. [Controls and suggested walkthrough](#controls-and-suggested-walkthrough)
15. [Repository structure](#repository-structure)
16. [Future work](#future-work)
17. [AI use disclosure](#ai-use-disclosure)
18. [Credits and license](#credits-and-license)

## Objective and problem

During an emergency, a person may have difficulty explaining their location, reaching trusted contacts, or finding someone nearby willing and able to help. A static alert does not necessarily reflect a moving person's latest position; broadcasting precise coordinates too broadly also creates privacy risks.

Maitri explores a more accountable response model:

- Start an SOS with a deliberate **three-second press-and-hold**.
- Track an active incident and discover eligible, available volunteers within a **moving 1,000 m radius**.
- Inform selected trusted contacts and provide appropriately authorized location updates.
- Require explicit responder acceptance before releasing precise incident coordinates.
- End the incident through server-validated **Safety PIN** closure.

The goal is to **improve coordination and user control** in a safety-oriented prototype, not to claim guaranteed emergency intervention.

## Our differentiator

| Capability | What makes it distinctive |
|---|---|
| **Moving 1 km discovery area** | PostGIS searches around the victim's **latest** qualifying location as they move, rather than treating the SOS origin as fixed. |
| **Accepted-responder continuity** | A responder who has accepted and remains authorized can continue receiving live incident updates after the victim moves outside their original 1 km discovery area. |
| **Location privacy by default** | Before accepting, a responder receives an invitation with approximate information, **not precise victim coordinates**. The server checks authorization again for subsequent reads and WebSocket access. |
| **Trusted circle** | Users manage their own contacts, alert preferences, paired accounts, check-ins, and consent-based trip sharing. |
| **Server-controlled closure** | Safety PIN validation and incident/grant revocation are handled by the backend. |
| **Human-centered interface** | The Figma design specifies **14 screens in Light and Dark themes**, implemented as native Jetpack Compose views rather than screenshots with touch overlays. |

## Current feature guide

The repository contains the following user-facing flows. **Code presence and automated tests do not mean every third-party integration or device scenario is production-verified.**

| Area | Current guide |
|---|---|
| **Welcome and accounts** | Welcome screen, Firebase-backed account flows (including email/Google integrations), profile sync, and account management. Phone OTP depends on Firebase provider/billing configuration. |
| **Home / SOS** | Three-second hold to initiate SOS; active incident screen, location updates, live status, and a Call 112 fallback. |
| **Moving geofence** | Server-side 1 km PostGIS matching and ongoing responder discovery as the incident location changes. |
| **Responder** | Volunteer application/review, availability controls, invitation inbox, accept/decline, authorized journey, and status updates. Live volunteering requires approval and appropriate permissions. |
| **Trusted circle** | Contact picker, manual contact management, notification preferences, account pairing, and circle-related views. |
| **Trip sharing and check-ins** | Consent-based sharing sessions, expiration/revocation, in-app inbox and check-in history/read state, subject to backend connectivity and recipient configuration. |
| **Evidence and documents** | Private-file upload flows are designed for volunteer documents, SOS voice/photo evidence and profile photos; these require configured private R2 storage and applicable user permissions. |
| **Safety and settings** | Six-digit Safety PIN, permissions, vibration patterns, notifications, privacy controls, profile editing, Light/Dark themes, and six-language selection (English, Hindi, Bengali, Marathi, Tamil, Telugu). |
| **Notifications and mapping** | Firebase Cloud Messaging; MapLibre maps using OpenFreeMap tiles; authenticated HTTP and real-time WebSocket updates. Delivery and maps depend on the device/network/provider. |

See [PROJECT_HANDBOOK.md](PROJECT_HANDBOOK.md), [API_DOCUMENTATION.md](API_DOCUMENTATION.md), and [BETA_TEST_REPORT.md](BETA_TEST_REPORT.md) for the actual workflows and observed test evidence.

## Architecture

```mermaid
flowchart TD
    U[Android app<br/>Kotlin / Jetpack Compose] -->|Firebase ID token + HTTPS| API[FastAPI on Render]
    F[Firebase Auth] -->|Verify identity| API
    API --> DB[(Neon PostgreSQL<br/>+ PostGIS)]
    API -->|FCM invitation / circle alert| N[Firebase Cloud Messaging]
    N --> R[Android recipients]
    U <-->|Authorized WebSocket events| API
    API -->|Presigned upload/download URL| S[(Private Cloudflare R2)]
    U -->|Upload files using presigned URL| S
    API -->|Optional error reporting| E[Sentry]
```

**Trust boundary:** Firebase authenticates the request; FastAPI enforces account, invitation, role, and incident-specific permissions; PostGIS identifies nearby eligible responders. WebSocket tickets are short-lived, and accepted participation is checked server-side. Files are stored privately rather than served publicly by the API.

The documented beta deployment uses **one Render API worker with in-process real-time fan-out**. Redis is supported by the code for multi-worker environments but is **not required in the documented free-tier deployment**. Read [PROJECT_HANDBOOK.md](PROJECT_HANDBOOK.md) and [DEPLOYMENT.md](DEPLOYMENT.md) for details.

## System flow

```mermaid
flowchart TD
    A[User presses and holds SOS for 3 seconds] --> B[FastAPI authenticates user and creates incident]
    B --> C[Phone starts sending fresh GPS updates]
    C --> D[PostGIS finds nearby eligible responders within 1 km]
    D --> E[API sends privacy-minimized invitations through FCM]
    E --> F[Responder accepts or declines]
    F --> G[If accepted, backend grants incident access]
    G --> H[While SOS is active, the victim location keeps updating]
    H --> I[Accepted responders receive authorized live updates]
    I --> J[Backend reevaluates nearby responders as the victim moves]
    J --> H
    H --> K[User closes SOS with Safety PIN]
    K --> L[Backend closes incident and revokes responder access]
```

**Other flows:** Home → **Circle** for trusted contacts; Home → **Share my trip** for consent-based sharing; Home → **Check in** for server-recorded check-ins; Home → **Responder** for volunteer availability and invitations; Home → **Profile** for permissions, theme, language, PIN, and profile management.

## Technology stack

| Layer | Technologies |
|---|---|
| **Android** | Kotlin, Jetpack Compose, Material 3, Android Navigation, Coroutines/Flow, WorkManager, DataStore |
| **Maps and location** | Google Play Services Fused Location Provider, foreground location service, **MapLibre Android (OpenGL)**, OpenFreeMap |
| **Authentication and messaging** | Firebase Authentication, Google Sign-In / Credential Manager, Firebase Cloud Messaging |
| **Networking** | Retrofit, OkHttp, WebSockets, HTTPS |
| **Backend** | Python, FastAPI, Uvicorn, Pydantic, SQLAlchemy AsyncIO, asyncpg |
| **Spatial database** | PostgreSQL, PostGIS, GeoAlchemy2, Alembic migrations |
| **Cloud infrastructure** | Render (API), Neon (database), private Cloudflare R2 (files) |
| **Monitoring and distribution** | Optional Sentry, Firebase Crashlytics, Firebase App Distribution; optional Redis for multi-worker fan-out |
| **Testing** | JUnit, Compose UI testing, Espresso, pytest, pytest-asyncio, httpx, isolated PostGIS tests |
| **Design** | [Maitri Figma](https://www.figma.com/design/Tsvh9SuF1INsZyS7EEnTFr/maitri?node-id=0-1), exported Light/Dark SVG boards |

## Limitations

> [!WARNING]
> Maitri is currently **beta**. Do not depend on it as your only emergency channel.

- **Availability is not guaranteed.** Render/Neon free-tier services can sleep, cold-start, experience quota limits, or fail. Connectivity loss can delay incident updates and alerts.
- **Android background limits apply.** Doze, battery optimization, force-stop, device shutdown, and manufacturer policies can interrupt tracking or notification behavior.
- **GPS and estimates can be inaccurate.** Indoor positioning, poor coverage, and routing providers may yield delayed or uncertain results.
- **Push delivery is not guaranteed.** FCM provider acceptance is not proof that a notification was displayed or read.
- **Volunteer coverage is not guaranteed.** Responses require appropriately configured, approved, available volunteers nearby; not every location has coverage.
- **External integrations require setup.** Private file uploads require R2; some Firebase phone-auth functions may require additional billing/provider setup. Test or beta reports may describe configurations not available to every developer.
- **No automatic official dispatch.** The app offers a user-controlled emergency dialer; it is **not** integrated with an official 112 dispatch backend or a public-safety answering point.
- **Documentation includes older phases.** Some historical files still say “SurakshaNet” or describe an older Render Postgres/Redis stack. For current behavior, prioritize the **actual code**, [PROJECT_HANDBOOK.md](PROJECT_HANDBOOK.md), [DEPLOYMENT.md](DEPLOYMENT.md), and the latest verified beta test evidence.

## Screenshot walkthrough

**View the full interactive design in Figma:**  
### [🎨 Open Maitri — 14 screens × Light and Dark](https://www.figma.com/design/Tsvh9SuF1INsZyS7EEnTFr/maitri?node-id=0-1)

The design contains **14 screen types with Light and Dark variations** (28 reference screens). The previews below are shown directly from the repository folder `design-reference/crops/`.

| Screen | Walkthrough | Light preview | Dark preview |
|---|---|---|---|
| 01 | Welcome and **Get started** | <img src="design-reference/crops/light-01.png" alt="Light Screen 01" width="140"> | <img src="design-reference/crops/dark-01.png" alt="Dark Screen 01" width="140"> |
| 02 | Home, SOS hold, trip sharing and check-in | <img src="design-reference/crops/light-02.png" alt="Light Screen 02" width="140"> | <img src="design-reference/crops/dark-02.png" alt="Dark Screen 02" width="140"> |
| 03 | Log in / Sign up | <img src="design-reference/crops/light-03.png" alt="Light Screen 03" width="140"> | <img src="design-reference/crops/dark-03.png" alt="Dark Screen 03" width="140"> |
| 04 | Volunteer application and verification | <img src="design-reference/crops/light-04.png" alt="Light Screen 04" width="140"> | <img src="design-reference/crops/dark-04.png" alt="Dark Screen 04" width="140"> |
| 05 | Trusted Circle management | <img src="design-reference/crops/light-05.png" alt="Light Screen 05" width="140"> | <img src="design-reference/crops/dark-05.png" alt="Dark Screen 05" width="140"> |
| 06 | Incoming SOS invitation | <img src="design-reference/crops/light-06.png" alt="Light Screen 06" width="140"> | <img src="design-reference/crops/dark-06.png" alt="Dark Screen 06" width="140"> |
| 07 | Responder journey / live guidance | <img src="design-reference/crops/light-07.png" alt="Light Screen 07" width="140"> | <img src="design-reference/crops/dark-07.png" alt="Dark Screen 07" width="140"> |
| 08 | Active SOS and Safety PIN closure | <img src="design-reference/crops/light-08.png" alt="Light Screen 08" width="140"> | <img src="design-reference/crops/dark-08.png" alt="Dark Screen 08" width="140"> |
| 09 | Profile and settings | <img src="design-reference/crops/light-09.png" alt="Light Screen 09" width="140"> | <img src="design-reference/crops/dark-09.png" alt="Dark Screen 09" width="140"> |
| 10 | Language settings | <img src="design-reference/crops/light-10.png" alt="Light Screen 10" width="140"> | <img src="design-reference/crops/dark-10.png" alt="Dark Screen 10" width="140"> |
| 11 | Test vibration | <img src="design-reference/crops/light-11.png" alt="Light Screen 11" width="140"> | <img src="design-reference/crops/dark-11.png" alt="Dark Screen 11" width="140"> |
| 12 | Permission settings | <img src="design-reference/crops/light-12.png" alt="Light Screen 12" width="140"> | <img src="design-reference/crops/dark-12.png" alt="Dark Screen 12" width="140"> |
| 13 | Safety PIN setup | <img src="design-reference/crops/light-13.png" alt="Light Screen 13" width="140"> | <img src="design-reference/crops/dark-13.png" alt="Dark Screen 13" width="140"> |
| 14 | Edit profile | <img src="design-reference/crops/light-14.png" alt="Light Screen 14" width="140"> | <img src="design-reference/crops/dark-14.png" alt="Dark Screen 14" width="140"> |

*These are **design references**, not proof of complete pixel-perfect physical-device rendering. Consult [MAITRI_ALL_SCREENS_VISUAL_REPORT.md](MAITRI_ALL_SCREENS_VISUAL_REPORT.md) for comparison evidence.*

## Run locally

### Prerequisites

- Android Studio with a compatible installed JDK / Gradle runtime and Android SDK (the app targets SDK 37; minimum SDK 26).
- Python **3.11–3.13** for backend development.
- PostgreSQL with **PostGIS**, or Docker Desktop for local database services.
- A Firebase project configured for the Android package **`in.maitri.app`**, with the required authentication providers.
- Separate test identities and a **test-only** backend environment.

### Clone

```bash
git clone https://github.com/ayankrmondal2003-rgb/Maitri-app.git
cd Maitri-app
```

### Android: configure and build

Open the project in Android Studio, sync Gradle, and ensure `app/google-services.json` corresponds to your **own** Firebase Android app. Keep machine-specific credentials in ignored `local.properties`.

```properties
# Add to local.properties; do not commit this file
maitriDevApiUrl=https://YOUR-DEV-API.example.com/
```

**Windows (PowerShell):**

```powershell
.\gradlew.bat :app:testDebugUnitTest
.\gradlew.bat :app:assembleDebug
.\gradlew.bat :app:installDebug
```

The debug APK is generated at `app/build/outputs/apk/debug/app-debug.apk`. Installation requires a compatible Android device or running emulator; the phone must be unlocked and ADB-authorized.

### Backend: configure and run

```powershell
cd backend
py -3.12 -m venv .venv
.\.venv\Scripts\python.exe -m pip install -r requirements.txt
Copy-Item .env.example .env
```

Edit `.env` with **local-only** values. Set a private local `POSTGRES_PASSWORD`, `DATABASE_URL`, `FIREBASE_PROJECT_ID`, and Firebase Admin credential path or application default credentials. Do **not** use the production Neon URL for local tests.

```powershell
# From backend/ after Docker Desktop is running
$env:POSTGRES_PASSWORD = "YOUR_PRIVATE_LOCAL_PASSWORD"
docker compose up -d db
.\.venv\Scripts\python.exe -m alembic upgrade head
.\.venv\Scripts\python.exe -m uvicorn app.main:create_app --factory --host 127.0.0.1 --port 8000 --reload
```

Follow [backend/README.md](backend/README.md) for correct local database URL, PostGIS setup, Firebase credentials, and optional Redis testing. `GET /health` and `GET /ready` are lightweight health checks; use server documentation for deeper database checks.

## Deployment

The repository includes a **Render Blueprint** at [`render.yaml`](render.yaml) and the detailed [DEPLOYMENT.md](DEPLOYMENT.md) runbook.

**Documented beta architecture:**

1. **Render** hosts the FastAPI Docker web service (currently configured with manual deployment).
2. **Neon PostgreSQL + PostGIS** provides persistent database and proximity queries.
3. **Cloudflare R2** stores private user documents and media using short-lived presigned URLs.
4. **Firebase** handles identity and FCM push notifications.
5. **Single-worker WebSocket delivery** uses the in-process event bus; Redis is optional for scaling.
6. **Firebase App Distribution** can distribute signed beta APKs to selected testers.

Deploy only after configuring environment variables and checking the current migration head. Keep Firebase Admin credentials, R2 secrets, PIN encryption keys, and database connection strings out of Git. Review provider pricing, free-tier limitations, and privacy obligations before enabling access for new users.

**Backend example:** [`https://surakshanet-api.onrender.com/ready`](https://surakshanet-api.onrender.com/ready) (a documented development/beta endpoint; a health response does **not** prove emergency readiness).

## Android development

- **Package / application ID:** `in.maitri.app`.
- **UI:** Jetpack Compose, with Maitri-specific screens under `app/src/main/java/in/maitri/app/ui/`.
- **Navigation:** `navigation/SafetyApp.kt`; bottom tabs are **Home → Circle → Responder → Profile**.
- **Design assets:** `design-reference/` and `app/src/main/res/drawable/`; original SVG boards remain authoritative visual references.
- **Localization:** `app/src/main/assets/i18n/` contains translated catalogs for Hindi, Bengali, Marathi, Tamil, and Telugu.
- **Location:** `location/` implements the GPS and tracking lifecycles; `network/` manages authenticated backend calls and WebSockets.
- **Beta builds:** app version, signing, build variants, and Firebase App Distribution are configured through `app/build.gradle.kts`.

Prefer a small, tested change per feature. Before changing incident behavior, check [SECURITY.md](SECURITY.md), [PRIVACY.md](PRIVACY.md), and the [existing tests](app/src/test/).

## Data preparation

Do **not** use real victims' location histories, identity documents, phone numbers, or medical notes for automated development tests.

- Create **synthetic Firebase test identities** with separate victim, trusted-contact, volunteer, and unauthorized-user roles.
- Use an **isolated PostgreSQL/PostGIS database** and apply Alembic migrations there.
- Generate **synthetic GPS fixes** with valid timestamps and accuracy values, including movement across the 1 km discovery boundary.
- Configure a private R2 test bucket for file workflows, with disposable, non-personal sample files.
- Configure required Android/Firebase credentials privately; never place tokens, service-account keys, database dumps, keystores, or secrets in commits.
- If verifying real FCM receipt, use only consenting test devices/accounts and record provider acceptance separately from device receipt.
- For safety-sensitive demos, review all recipient permissions before triggering an incident.

## Testing

| Layer | Command / approach | What it verifies |
|---|---|---|
| Android unit tests | `./gradlew :app:testDebugUnitTest` | State logic, validation, translations, feature regression |
| Android device tests | `./gradlew :app:connectedDebugAndroidTest` | Compose/UI behavior on an available emulator or device |
| Android debug build | `./gradlew :app:assembleDebug` | Kotlin/resources/Gradle compilation |
| Backend isolated tests | `cd backend && python tests/run_isolated.py` | Isolated PostGIS migrations and pytest API/security logic |
| Server integration | `backend/test-maitri-server.ps1` (Windows) | Server-based regression scenarios and integration evidence |
| Real beta field trials | [BETA_TEST_REPORT.md](BETA_TEST_REPORT.md) | Specific observed handset, invitation, delivery and location behavior |

**Do not copy test counts from historical phases as current results.** Run tests against your current commit and label each check **passed**, **failed**, **partial**, or **blocked**. Keep real-account and live-device tests separate from isolated tests.

Additional evidence: [MAITRI_SERVER_TEST_REPORT.md](MAITRI_SERVER_TEST_REPORT.md), [MAITRI_FINAL_READINESS.md](MAITRI_FINAL_READINESS.md), and [TEST_RESULTS.md](TEST_RESULTS.md) (the last file describes an earlier development phase).

## Controls and suggested walkthrough

For a **controlled, consenting test** with at least two separately authenticated phones:

1. Open Maitri and sign in with a verified account. Explore **Home**, **Circle**, **Responder**, and **Profile**.
2. Add a trusted circle member; verify contact ownership and notification preferences. Use pairing where required.
3. Visit **Profile** to configure the Safety PIN, notifications, permissions, theme and language.
4. On the responder device, complete the volunteer approval process, opt into availability and grant the required location permission.
5. On the requesting user's device, **hold SOS for three seconds**. Verify backend incident creation; do not assume the alert was delivered just because the button changed state.
6. Confirm the responder receives a genuine invitation, sees only approximate information before acceptance, and explicitly taps **Accept**.
7. Confirm the responder journey shows authorized location, and that location updates continue as the requesting user moves.
8. End the SOS using the correct Safety PIN. Check that responder access and live-location subscriptions are revoked.
9. Separately demonstrate **Share my trip**, **Check in**, and their recipient/inbox behavior without confusing them with SOS dispatch.
10. End by showing the **Call 112** option and explaining that Maitri cannot guarantee official emergency dispatch or rescue.

Avoid initiating a test SOS near or to unrelated people. The [three-phone guide](THREE_PHONE_DEMO.md) and [beta report](BETA_TEST_REPORT.md) document more detailed demonstrations and caveats.

## Repository structure

```text
Maitri-app/
├── app/                         # Native Android Kotlin + Compose application
│   └── src/main/java/in/maitri/app/
│       ├── ui/                  # Maitri screens, maps and reusable components
│       ├── navigation/          # App navigation
│       ├── data/                # Auth, contact and profile repositories
│       ├── location/            # Fused GPS and tracking service
│       ├── network/             # API clients, trip sharing, WebSocket and uploads
│       └── notification/        # FCM, responder request and alert handling
├── backend/                     # FastAPI application and isolated tests
│   ├── app/                     # Routes, services, authorization and storage
│   ├── alembic/                 # Database migrations
│   └── tests/                   # Backend regression suite
├── design-reference/            # Original Light/Dark boards and 14 screen crops
├── docs/                        # Diagrams, website and beta-test material
├── render.yaml                  # Render deployment blueprint
├── SETUP.md                     # Setup details
├── DEPLOYMENT.md                # Current deployment runbook
├── PROJECT_HANDBOOK.md          # Architecture and feature walkthrough
├── API_DOCUMENTATION.md         # API contracts
├── SECURITY.md                  # Security model
├── PRIVACY.md                   # Privacy and retention
└── README.md                    # This file
```

## Future work

- Independent security, accessibility, privacy, abuse-prevention, and reliability reviews before any broad public launch.
- Always-on hosting, database failover, monitored service-level objectives, load testing, and incident-response operations.
- Broader volunteer enrollment, identity-review governance and verified coverage across locations.
- Expanded multi-device field tests covering different Android/OEM battery policies, poor connectivity, Doze and provider outages.
- Improved notification observability, delivery diagnostics, localization quality and accessible large-text layouts.
- Further privacy controls for sharing, consent, media retention, and account data portability.
- Formal institutional integration if official dispatch services are ever pursued; do not imply such integration exists today.

## AI use disclosure

AI coding and writing assistants, including **ChatGPT and Codex**, have been used to assist with development planning, code generation/refactoring, debugging, testing guidance, design-to-code prompts and documentation. The repository also contains historical Claude-related development artifacts. **AI-generated material must be reviewed, tested and approved by human maintainers**; automated output is not a substitute for safety, privacy, security or field validation.

The team remains responsible for product decisions, source attribution, verification, deployment, and compliance. This disclosure does not imply that an AI system is a human team member or that any individual AI tool authored the entire application.

## Credits and license

**Built by Team 4 Loops.**  
- *Arpan Biswas*
- *Ayan Kumar Mondal*
- *Aritra Adak*
- *Aritra Pal*

**Maintainer:** [@ayankrmondal2003-rgb](https://github.com/ayankrmondal2003-rgb)  
**Design:** [maitri Figma design](https://www.figma.com/design/Tsvh9SuF1INsZyS7EEnTFr/maitri?node-id=0-1)  
**Core technologies:** Android/Jetpack Compose, Kotlin, Firebase, FastAPI, PostgreSQL/PostGIS, MapLibre, Render, Neon and Cloudflare R2.
Third-party dependencies and bundled font/asset licenses retain their respective terms (for example, the [Inter font license](app/src/main/assets/licenses/Inter-OFL.txt)).

**License status:** This repository does **not currently contain a top-level `LICENSE` file**. Public visibility does not automatically grant permission to copy, modify or redistribute the project. Team 4 Loops should choose and add an explicit license after confirming ownership and third-party obligations. Until then, no open-source license is asserted here.

---

<div align="center">

**maitri · Here for you. Safety, together.** 🩷  
[Explore the design in Figma](https://www.figma.com/design/Tsvh9SuF1INsZyS7EEnTFr/maitri?node-id=0-1) · [View the source](https://github.com/ayankrmondal2003-rgb/Maitri-app)

</div>
