# TaskAces — Best Practices

**App Name:** TaskAces  
**Version:** 1.0.5  
**Platform:** iOS & Android (React Native / Expo)  
**Date:** April 2026  

---

## 1. Code Updates and Maintenance

### 1.1 Follow a Consistent Branching Strategy

Use **Git Flow** or a simplified variant to keep the repository clean and release-ready at all times.

| Branch | Purpose |
|--------|---------|
| `main` | Stable, production-ready code only |
| `develop` | Integration branch for completed features |
| `feature/<name>` | Individual feature development |
| `fix/<name>` | Bug fixes |
| `release/<version>` | Pre-release stabilization (e.g., `release/1.1.0`) |

Never commit directly to `main`. All changes must arrive via a Pull Request with at least one reviewer approval before merging. The current naming convention `TaskAces_1.0.0` should be reserved for release branches — not general development work.

### 1.2 Write Meaningful Commit Messages

Follow the **Conventional Commits** specification for consistency and to enable automatic changelog generation.

```
<type>(<scope>): <short summary>

Examples:
feat(lists): add swipe-to-delete on task rows
fix(notifications): reschedule notification when due date changes
chore(deps): upgrade expo-notifications to 0.32.0
refactor(theme): extract color tokens to ThemeContext
docs(readme): add environment variable setup instructions
```

Valid types: `feat`, `fix`, `docs`, `style`, `refactor`, `test`, `chore`.

### 1.3 Keep Components Focused and Small

- Each screen file should own its UI and local state only. Business logic — data reads and writes, notification scheduling, auth calls — belongs in dedicated modules under `lib/`.
- [app/(tabs)/lists.tsx](../app/(tabs)/lists.tsx) is currently ~900 lines. Extract sub-components such as `TaskRow`, `PrioritySection`, and `AddTaskModal` into a `components/` directory to improve readability and allow independent testing.
- Aim for screen files under 300 lines and utility functions that do one thing each.

### 1.4 Enforce TypeScript Strict Mode

The project has `"strict": true` in [tsconfig.json](../tsconfig.json). Maintain this setting at all times.

- Do not suppress TypeScript errors with `// @ts-ignore` or untyped `any` casts.
- Write explicit type definitions for all new interfaces, especially the `Task` object and any Supabase response shapes.
- Run `npm run typecheck` before every Pull Request and treat type errors as build failures.

### 1.5 Environment Variables

- All secrets and environment-specific values live in `.env` only — never hard-coded in source files.
- `.env` must remain in `.gitignore` and is never committed to the repository under any circumstances.
- All client-side env vars must be prefixed with `EXPO_PUBLIC_` so Expo bundles them correctly into the app.
- Maintain a `.env.example` file in the repository documenting every required key with placeholder values so new developers know what to configure without needing to ask.

### 1.6 Code Reviews

Every Pull Request into `develop` or `main` requires review before merge. Reviewers should check:

- No hard-coded secrets, URLs, or credentials
- TypeScript errors are resolved — `npm run typecheck` passes
- No unused imports or dead code
- Feature has been tested on both iOS and Android
- AsyncStorage reads and writes are not blocking the UI thread unnecessarily

Keep PRs small and focused — one feature or fix per PR makes review faster and rollbacks surgical.

### 1.7 Testing

- Write **unit tests** for pure functions in `lib/` — especially the NLP parser (`lib/nlp.ts`) and date helpers.
- Use **Jest** with `@testing-library/react-native` for component-level tests.
- Before any production release, manually test the complete golden path on a **physical device** (not just a simulator): create a task, set a due date, receive a notification, attach a photo, toggle dark mode, sign in and sign out with Google.
- Test on both iOS and Android before every release — gesture behavior and notification rendering differ between platforms.

### 1.8 Linting and Formatting

```bash
npm run lint        # ESLint — catches code style and common bugs
npm run typecheck   # TypeScript — catches type errors
```

Add a pre-commit hook using `husky` and `lint-staged` so these checks run automatically before every commit, making it impossible to commit broken code:

```bash
npm install --save-dev husky lint-staged
npx husky init
```

---

## 2. APIs, SDKs, and Libraries — Updates and Maintenance

### 2.1 Audit Dependencies Regularly

Run a security audit at least once per month:

```bash
npm audit                   # list known vulnerabilities
npm audit fix               # auto-fix safe patches
npm outdated                # show packages with newer versions available
```

Respond to audit results by severity:

| Severity | Response time |
|----------|--------------|
| Critical | Immediately — same day |
| High | Within the current sprint |
| Medium | Within the next sprint |
| Low | At next scheduled dependency update |

### 2.2 Dependency Update Order

Do not update all packages at once. Follow this priority order to minimize risk:

