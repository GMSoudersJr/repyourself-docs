# Web — Cross-Platform Gap Notes

Gaps, inconsistencies, or issues found in `web/` while working in a different platform folder. Logged here rather than edited directly — see the root `CLAUDE.md`'s Cross-Platform Gap Notes section.

---

### [Open] Add a `/support` page — needed as iOS's (and eventually Android's) Support URL
Found while: closing out iOS's App Store Connect listing checklist (2026-08-11/12, see `ios/AppStoreListing.md` and `ios/CLAUDE.md` §9).
Details: iOS's App Store listing needs a Support URL and settled on `https://repyourself.app/support`, matching how the Privacy Policy URL already lives on this same domain (Android's `DashboardScreen.kt` links to `https://repyourself.app/privacy`; see the parallel entry in `gaps/android.md` — Android has no Support URL yet either, and should point at this same page once it exists). The page doesn't exist yet — this repo is where it needs to be built. iOS's checklist can't be marked ready to submit until it's live.

Suggested approach, from investigating this repo's structure during that session (Next.js App Router, deployed to Vercel):
- New route mirroring the existing `/privacy` page: `src/app/(privacy)/privacy/page.tsx` + `src/components/privacy/PrivacyPolicy.tsx`/`.module.css` is the pattern to follow — reuse the `(privacy)` route group (its `layout.tsx` is generic, not privacy-specific) rather than inventing a new layout.
- Add a `/support` entry to `src/app/sitemap.ts` alongside the existing `/privacy` one.
- `src/components/landing/Footer.tsx`'s `SUPPORT` link currently points straight at `mailto:support@repyourself.app` with no landing page behind it — once `/support` exists, point that link at the new page instead (same pattern as `PRIVACY POLICY` linking to `/privacy` rather than mailing directly), keeping the mailto as a contact link *on* the new page.
