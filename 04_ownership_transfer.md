# TaskAces — Transferring Ownership to the Sponsor

**App Name:** TaskAces  
**Version:** 1.0.5  
**Platform:** iOS & Android (React Native / Expo)  
**Date:** April 2026  

> This document provides a step-by-step guide for transferring all assets, credentials, and responsibilities for the TaskAces application from the development team to the sponsoring organization. Each section identifies what is being transferred, how to transfer it, and what the receiving party must do to complete the handoff.

---

## 1. Code Repository

### 1.1 What Is Being Transferred

The full source code for TaskAces, hosted at:

**GitHub Repository:** `https://github.com/qualaces/mobileAssistantApp.git`

This includes all application source, configuration files, database migration files, build scripts, and commit history.

### 1.2 Transfer Steps

**Option A — Transfer the repository directly to the sponsor's GitHub organization (recommended)**

1. The current owner navigates to the repository on GitHub: **Settings → Danger Zone → Transfer ownership**
2. Enter the sponsor's GitHub organization name or personal username
3. Confirm the transfer — GitHub sends a confirmation email to both parties
4. The sponsor accepts the transfer from their GitHub notifications
5. After transfer, the original URL (`github.com/qualaces/mobileAssistantApp`) automatically redirects to the new location for a grace period, then the redirect expires

**Option B — Fork / mirror to the sponsor's organization**

1. Sponsor creates a new repository in their GitHub organization
2. Development team runs:
   ```bash
   git clone --mirror https://github.com/qualaces/mobileAssistantApp.git
   cd mobileAssistantApp.git
   git push --mirror https://github.com/<sponsor-org>/<new-repo-name>.git
   ```
3. Full history, branches, and tags are preserved
4. Original repository is archived (set to read-only) after confirming the mirror is complete

### 1.3 Post-Transfer Checklist

- [ ] Sponsor confirms they can clone and run the project locally
- [ ] Sponsor has invited their own developers as repository collaborators
- [ ] Branch protection rules are re-applied on the new repository (see Best Practices doc)
- [ ] Original development team members are removed or their access downgraded
- [ ] EAS project is re-linked to the new repository owner's Expo account (see Section 5)

### 1.4 Repository Contents to Verify

| Item | Location | Status |
|------|---------|--------|
| Full application source | `app/`, `lib/`, `contexts/` | Included |
| Database migrations | `supabase/migrations/` | Included |
| Build configuration | `app.json`, `eas.json` | Included |
| Dependency manifest | `package.json`, `package-lock.json` | Included |
| Environment variable template | `.env.example` | Must be created before handoff |
| Actual secrets / credentials | `.env` | **NOT in repository** — transfer separately |

---

## 2. Databases

### 2.1 What Is Being Transferred

The Supabase project hosting the TaskAces cloud backend, including:

- PostgreSQL database (tables: `tasks`, `task_photos`, `lists`, auth tables)
- Row Level Security policies
- Supabase Edge Functions
- Authentication configuration (Google OAuth provider setup)
- Storage buckets (if photo cloud sync is enabled)

**Current Supabase Project Reference:** Available in the Supabase dashboard under **Project Settings → General**

### 2.2 Transfer Steps

Supabase does not currently support direct project ownership transfer between accounts. The recommended approach is to migrate to a new Supabase project owned by the sponsor:

1. **Export the current schema:**
   ```bash
   supabase db dump --schema-only -f schema.sql
   ```

2. **Export current data (if any production data exists):**
   ```bash
   supabase db dump --data-only -f data.sql
   ```

3. **Sponsor creates a new Supabase project** at https://supabase.com under their own organization and billing account

4. **Apply the schema to the new project:**
   ```bash
   supabase link --project-ref <new-project-ref>
   supabase db push
   ```
   Or apply `schema.sql` directly from the Supabase SQL editor.

5. **Import data** if migrating existing production records:
   ```bash
   psql <new-project-connection-string> < data.sql
   ```

6. **Reconfigure Google OAuth** in the new Supabase project:
   - Navigate to **Authentication → Providers → Google**
   - Enter the Google Cloud Console client credentials (see Section 4)

7. **Update environment variables** in the app and EAS build environment with the new project's URL and ANON key

8. **Verify RLS policies** are active on all tables in the new project before going live

### 2.3 Data Handoff Checklist

- [ ] Schema exported and imported successfully — all tables present
- [ ] RLS enabled on every table in the new project
- [ ] Auth provider (Google) re-configured in the new project
- [ ] New project URL and ANON key distributed to the sponsor's team via a secure channel
- [ ] Old Supabase project is archived or deleted after confirming the new project is live
- [ ] Supabase billing moved to the sponsor's payment method

### 2.4 Local Device Data

Task data stored locally via AsyncStorage lives on each user's device. There is nothing to transfer for local storage — it belongs to each individual user's device and cannot be extracted or migrated server-side.

---

## 3. Algorithms and Models

### 3.1 What Is Being Transferred