1. **Security patches** — update immediately regardless of other work in progress.
2. **Expo SDK** — Expo ships a new SDK approximately every quarter. Upgrade within 6 weeks of each release. Always follow the official [Expo SDK upgrade guide](https://docs.expo.dev/workflow/upgrading-expo-sdk-walkthrough/) which lists every required companion version bump and breaking change. Note: the current SDK 54 was chosen specifically because SDK 55 introduced a worklets incompatibility with React Native Reanimated — verify this is resolved before upgrading.
3. **React Native** — the React Native version is tied directly to the Expo SDK version. Upgrade them together, not independently.
4. **Supabase JS** — review the Supabase JS changelog for breaking changes to auth flows or query APIs before upgrading across major versions.
5. **Other libraries** — batch minor and patch updates into a monthly maintenance PR.

### 2.3 Lock File Discipline

Always commit `package-lock.json`. This file ensures every developer and every CI/CD build installs the exact same dependency tree, eliminating "works on my machine" failures. Never delete and regenerate the lock file without a documented reason.

### 2.4 Test After Every Dependency Update

After any library update — even a patch:

1. Run `npm install`
2. Run `npm run typecheck`
3. Start the dev server and manually test features related to the updated library on a device
4. Run a clean EAS development build before marking the update complete

### 2.5 Watch for Expo SDK Deprecations

The Expo team announces API deprecations in advance. Subscribe to the [Expo blog](https://expo.dev/blog) and review SDK changelogs on each release. Key areas to watch for TaskAces:

- `expo-notifications` — notification scheduling format and permission APIs have changed across SDK versions
- `expo-auth-session` — OAuth flow and redirect URI handling updates
- Expo Router — navigation API changes between minor versions

### 2.6 Google Sign-In Maintenance

`@react-native-google-signin/google-signin` depends on OAuth client IDs registered in Google Cloud Console. If client IDs are rotated, OAuth consent screen scopes change, or the app's bundle identifiers change:

- Update the relevant values in `.env`
- Re-test the full sign-in flow on both iOS and Android physical devices before releasing

### 2.7 Supabase Client SDK

Before upgrading `@supabase/supabase-js` across major versions:

- Read the Supabase JS migration guide for that version
- Test all auth flows: sign-in, session persistence across app restarts, and sign-out
- Verify that Row Level Security policies still apply correctly after any schema or client changes

---

## 3. Repository and Versioning Management

### 3.1 Semantic Versioning

TaskAces follows **Semantic Versioning** (`MAJOR.MINOR.PATCH`):

| Segment | When to increment |
|---------|------------------|
| MAJOR | Breaking change visible to users — complete redesign, incompatible data migration |
| MINOR | New feature added in a backwards-compatible way |
| PATCH | Bug fix, performance improvement, copy change, or minor UI tweak |

Both [app.json](../app.json) and [package.json](../package.json) must reflect the same version string and must be updated together in the same commit for every release.

### 3.2 Build Numbers

- iOS build numbers and Android version codes are managed automatically by EAS (`"autoIncrement": true` in [eas.json](../eas.json)).
- Never manually edit build numbers in `ios/` or `android/` native files — let EAS be the single source of truth.
- The `"versionSource": "remote"` setting in `eas.json` means EAS tracks the authoritative build count.

### 3.3 Tag Every Release

Tag the exact commit that was submitted to the App Store and Play Store:

```bash
git tag -a v1.0.5 -m "Release 1.0.5 — dark mode persistence fix"
git push origin v1.0.5
```

Tags create a permanent, traceable snapshot and allow any past release to be checked out, reproduced, or hotfixed.

### 3.4 Maintain a Changelog

Keep a `CHANGELOG.md` in the repository root and update it with every release:

```markdown
## [1.1.0] — 2026-05-01
### Added
- Today view for tasks due today
### Fixed
- Notification not rescheduling when due date changes

## [1.0.5] — 2026-04-05
### Fixed
- Dark mode preference now persists after app restart
```

Use the Conventional Commits history to generate this semi-automatically with a tool like `conventional-changelog`.

### 3.5 Branch Protection Rules

Configure the following rules on the `main` branch in GitHub:

- Require at least 1 pull request review before merging
- Require status checks to pass (lint, typecheck)
- Disallow direct pushes — no commits bypass PR review
- Disallow force pushes
- Disallow branch deletion

### 3.6 Repository Hygiene

- Delete or archive stale branches after they are merged.
- Move in-progress work to feature branches — the `app/temp/` directory currently contains an uncommitted WIP screen (`_today.tsx`) that should live in a feature branch, not `main`.
- Never commit `.env`, `node_modules/`, build artifacts (`android/app/build/`, `ios/build/`), or EAS credentials files.

---

## 4. Licensing Management

### 4.1 Add a Project License

The repository currently has no `LICENSE` file, meaning all rights are implicitly reserved and the code cannot legally be used, modified, or distributed by others. Choose a license and add it as a `LICENSE` file in the repository root before any public release or handoff.

| License | When to use |
|---------|------------|
| MIT | Permissive — others can use the code freely in any context |
| Apache 2.0 | Permissive with explicit patent protection clause |
| GPL v3 | Copyleft — anyone who distributes must open-source their changes |
| Proprietary | Closed-source commercial product — all rights reserved |

For a commercial mobile app, either **MIT** (open) or a **proprietary all-rights-reserved** declaration is most common.

### 4.2 Third-Party License Compliance

All current dependencies are MIT-licensed, which allows use in closed-source commercial applications with no restrictions beyond attribution. To monitor this:

```bash
npx license-checker --summary              # quick overview
npx license-checker --csv > licenses.csv   # full export for audits
```

Run this check before any major dependency update to detect if a new transitive dependency introduces a restrictive license (GPL, AGPL, CC-BY-NC) that would require legal review.

### 4.3 App Store and Play Store Requirements

Both stores require a **privacy policy** in the app listing. The privacy policy must be hosted at a stable URL and disclose:

- What data is collected (task content, Google account info if signed in, photos)
- Where data is stored (on-device, and Supabase if signed in)
- How data is used
- How users can request deletion

### 4.4 Third-Party Service Terms — Annual Review

Review the terms of each integrated service annually or when upgrading to a paid plan:

| Service | Key term to monitor |
|---------|-------------------|
| Google Sign-In | Google API Terms of Service — prohibited uses and scope restrictions |
| Supabase | Free tier limits (project pausing, row/storage caps) vs paid plan |
| Expo / EAS | Free tier build limits — EAS Production plan for high-volume releases |
| Apple Developer | Annual $99 membership renewal — lapse removes app from App Store |
| Google Play | One-time $25 registration — no renewal, but developer policy updates require review |

---

## 5. Hosting and Domain Services

### 5.1 Supabase Project Management

- **Do not use the free tier for production.** The Supabase free tier pauses projects after 1 week of inactivity and enforces limits on storage and database size. Upgrade to the **Pro plan** before public launch.
- Enable **Point-in-Time Recovery (PITR)** on the Pro plan to restore the database to any moment within the retention window.
- **Rotate API keys** annually, or immediately if any key is accidentally exposed in a commit or log.
- Monitor database size, active auth users, and storage usage monthly from the Supabase dashboard.
- Enable **daily automatic backups** — included on the Pro plan.

### 5.2 Supabase Edge Functions

Edge Functions in `supabase/functions/` are deployed with the Supabase CLI:

```bash
supabase functions deploy <function-name>
```

- Keep Edge Functions small and single-purpose.
- Store secrets (API keys, webhooks) in **Supabase Secrets**, not in function source code.
- Test functions locally with `supabase functions serve` before deploying to production.

### 5.3 Expo Application Services (EAS)

- Enable **OTA (Over-the-Air) updates** for JavaScript-only fixes — bug fixes and copy changes can be pushed to users without App Store or Play Store review.
- Use EAS **channels** (`production`, `preview`) to deliver updates to the correct audience.
- OTA updates cannot change native modules, app permissions, icons, or splash screens — those changes require a full binary build and store review.
- Store EAS credentials securely. Do not commit `credentials.json` or Apple/Google service account key files to the repository.

### 5.4 Domain and Web Presence

TaskAces is a mobile-only app with no web frontend. Supabase provides its own API endpoint (`https://<project-ref>.supabase.co`). If a marketing site or support portal is added in the future:

- Register the domain through a reputable registrar (Namecheap, Cloudflare Registrar)
- Use **Cloudflare** for DNS and DDoS protection
- Host static sites on **Vercel** or **Netlify** (both have free tiers adequate for a landing page)
- Set up auto-renewal on the domain to prevent accidental expiration

---

## 6. Software Distribution Platforms

### 6.1 Apple App Store

**Current identifiers:**  
Bundle ID: `com.zearot.boltexponativewind`  
App Store Connect App ID: `6755730110`

**Best practices:**

- **Two-factor authentication:** Ensure the Apple Developer account has 2FA enabled. A compromised account can push updates to all users instantly.
- **App Store Connect API Key:** Use an API key (not an Apple ID password) for EAS submission — this avoids storing credentials in plain text and supports CI/CD pipelines.
- **Certificates and provisioning profiles:** These are managed automatically by EAS. Do not manually create, revoke, or download certificates through Xcode unless directed — doing so can break EAS builds and require credential regeneration.
- **Privacy policy:** Apple requires a privacy policy URL in every store listing. Host it at a stable URL that will not move or go offline.
- **Permission strings:** Camera, photo library, and notification usage require a purpose string in `app.json` explaining why the app needs each permission. Missing or vague strings are a common cause of App Store rejection.
- **TestFlight first:** Always release to TestFlight internal testers for at least 48 hours before submitting for App Store review. This catches environment-specific bugs that simulators miss.
- **Review guidelines:** Re-read the [App Store Review Guidelines](https://developer.apple.com/app-store/review/guidelines/) before submitting a new feature — guidelines change several times per year.

### 6.2 Google Play Store

**Current identifier:**  
Package name: `com.brunnhilde14.boltexponativewind`

**Best practices:**

- **Service account:** Create a Google Play service account in Google Cloud Console and upload the JSON key to EAS for automated submissions. Use this instead of personal Google account credentials.
- **Play App Signing:** Enroll in Play App Signing so Google holds the upload key. This protects against catastrophic keystore loss and allows key recovery.
- **Internal testing track:** Promote builds through Internal Testing → Closed Testing → Production, never directly to Production without testing.
- **Target API level:** Google enforces a minimum `targetSdkVersion` for new and existing apps. Expo and React Native handle this, but verify the target SDK level on every Expo SDK upgrade.
- **Data Safety form:** Complete the Data Safety section in Play Console, disclosing all data the app collects: task content (stored locally), Google account profile (if signed in), photos (stored locally).

### 6.3 OTA Updates (EAS Update)

For JavaScript-layer changes only (bug fixes, text changes, minor UI tweaks):

```bash
eas update --branch production --message "Fix notification scheduling bug"
```

OTA updates:
- Skip App Store / Play Store review — reach users within minutes
- Cannot modify native code, permissions, or binary-level changes
- Should not be used to add significant new features that should go through store review
- Keep a log of all OTA update messages alongside binary release notes in `CHANGELOG.md`

---

## 7. Database Services

### 7.1 Schema Changes via Migrations Only

All database schema changes must be made through migration files committed to the repository. Never alter tables directly in the Supabase dashboard on a production database.

```bash
# Create a new migration file
supabase migration new add_task_notes_column

# Test locally
supabase db reset

# Deploy to production
supabase db push
```

Migration files live in `supabase/migrations/` and are version-controlled alongside the app code. This ensures the schema history is auditable and the database can be rebuilt from scratch at any time.

### 7.2 Row Level Security — Never Disable

Row Level Security (RLS) is already enabled on all tables (`tasks`, `task_photos`, `lists`). This is the most critical database security control in the project.

Rules:
- **Never disable RLS** on any table that contains user data.
- Every new table created must have RLS enabled and a user-scoped policy before it is used by the app.
- Test RLS policies by attempting to query another user's rows using a different JWT — the result should always be zero rows.
- Review all RLS policies when adding new tables or changing the auth model.

### 7.3 Backup and Recovery

| Environment | Backup approach |
|-------------|----------------|
| Development | None required — rebuild from migrations with `supabase db reset` |
| Staging | Weekly manual export: `supabase db dump -f staging_backup.sql` |
| Production | Supabase Pro daily automatic backups + Point-in-Time Recovery (PITR) |

Store production backup exports in a location separate from the primary database — an encrypted S3 bucket or a restricted Google Drive folder. Test the restore process at least once per quarter by restoring to a staging environment.

### 7.4 Local Storage (AsyncStorage)

AsyncStorage is the primary datastore for tasks and is stored unencrypted on the device filesystem.

- **Do not store sensitive data** — passwords, auth tokens, or PII beyond task content — in AsyncStorage without encryption. Auth tokens are handled securely by the Supabase and Google Sign-In SDKs and must not be duplicated into AsyncStorage manually.
- The `@tasks_storage` key holds the entire task array as a JSON string. As data grows, consider paginating reads or introducing an index rather than deserializing the entire array on every screen mount.
- If cloud sync is added in the future, implement a **conflict resolution strategy** before merging local and remote state. The recommended approach is **last-write-wins** using the `updated_at` timestamp already present on every task record.

### 7.5 Query Performance

When Supabase sync is active:

- Always include a `user_id` filter on every query so the database uses the index instead of performing a full table scan.
- Avoid `select('*')` in production — select only the columns the screen needs.
- Add database indexes on `tasks.user_id` and `tasks.due_date` to support the Today, This Week, and This Month filter views efficiently.
- Monitor slow queries monthly in the Supabase dashboard under **Database > Query Performance**.

### 7.6 Data Retention and User Privacy

Define and document a data retention policy before public launch:

- Deleted tasks should be **soft-deleted** (a `deleted_at` column set to the deletion timestamp) for a defined grace period (e.g., 30 days), then purged by a scheduled job.
- If a user requests **account deletion** (required under GDPR for EU users and CCPA for California users), implement a cascade delete that removes all rows tied to their `user_id` across every table in a single transaction.
- Document what data is collected, where it is stored, and for how long in the app's **privacy policy** and **Data Safety** disclosure on Google Play.

---

*TaskAces v1.0.5 — Best Practices — April 2026*
