# Android — Cross-Platform Gap Notes

Gaps, inconsistencies, or issues found in `android/` while working in a different platform folder. Logged here rather than edited directly — see the root `CLAUDE.md`'s Cross-Platform Gap Notes section.

---

### [Resolved] Analytics opt-out doesn't apply until the toggle fires again, not on cold start
Found while: porting Firebase Analytics from Android to iOS (`ios/CLAUDE.md` §9, `add-firebase-analytics`, PR #30).
Details: Android only calls `setAnalyticsCollectionEnabled` from the Settings toggle's own `onCheckedChange` handler. If a user disables "Share anonymous usage data" and then force-quits/relaunches the app, the next cold start silently reverts to Firebase's default-enabled state — no code path re-applies the stored preference at launch. iOS fixed this by re-applying the persisted opt-out in `AppDelegate.application(_:didFinishLaunchingWithOptions:)` before any event can fire; Android has no equivalent fix yet. Worth confirming still reproducible and porting the same fix (re-apply the stored preference at app-start, not just from the toggle handler).
Resolved: 2026-08-12, `android` branch `fix-analytics-optout-cold-start` (merged) — `RepYourselfApplication.onCreate()` now reads the persisted `SettingsRepository.isAnalyticsEnabled` value and re-applies it via `AnalyticsHelper.setAnalyticsCollectionEnabled` before any event can fire, matching iOS's `AppDelegate` fix. Verified via Logcat on a real cold start (force-stop + relaunch).

### [Open] `onboarding_start_fresh` analytics event may log twice
Found while: same Firebase Analytics port as above.
Details: iOS's port doc notes iOS deliberately logs `onboarding_start_fresh` once, calling out that this diverges from "Android's apparent double-log." This was an observation made while porting the taxonomy, not a confirmed/reproduced bug — worth verifying against Android's actual onboarding flow before treating as real, but flagging since a duplicate analytics event would skew usage metrics.

### [Resolved] `program_reset_failed` sends a raw, unsanitized exception message to analytics
Found while: same Firebase Analytics port as above.
Details: Android's `program_reset_failed` event's `error_message` param sends the raw exception message text. iOS deliberately uses a coarse error type name instead (`String(describing: type(of: error))`) rather than the raw message. Both apps' privacy policies (`repyourself.app/privacy`) promise only anonymous, non-personal diagnostic data — a raw exception message is more likely than a type name to incidentally include something that isn't purely anonymous (e.g. a file path or other contextual string). Worth reviewing whether Android's raw message ever contains anything beyond a generic error description, and whether to match iOS's coarser approach.
Resolved: 2026-08-12, `android` branch `fix-program-reset-failed-analytics-privacy` — `SettingsViewModel.kt` now sends `e::class.java.simpleName` instead of `e.message`, matching iOS's coarse-type approach.

### [Open] No Play Store Support URL exists yet
Found while: building out iOS's App Store Connect listing checklist this session (2026-08-11/12, see `ios/AppStoreListing.md` and `ios/CLAUDE.md` §9).
Details: A repo-wide grep during that work found Android only references `https://repyourself.app/privacy` (in `DashboardScreen.kt`) — no Support URL/email reference anywhere in `android/`. iOS settled on `https://repyourself.app/support` for its own Support URL (page not built yet — see `gaps/web.md`), matching the shared-domain pattern already used for the privacy policy. Android's Play Store listing will need an equivalent Support URL eventually; once the web `/support` page exists, Android should link to the same URL rather than inventing a separate one.
