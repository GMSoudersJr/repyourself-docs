# iOS — Cross-Platform Gap Notes

Gaps, inconsistencies, or issues found in `ios/` while working in a different platform folder. Logged here rather than edited directly — see the root `CLAUDE.md`'s Cross-Platform Gap Notes section.

---

### [Open] Onboarding import instructions name the wrong export filename
Found while: investigating whether web's downloadable workout-history JSON would be readable by iOS's import function (2026-08-14 session, no branch — pure research, no code changes made).
Details: iOS's onboarding `ImportInstructionsSheet` (`ios/RepYourself/RepYourself/Views/OnboardingView.swift:132-176`, copy sourced from `Localizable.xcstrings`) tells users looking to import legacy data to look for a file named `RepYourself_Export.json`. But web's actual "Download Workout History" button (`web/src/components/program/DownloadDataButton.tsx:28`) names the downloaded file `Pull-Up-App-Backup.json` — a different name entirely. This doesn't break the import itself (`LegacyPwaMigrator` validates JSON content, not filename), but a user following the on-screen instructions literally would be looking for a file that doesn't exist, which is confusing. Fix is presumably on the iOS side: update the instructions sheet's copy to reference `Pull-Up-App-Backup.json` instead of `RepYourself_Export.json` (or, if the filename is considered part of iOS's own product copy, coordinate with web instead — but web's filename is already shipped/stable, so updating iOS's copy is the lower-risk fix).
