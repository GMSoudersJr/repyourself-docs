# Rep Yourself — Project Context

Rep Yourself — Armstrong Pullup Program tracker. Three independent codebases (polyrepo, not a monorepo). Android is canonical for data and logic; web is legacy. **iOS is the current active development focus** — android/ and web/ are in maintenance mode, receiving fixes as gaps surface from iOS work (see Cross-Platform Gap Notes below).

## Structure

- `android/` — Kotlin/Java, built in Android Studio, Room for local storage. **Canonical reference for both business logic and data schema.** Currently maintenance-only.
- `web/` — PWA, IndexedDB for local storage. **Legacy** — predates Android and is no longer the schema reference; kept only to support one-time data import into mobile apps. Currently maintenance-only.
- `ios/` — Native SwiftUI app, **current active focus**. See `ios/CLAUDE.md` before working in this folder.

## Data contract

- **Android's schema is canonical going forward.** It's the more recently developed of the two and is the modern source of truth for data structure and business logic.
- **Web (PWA) data is legacy.** The web app predates Android by a few years and its schema is older and less refined.
- **Data flow is one-directional: web → mobile only, one-time.** A user can export their web data and import it into a mobile app as a one-time migration. There is no path, and no planned path, from a mobile app back to the web app.
- New platforms (iOS) build against Android's schema as the source of truth. The web/legacy schema is referenced only to support importing old user data in, never treated as an ongoing contract to stay compatible with.

## Cross-directory rule

Each platform folder is its own independent git repository. When working inside one platform's folder, treat the other two as read-only reference material — read them freely, don't edit them directly. If you notice something worth fixing there, log it instead — see Cross-Platform Gap Notes below.

## Cross-Platform Gap Notes

This directory is a waypoint between platform sessions, not just a docs archive. If you're working in one platform folder and notice a gap, inconsistency, or issue in a *different* platform's folder, don't switch over and fix it there — log it here so a future session actually working in that folder picks it up.

- `gaps/android.md` — gaps found in Android, logged while working elsewhere
- `gaps/web.md` — gaps found in web, logged while working elsewhere
- `gaps/ios.md` — gaps found in iOS, logged while working elsewhere

**Starting work in a platform folder?** Check that folder's gaps file first — treat open entries as known context, not a surprise you're discovering fresh.

**Entry format:**
```
### [Open] <short title>
Found while: <what task/context surfaced this>
Details: <description of the gap>
```
Mark `[Resolved]` rather than deleting once addressed, so there's a record of what got fixed and when.

## Git workflow (applies in every repo — android/, web/, ios/)

- **Never commit directly to `main`.** Always create a branch first.
- **Branch names describe the work**, not a ticket number or generic label — e.g. `fix-streak-calculation`, `add-exercise-history-chart`, not `bugfix` or `patch-1`.
- **Commits are atomic.** Each commit is one logical change that could be reverted on its own — don't bundle an unrelated fix, a refactor, and a new feature into one commit.