TaskAces does not use external machine learning models or paid AI services. Two internal modules implement lightweight algorithmic logic:

**NLP Task Parser (`lib/nlp.ts`)**  
A rule-based natural language parser that extracts task priority, due dates, and times from free-form text input. It uses pattern matching and keyword lists — no trained model, no external API, no weights file. The entire logic is contained in the source file and is transferred as part of the code repository.

**Local Task Generator (`lib/localAi.ts`)**  
A simple string-processing function that extracts a title (first sentence, max 50 characters) and description (remaining text, max 200 characters) from a block of text. No model involved. Transferred as part of the repository.

### 3.2 Licensing Note

Both algorithms were written entirely by the development team and contain no third-party model weights, no licensed datasets, and no dependencies on external AI providers. Ownership transfers completely with the repository. No separate licensing agreement is required.

### 3.3 Future AI Integration

If the sponsor plans to integrate a large language model (e.g., OpenAI, Anthropic Claude, or Google Gemini) into TaskAces after handoff:

- Store API keys in `.env` only — never in source code
- Add the relevant SDK to `package.json` via `npm install`
- Reference the NLP module in `lib/nlp.ts` as the integration point — it is designed to be replaced by a more capable backend parser

---

## 4. Server Credentials

> **Security notice:** Credentials must never be transferred via email, Slack, or any unencrypted channel. Use a password manager with sharing capability (1Password, Bitwarden, LastPass) or a secrets management tool (HashiCorp Vault, AWS Secrets Manager). Delete credentials from any temporary sharing location immediately after the recipient confirms receipt.

### 4.1 Credential Inventory

| Credential | Where it is used | How to transfer |
|------------|-----------------|----------------|
| Supabase URL | `.env` — Supabase client init | Share via password manager |
| Supabase ANON key | `.env` — Supabase client init | Share via password manager |
| Supabase service role key | Server-side only (Edge Functions) | Share via password manager |
| Google OAuth Web Client ID | `.env` | Share via password manager |
| Google OAuth iOS Client ID | `.env` | Share via password manager |
| Google OAuth Android Client ID | `.env` | Share via password manager |
| Google OAuth Client Secret | Google Cloud Console | Transfer console access (see Section 5) |
| Apple Developer account credentials | App Store Connect | Transfer account or create new (see Section 5) |
| EAS / Expo account | EAS CLI | Transfer project ownership (see Section 5) |
| Android keystore (if not using Play App Signing) | Local file | Encrypt and transfer via password manager |

### 4.2 Rotation After Transfer

Every credential should be rotated immediately after the transfer is confirmed. This limits exposure from the handoff process itself:

1. **Supabase:** Navigate to **Project Settings → API → Reset API keys** — generate new ANON and service role keys, then update `.env` in the new repository and redeploy
2. **Google OAuth:** Rotate the client secret in Google Cloud Console → **Credentials** → select the OAuth client → **Reset secret** — update in Supabase Auth settings and `.env`
3. **EAS / Expo:** The receiving party creates their own Expo account and the project is re-linked — old credentials are not shared

### 4.3 Creating a `.env` File for the Sponsor

Before handoff, create a `.env` file containing the current values and deliver it to the sponsor via a password manager or encrypted file transfer. Do not send `.env` contents in plain text. The `.env.example` file in the repository shows exactly which keys are needed.

---

## 5. Online Services Credentials

### 5.1 Expo / EAS Account

TaskAces is built and deployed through Expo Application Services.

**Current EAS Project ID:** `da173da0-3ef8-4941-89b2-94641ee4caf5`

**Transfer steps:**

1. Sponsor creates an account at https://expo.dev
2. Sponsor installs EAS CLI: `npm install -g eas-cli` and logs in: `eas login`
3. The development team updates `app.json` to set `extra.eas.projectId` to the sponsor's new project ID, or:
4. The development team adds the sponsor as an **owner** or **admin** in the Expo dashboard under the project's **Settings → Members**
5. The sponsor verifies they can run `eas build --profile development` successfully

### 5.2 Apple Developer Program

**Current account:** `saulmacias5142004@gmail.com`  
**App Store Connect App ID:** `6755730110`  
**Bundle ID:** `com.zearot.boltexponativewind`

**Transfer steps:**

**Option A — Transfer the app to the sponsor's Apple Developer account (recommended for ongoing ownership):**

1. Both the sender and recipient must have active Apple Developer Program memberships ($99/year)
2. In App Store Connect, navigate to the app → **App Information → Additional Information → Transfer App**
3. Follow Apple's transfer checklist (the app must have had at least one approved version, and the bundle ID must not be associated with In-App Purchases that cannot be transferred)
4. Apple's process takes 1–3 business days

**Option B — Add the sponsor as an Admin on the existing account:**

1. In App Store Connect → **Users and Access → Invite User**
2. Assign the **Admin** role to the sponsor's Apple ID
3. Transfer account primary ownership through Apple's account settings if full ownership is required

**After transfer:**

