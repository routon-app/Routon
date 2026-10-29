# Routon Fleet - Flutter client

Flutter app for the **Routon Fleet Intelligence API** (`/v3/api-docs`).
White theme, fully responsive: phone, tablet, desktop, web (one codebase).
Login screen included (`POST /api/v1/auth/login`); token + companyId are saved automatically.

## Run

Requires Flutter **3.27+** (Dart 3.6+).

    chmod +x setup.sh && ./setup.sh      # macOS / Linux  (Windows: see below)
    flutter run -d chrome                # or: flutter run  (device / emulator)

`setup.sh` runs `flutter create` (generates android/ios/web/windows/macos/linux folders),
adds the Android INTERNET permission + cleartext HTTP for local servers, and runs `flutter pub get`.

Windows manual steps:

    flutter create --platforms=android,ios,web,windows --project-name routon_fleet .
    del test\widget_test.dart
    flutter pub get

then add `<uses-permission android:name="android.permission.INTERNET"/>` to
`android/app/src/main/AndroidManifest.xml` (before `<application`) and
`android:usesCleartextTraffic="true"` on the `<application>` tag if your server is plain http.
macOS: add `com.apple.security.network.client` = true to both `macos/Runner/*.entitlements` files.

## Backend

Default server is already set to:
`http://k8s-routonte-routonal-8e990dc6a0-86763539.ap-south-2.elb.amazonaws.com`

Override at build time: `flutter run -d chrome --dart-define=API_URL=http://other-host`

## First start

Open **Settings** in the app and set:
- Server URL (Android emulator -> `http://10.0.2.2:8080`, real phone -> your PC's LAN IP)
- Company ID (scopes list/dashboard calls)
- JWT token (only if your server requires auth)

## API Explorer (matches Swagger UI)

System > **API Explorer** loads `/v3/api-docs` live from your server and lists every endpoint grouped by controller/tag (same as Swagger UI), with parameter fields, JSON body, Execute and response view. Any controller not yet built as a screen (e.g. alert-config) is usable here immediately.

## Structure

    lib/core/        config (settings), api_client (Dio + Bearer token), theme (white), utils
    lib/services/    ModuleService (generic CRUD + actions), ViewerService, DashboardService
    lib/data/        modules.g.dart  <- GENERATED from the OpenAPI spec (35 modules, 19 report views)
    lib/ui/          shell (responsive sidebar/drawer), pages, dynamic forms, widgets

## Features (all API driven)

Dashboard (vehicle status, alerts, trips, expiring documents, activity) - Live map (auto refresh) -
Fleet: vehicles (approve/reject), drivers, trips (start/complete/cancel/assign), routes, zones, geofences, vehicle groups, schedules -
Devices: GPS devices, SIM cards, fuel sensors (full lifecycle actions) -
Inventory: warehouses, vendors, categories, units, items, stock, purchase orders, job cards, goods receipts, transfers, movements, assignments -
Service schedules/entries, tires - Alerts (acknowledge/dismiss), notifications, notification rules -
Companies, users, roles, states, districts, audit logs - Analytics & reports viewers.

## Adding login later

`AppSettings.token` is the only auth input. Build a LoginPage that calls `POST /api/v1/auth/login`
(`AuthRequest` -> `AuthResponse.token`), then `settings.save(token: ..., companyId: ...)`, and show it
from `RoutonApp.home` when the token is empty. Nothing else changes.

## Notes

- Some collection paths (`/drivers`, `/zones`) were missing from the exported API doc text and are inferred by REST convention - fix `path` in `modules.g.dart` if your server differs.
- Foreign-key fields (vehicleId, driverId, ...) are plain number inputs for now; pickers are a good next step.
