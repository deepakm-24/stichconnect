# StitchConnect — Android App

A tailor-booking marketplace app (customers, tailors, and delivery partners)
built with Kotlin + Jetpack Compose + Room, following the module spec:
search/dashboard, customer profile & measurements, tailor module, delivery
module, in-app messaging, and help & support.

## What changed in this pass

**Fixed a build-breaking bug.** `app/build.gradle.kts` pointed the debug
build at `debug.keystore`, a file that didn't exist in the project — so a
fresh checkout failed to build until someone deleted that line by hand (the
old README even said to do this manually). A real debug keystore is now
committed at the project root so `./gradlew assembleDebug` / the Android
Studio Run button works immediately, no manual edits required.

**Fixed a real efficiency bug.** `StitchViewModel.selectTailor()` and
`selectOrder()` used to launch a brand-new, never-cancelled
`Flow.collect { }` coroutine every time you viewed a tailor or order — so
browsing multiple tailors left old collectors running for the life of the
app. Rewritten using `flatMapLatest` + `stateIn`, which cancels the previous
collection automatically when the selected ID changes.

**Added database indices** on every foreign-key-style column (`orders.customerId`,
`orders.tailorId`, `orders.deliveryPartnerId`, `reviews.tailorId`,
`chat_messages.conversationId`, etc.) so lookups don't full-scan tables as
data grows.

**Built out the two modules that were previously just placeholder text:**
- **Messaging** (spec section 5): real conversation threads backed by a
  `chat_messages` table — `ui/messaging/MessagingScreens.kt` (conversation
  list + chat detail), wired to the customer Profile screen and to
  `Screen.Messages` / `Screen.ChatDetail` in navigation.
- **Support** (spec section 6): `ui/support/SupportScreens.kt` — a real FAQ
  accordion, a "Report an issue / Feedback" form backed by a
  `support_tickets` table, and a ticket-history tab — replacing the old
  static help text.

**Extended measurement profiles** (spec section 2) to be garment-aware:
`CreateMeasurementScreen` now has a Shirt / Pant / Blouse selector and shows
the fields that actually apply — chest, shoulder, sleeve length, sleeve
opening, neck round, and shirt length for shirts; waist, hip, inseam, thigh,
bottom opening, and pant length for pants; bust, waist, shoulder, sleeve,
armhole, and blouse length for blouses — instead of one generic field set
for every garment.

## Getting an installed app on your phone

I can't compile an APK from this chat — there's no Android SDK or network
access in this environment to fetch Gradle/dependencies. Pick one of these:

### Option A — Android Studio (easiest, no setup beyond the IDE)
1. Install [Android Studio](https://developer.android.com/studio) (free).
2. **Open** this project folder.
3. Let it sync (downloads Gradle + dependencies automatically).
4. Plug in your phone (USB debugging on) or use the emulator, hit **Run ▶**.
5. To get a standalone APK file to send/install anywhere: **Build > Build App
   Bundle(s) / APK(s) > Build APK(s)**. It lands in
   `app/build/outputs/apk/debug/app-debug.apk` — copy that to your phone and
   open it to install (you'll need to allow "install unknown apps" once).

### Option B — GitHub Actions (get an APK without installing anything)
This project now includes `.github/workflows/build-apk.yml`. Push this repo
to a GitHub repository, then:
1. Go to the **Actions** tab → the workflow run.
2. Download the `stitchconnect-debug-apk` artifact.
3. Unzip it, transfer `app-debug.apk` to your phone, and install it.

### Option C — command line, if you have the Android SDK installed locally
```bash
# from the project root
gradle wrapper --gradle-version 8.11   # generates gradlew (not included)
./gradlew assembleDebug
# APK at app/build/outputs/apk/debug/app-debug.apk
```

## Notes
- `.env` / `GEMINI_API_KEY`: nothing in the current codebase actually reads
  this, so it's optional unless you're adding a Gemini-powered feature
  yourself.
- Release signing (`signingConfigs.getByName("release")`) still requires
  `KEYSTORE_PATH`, `STORE_PASSWORD`, and `KEY_PASSWORD` env vars plus a real
  upload key — that's intentionally left as-is; a production signing key
  shouldn't be generated for you sight-unseen.
- This is still backed by a local Room database with seeded demo data
  (no real backend/auth) — fine for demoing the full flow end-to-end, but
  before a real launch you'd want a backend API and real authentication.
