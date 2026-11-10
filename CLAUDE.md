# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

"Aoun" is a Flutter civic incident-reporting app for Jordan. Citizens submit reports (form, image, video, voice, or SOS emergency) that get routed to government departments (GAM, Miyahuna, JEPCO, PSD, etc. — see `lib/department_utils.dart`); admins review and process them.

## Commands

```
flutter pub get                       # install dependencies
flutter run                           # run the app
flutter analyze                       # lint / static analysis
flutter test                          # run all tests
flutter test test/widget_test.dart    # run a single test file
dart run intl_utils:generate          # regenerate l10n after editing .arb files
```

## Architecture

### Roles and auth routing
- Firebase Auth + Firestore. Two roles: regular users live in the `users` collection (doc ID = Firebase Auth uid), admins live in the `admin` collection.
- `AuthGate` in `lib/main.dart` decides the landing page: if `users/{uid}` exists → `HomePage`, otherwise → `AdminPage`. It checks the Firestore cache first, then the server.

### Firestore collections
- `users` — regular user profiles (doc ID = auth uid)
- `admin` — admin accounts, keyed/filtered by department
- `reports` — all report types; admins query by department
- `notifications` — per-user status-change notifications, created by admins in `AdminFormPage`

### Code layout
- Legacy/main code: flat PascalCase page files directly in `lib/` (e.g., `HomePage.dart`, `ReportForm.dart`, `AdminPage.dart`). Most use plain `StatefulWidget` + direct Firestore calls.
- Newer feature code: `lib/features/<feature>/{data,presentation}` structure using Riverpod (`ProviderScope` wraps the app). Currently only `voice_agent`. Prefer this structure for new features.

### Localization
- flutter_intl / intl_utils. Source strings in `lib/l10n/intl_ar.arb` and `intl_en.arb`; generated code in `lib/generated/`. Access via `S.of(context).key`.
- Default locale is Arabic (`ar`). Any user-visible string must be added to **both** .arb files, then regenerate.
- Locale and theme are toggled globally via `MyApp.of(context)?.setLocale(...)` / `.toggleTheme()` (ancestor-state lookup pattern in `main.dart`).

### Secrets
- `lib/app_secrets.dart` is gitignored; copy `lib/app_secrets.template.dart` and fill in real values (Groq API key, summarize endpoint). Never commit real keys.

### External services
- Firebase: Auth, Firestore, Storage (project `auonproject2`)
- Groq chat-completions API (key in app_secrets)
- Video summarizer endpoint on Hugging Face Spaces
- Voice agent (`lib/features/voice_agent/`) records audio, transcribes/chats/synthesizes speech, and files reports into the `reports` collection

### UI conventions
- Page transitions are disabled globally (`_NoTransitionsBuilder` in `main.dart`).
- Dark mode is supported via `darkTheme` in `main.dart`; use `Theme.of(context)` colors rather than hardcoding so both modes work.
- Shared bottom navigation lives in `lib/shared_bottom_nav.dart`.