- Update the bundle ID in `app.json` to match the sponsor's preferred identifier if they want to use their own reverse-domain naming convention
- Re-run an EAS production build and re-submit to the App Store under the new account

### 5.3 Google Play Console

**Current package name:** `com.brunnhilde14.boltexponativewind`

**Transfer steps:**

Google Play does not support direct account-to-account app transfers at the organization level. The recommended approach:

**Option A — Add sponsor as account owner:**

1. Open Google Play Console → **Users and Permissions → Invite new users**
2. Assign **Account Owner** permissions
3. The original account owner can then be removed after the sponsor has access

**Option B — Re-publish under the sponsor's account (if the app has not yet launched publicly):**

1. Sponsor creates a Google Play Developer account ($25 one-time fee)
2. The app is published fresh under the sponsor's account with the new package name if desired
3. The original listing is unpublished or left archived

**After transfer:**

- Update the Android package name in `app.json` if it is changing
- Run `npm run android` and verify the app builds with the new identifier
- Rebuild the Android native project if the package name changes

### 5.4 Google Cloud Console (OAuth Credentials)

Google OAuth client IDs for Sign-In are managed in Google Cloud Console.

**Transfer steps:**

1. Open Google Cloud Console → **IAM & Admin → IAM**
2. Add the sponsor's Google account as an **Owner** on the project
3. The sponsor verifies they can see **APIs & Services → Credentials** and the OAuth 2.0 client IDs
4. Remove the development team accounts from IAM after confirming access
5. The sponsor rotates the OAuth client secret after receiving access (see Section 4.2)

### 5.5 Supabase Dashboard

**Transfer steps:**

1. Sponsor creates a Supabase account at https://supabase.com
2. The development team creates a new Supabase project under the sponsor's organization (see Section 2 for database migration steps)
3. The sponsor navigates to **Organization Settings → Members** and confirms they have **Owner** access
4. All development team members are removed from the organization after the sponsor confirms full access
5. Supabase billing is moved to the sponsor's payment method under **Organization Settings → Billing**

---

## 6. Contact Information for Ongoing Support

### 6.1 Development Team

The following individuals built TaskAces and are available for knowledge transfer sessions, bug reports, and onboarding questions during the post-handoff support period.

| Role | Name | Email | Availability |
|------|------|-------|-------------|
| Lead Developer | Saul M | saulmacias5142004@gmail.com | To be confirmed with sponsor |
| Team Member | *(add name)* | *(add email)* | *(add availability)* |
| Team Member | *(add name)* | *(add email)* | *(add availability)* |

> **Note to development team:** Fill in all team member names, emails, and availability windows before delivering this document to the sponsor.

### 6.2 Post-Handoff Support Period

Define a clear support window before handoff is complete. Recommended structure:

| Period | Support scope |
|--------|--------------|
| Weeks 1–2 | Daily availability — onboarding calls, environment setup assistance, credential questions |
| Weeks 3–4 | Response within 24 hours — bug triage, build pipeline questions |
| Months 2–3 | Response within 72 hours — architectural questions, major bug fixes only |
| After Month 3 | No obligation — contact at discretion of both parties |

### 6.3 Third-Party Support Channels

For issues with integrated services, the sponsor should contact the respective vendor directly:

| Service | Support channel |
|---------|----------------|
| Supabase | https://supabase.com/support — dashboard support ticket |
| Expo / EAS | https://expo.dev/support — support portal and Discord community |
| React Native | https://reactnative.dev — community forums and GitHub issues |
| Google Sign-In | Google Cloud Console support — https://cloud.google.com/support |
| Apple App Store | App Store Connect help — https://developer.apple.com/support |
| Google Play | Play Console Help Center — https://support.google.com/googleplay/android-developer |

### 6.4 Key Documentation Locations

All project documentation lives in the repository under the `docs/` directory:

| Document | File |
|----------|------|
| Technical Documentation | `docs/01_technical_documentation.md` |
| User Guide | `docs/02_user_guide.md` |
| Best Practices | `docs/03_best_practices.md` |
| Ownership Transfer (this document) | `docs/04_ownership_transfer.md` |

### 6.5 Handoff Sign-Off

Both parties should acknowledge receipt and completeness of the handoff before the support period begins.

| Item | Transferred | Confirmed by sponsor | Date |
|------|------------|---------------------|------|
| Source code repository access | ☐ | ☐ | |
| Supabase project access | ☐ | ☐ | |
| Apple Developer account / app access | ☐ | ☐ | |
| Google Play Console access | ☐ | ☐ | |
| Google Cloud Console (OAuth) access | ☐ | ☐ | |
| Expo / EAS project access | ☐ | ☐ | |
| `.env` credentials delivered securely | ☐ | ☐ | |
| Local development environment confirmed working | ☐ | ☐ | |
| All credentials rotated after handoff | ☐ | ☐ | |
| Documentation reviewed and accepted | ☐ | ☐ | |

---

*TaskAces v1.0.5 — Ownership Transfer — April 2026*
