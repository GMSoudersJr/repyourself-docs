# iOS — Cross-Platform Gap Notes

Gaps, inconsistencies, or issues found in `ios/` while working in a different platform folder. Logged here rather than edited directly — see the root `CLAUDE.md`'s Cross-Platform Gap Notes section.

---

### [Resolved] Onboarding import instructions name the wrong export filename
Found while: investigating whether web's downloadable workout-history JSON would be readable by iOS's import function (2026-08-14 session, no branch — pure research, no code changes made).
Details: iOS's onboarding `ImportInstructionsSheet` (`ios/RepYourself/RepYourself/Views/OnboardingView.swift:132-176`, copy sourced from `Localizable.xcstrings`) tells users looking to import legacy data to look for a file named `RepYourself_Export.json`. But web's actual "Download Workout History" button (`web/src/components/program/DownloadDataButton.tsx:28`) names the downloaded file `Pull-Up-App-Backup.json` — a different name entirely. This doesn't break the import itself (`LegacyPwaMigrator` validates JSON content, not filename), but a user following the on-screen instructions literally would be looking for a file that doesn't exist, which is confusing. Fix is presumably on the iOS side: update the instructions sheet's copy to reference `Pull-Up-App-Backup.json` instead of `RepYourself_Export.json` (or, if the filename is considered part of iOS's own product copy, coordinate with web instead — but web's filename is already shipped/stable, so updating iOS's copy is the lower-risk fix).

Resolved: 2026-08-16, `ios` branch `fix-import-export-filename-mismatch`. Confirmed Android's own export (`android/.../SettingsScreen.kt:59`) also uses `RepYourself_Export.json`, matching iOS's — so the single-filename card wasn't simply wrong, it was incomplete: two distinct valid filenames exist depending on source (native iOS/Android vs. web). Rather than swap one literal for another, `ImportInstructionsSheet` now shows both filenames as separate cards, each with a caption naming its source ("From Rep Yourself on iOS or Android" / "From the web app"). New captions added to `Localizable.xcstrings` per the project's no-hardcoded-strings convention. Verified visually in Simulator (iPhone 17, fresh onboarding state).

### [Resolved] `ModelContainer` init failure hard-crashes via `fatalError` — no fallback UI, closest iOS analog to a bug just fixed on `web`
Found while: cross-platform check for the same bug class as a `web` fix (2026-08-15), where an unguarded IndexedDB init could throw synchronously and bypass all UI fallback handling, blanking the dashboard before the fallback-carrying component ever mounted. `web`'s fix also involved deduplicating a per-call-connection pattern (opening/closing a new connection per read) into a single long-lived shared connection, after an initial attempt at the first fix introduced a race on a shared connection variable.
Details: `RepYourselfApp.swift:42-54` creates the app's single `ModelContainer` once, as a stored property computed via an immediately-invoked closure on the `@main` App struct:
```swift
var sharedModelContainer: ModelContainer = {
    let schema = Schema([WorkoutHistory.self, UserProfile.self])
    let modelConfiguration = ModelConfiguration(schema: schema, isStoredInMemoryOnly: false)
    do {
        return try ModelContainer(for: schema, configurations: [modelConfiguration])
    } catch {
        fatalError("Could not create ModelContainer: \(error)")
    }
}()
```
This part is done *correctly* relative to the `web` bug's race — it's injected once via `.modelContainer(sharedModelContainer)` and consumed uniformly through the environment-injected `modelContext` everywhere (`DashboardView`, `SettingsView`, `OnboardingView`, `WorkoutDayView`); no ad-hoc `ModelContext(container)` construction or shared-mutable-reference race exists anywhere in production code (only `#Preview` blocks construct their own isolated in-memory containers, which is normal and not a production path).

The gap is the `catch` block: `fatalError` runs synchronously during `App` struct init, *before* `WindowGroup`/`RootView` ever builds a view — if `ModelContainer(for:configurations:)` throws (corrupted store file, disk full, or a future migration failure), the process terminates immediately with no recovery path, no retry UI, and nothing surfaced to the user beyond an OS-level crash. This is the same *class* of bug as the `web` fix (synchronous throw during store-init, at a point outside any UI-level error/fallback handling, before the first screen ever mounts) but arguably worse in effect: not a silently blank screen a user might just reload, but an outright unrecoverable crash. Compounding this: no `SchemaMigrationPlan`/`VersionedSchema` exists anywhere (confirmed via grep — zero hits), so this is a single unversioned `Schema`; normal for a pre-1.0 app with no shipped schema history yet, but it means any *future* migration failure would also route through this same unconditional `fatalError`, not a graceful fallback.

Resolved: 2026-08-16, `ios` branch `fix-modelcontainer-init-crash`. Checked Apple's own `ModelContainer`/`ModelConfiguration` docs first (per `ios/CLAUDE.md`'s "check SwiftUI docs before changes" note) — no blessed recovery API beyond try/catch, so the fix follows the same fallback pattern the `web` fix used: `RepYourselfApp` now tries the persistent store first and, only on failure, falls back to an `isStoredInMemoryOnly: true` container so the app can still launch (a genuinely-broken in-memory container, which would indicate a real schema/programmer bug rather than an environment issue, is still a `fatalError` — retrying wouldn't help that case). A new `isPersistentStoreDegraded` `EnvironmentValues` key carries the fallback state down to `RootView`, which now shows a persistent orange top banner ("Couldn't open your saved data. Changes made now won't be saved once you close the app.") rather than staying silent about it. Verified both paths in Simulator (build + full test suite green in both states): normal launch shows no banner; a temporarily-forced-true build shows the banner correctly with no crash, then the hack was reverted before committing.
