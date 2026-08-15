# Web — Cross-Platform Gap Notes

Gaps, inconsistencies, or issues found in `web/` while working in a different platform folder. Logged here rather than edited directly — see the root `CLAUDE.md`'s Cross-Platform Gap Notes section.

---

### [Resolved] Add a `/support` page — needed as iOS's (and eventually Android's) Support URL
Found while: closing out iOS's App Store Connect listing checklist (2026-08-11/12, see `ios/AppStoreListing.md` and `ios/CLAUDE.md` §9).
Details: iOS's App Store listing needs a Support URL and settled on `https://repyourself.app/support`, matching how the Privacy Policy URL already lives on this same domain (Android's `DashboardScreen.kt` links to `https://repyourself.app/privacy`; see the parallel entry in `gaps/android.md` — Android has no Support URL yet either, and should point at this same page once it exists). The page doesn't exist yet — this repo is where it needs to be built. iOS's checklist can't be marked ready to submit until it's live.

Suggested approach, from investigating this repo's structure during that session (Next.js App Router, deployed to Vercel):
- New route mirroring the existing `/privacy` page: `src/app/(privacy)/privacy/page.tsx` + `src/components/privacy/PrivacyPolicy.tsx`/`.module.css` is the pattern to follow — reuse the `(privacy)` route group (its `layout.tsx` is generic, not privacy-specific) rather than inventing a new layout.
- Add a `/support` entry to `src/app/sitemap.ts` alongside the existing `/privacy` one.
- `src/components/landing/Footer.tsx`'s `SUPPORT` link currently points straight at `mailto:support@repyourself.app` with no landing page behind it — once `/support` exists, point that link at the new page instead (same pattern as `PRIVACY POLICY` linking to `/privacy` rather than mailing directly), keeping the mailto as a contact link *on* the new page.

Resolved: 2026-08-13, `web` branch `add-support-page` (merged as PR #138) — built `src/app/(privacy)/support/` (`page.tsx`/`layout.tsx`/`default.tsx`/`layout.module.css`, mirroring `/privacy`) + `src/components/support/Support.tsx`/`.module.css`, added the `/support` entry to `sitemap.ts`, and repointed `Footer.tsx`'s SUPPORT link at the new page (mailto stays as a contact link on the page itself, per the suggested approach). Also fixed a pre-existing, unrelated rendering bug on `/privacy` found in the same session (literal `&apos;`/`&quot;` text visible on the page — see `PrivacyPolicy.tsx`'s git history for `a06ad05`).

Note: `support@repyourself.app` (referenced on the new page, in `Footer.tsx`, and elsewhere) does not have forwarding set up yet as of 2026-08-14 — Gerald flagged this but asked to leave it as-is in code for now (known TODO on his end, not a code problem).

### [Open] Empty dashboard in Chrome on a physical iPhone — no Today's Workout / Skip / Day 1
Found while: live-testing the iOS PWA download/Web Share fix (2026-08-14, branch `support-ios-pwa-download`, commit `a90f12b`) on a real iPhone.
Details: Gerald reported that opening the app in Chrome on a physical iPhone and tapping "Get Started" lands on an empty dashboard — no "Today's Workout" heading, no Skip button, no Day 1 link. Not diagnosed before the session ended (interrupted mid-investigation), but it looks related to an *already-known* issue: `tests/programPageDbFallback.spec.ts`'s docstring documents a prior "iOS PWA cold-launch bug" where `indexedDB.open()` never calls back (a documented WKWebView quirk on a fresh home-screen launch), which used to leave `programDayNumber` stuck at 0 forever with nothing to tap — fixed with a 3-second fallback timer in `Program.tsx` (see that test for the exact repro/fix shape). Worth checking first whether this is the same bug recurring in a new context (a regular Chrome tab, not a home-screen-installed PWA — the existing test/fix's docstring specifically frames it as a home-screen-launch scenario) or a distinct variant the fallback doesn't cover. Note: since Chrome-iOS runs on WebKit like every other iOS browser (see `ios/CLAUDE.md` and this session's Web Share research), this is plausibly not Chrome-specific and could reproduce in Safari too — worth checking there as well, not just Chrome.
