# Rep Yourself — Project Context

Rep Yourself — Armstrong Pullup Program tracker. Three independent codebases (polyrepo, not a monorepo). Android is canonical for data and logic; web is legacy.

## Structure

- `android/` — Kotlin/Java, built in Android Studio, Room for local storage. **Canonical reference for both business logic and data schema.**
- `web/` — PWA, IndexedDB for local storage. **Legacy** — predates Android and is no longer the schema reference; kept only to support one-time data import into mobile apps.
- `ios/` — Native SwiftUI app, in progress. See `ios/CLAUDE.md` before working in this folder.

## Data contract

- **Android's schema is canonical going forward.** It's the more recently developed of the two and is the modern source of truth for data structure and business logic.
- **Web (PWA) data is legacy.** The web app predates Android by a few years and its schema is older and less refined.
- **Data flow is one-directional: web → mobile only, one-time.** A user can export their web data and import it into a mobile app as a one-time migration. There is no path, and no planned path, from a mobile app back to the web app.
- New platforms (iOS) build against Android's schema as the source of truth. The web/legacy schema is referenced only to support importing old user data in, never treated as an ongoing contract to stay compatible with.

## Cross-directory rule

Each platform folder is its own independent git repository. When working inside one platform's folder, treat the other two as read-only reference material — read them freely, don't edit them, unless explicitly asked to.

## Git workflow (applies in every repo — android/, web/, ios/)

- **Never commit directly to `main`.** Always create a branch first.
- **Branch names describe the work**, not a ticket number or generic label — e.g. `fix-streak-calculation`, `add-exercise-history-chart`, not `bugfix` or `patch-1`.
- **Commits are atomic.** Each commit is one logical change that could be reverted on its own — don't bundle an unrelated fix, a refactor, and a new feature into one commit.
