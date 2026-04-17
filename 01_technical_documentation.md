# TaskAces — Technical Documentation

**App Name:** TaskAces  
**Version:** 1.0.5  
**Platform:** iOS & Android (React Native / Expo)  
**Repository:** https://github.com/qualaces/mobileAssistantApp.git  
**Date:** April 2026  

---

## 1. Architecture Diagrams

### 1.1 System Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                         TaskAces App                            │
│                    (React Native / Expo)                        │
│                                                                 │
│   ┌──────────────┐   ┌──────────────┐   ┌───────────────────┐  │
│   │   app/       │   │    lib/      │   │    contexts/      │  │
│   │  (Screens)   │   │ (Utilities)  │   │  (State / Theme)  │  │
│   └──────┬───────┘   └──────┬───────┘   └────────┬──────────┘  │
│          │                  │                    │              │
│          └──────────────────┴────────────────────┘             │
│                             │                                   │
│               ┌─────────────┴─────────────┐                    │
│               │      Local Storage         │                    │
│               │      (AsyncStorage)        │                    │
│               └─────────────┬─────────────┘                    │
└─────────────────────────────┼───────────────────────────────────┘
                              │  (optional cloud sync)
              ┌───────────────┼───────────────┐
              │               │               │
       ┌──────▼──────┐ ┌──────▼──────┐ ┌─────▼──────────┐
       │  Supabase   │ │Google OAuth │ │  Expo Push     │
       │  (Postgres) │ │  Sign-In    │ │  Notification  │
       │  + Storage  │ │             │ │  Service       │
       └─────────────┘ └─────────────┘ └────────────────┘
```

### 1.2 Navigation Architecture

```
Stack Navigator  (app/_layout.tsx)
│
├── /  →  redirects to /(tabs)/lists
│
├── (tabs)/           Bottom Tab Navigator  (app/(tabs)/_layout.tsx)
│   ├── lists         →  Task List Screen
│   └── settings      →  Settings Screen
│
└── task/[id]         →  Task Detail / Edit Screen
```

### 1.3 Data Flow

```
User Action
    │
    ▼
Screen Component
(app/(tabs)/lists.tsx  or  app/task/[id].tsx)
    │
    ├── Read  →  AsyncStorage  (@tasks_storage)
    ├── Write →  AsyncStorage  (@tasks_storage)
    │
    └── [Optional] Supabase sync  (lib/supabase.ts)
              │
              └── PostgreSQL
                    ├── tasks
                    ├── task_photos
                    └── lists
```

### 1.4 Database Schema

```
tasks
  ├── id             UUID        primary key
  ├── user_id        UUID        foreign key → auth.users
  ├── task_number    INTEGER
  ├── title          TEXT
  ├── description    TEXT
  ├── priority       TEXT        'P1' | 'P2' | 'P3'
  ├── due_date       TIMESTAMP
  ├── completed      BOOLEAN
  ├── list_name      TEXT
  ├── status_emoji   TEXT
  ├── created_at     TIMESTAMP
  └── updated_at     TIMESTAMP

task_photos
  ├── id             UUID        primary key
  ├── task_id        UUID        foreign key → tasks.id
  ├── photo_uri      TEXT
  └── created_at     TIMESTAMP

lists
  ├── id             UUID        primary key
  ├── user_id        UUID        foreign key → auth.users
  ├── name           TEXT
  ├── color          TEXT
  └── created_at     TIMESTAMP
```

Row Level Security (RLS) is enabled on all tables. Each user can only read and write their own rows.

---

## 2. File Navigation

### 2.1 Top-Level Project Structure

```
project_2/
├── app/              Screens — all UI routes (Expo Router)
├── lib/              Stateless utility modules
├── contexts/         React Context providers (theme)
├── hooks/            Custom React hooks
├── supabase/         Database migrations and Edge Functions
├── assets/           Images, icons, splash screens
├── android/          Native Android project files
├── ios/              Native iOS project files
├── app.json          Expo app configuration
├── eas.json          EAS build profiles
├── babel.config.js   Babel configuration
├── tsconfig.json     TypeScript configuration
├── package.json      Dependencies and scripts
└── .env              Environment variables (not in git)
```

### 2.2 App Directory (Screens)

```
app/
├── _layout.tsx           Root Stack layout — wraps theme provider and gesture handler
├── index.tsx             Entry point — redirects to /(tabs)/lists
├── +not-found.tsx        404 fallback screen
├── (tabs)/
│   ├── _layout.tsx       Bottom tab bar — Lists and Settings tabs
│   ├── lists.tsx         Primary screen — task management (~900 lines)
│   └── settings.tsx      Dark mode toggle and Google Sign-In
├── task/
│   └── [id].tsx          Dynamic task detail and edit screen (~745 lines)
└── temp/
    └── _today.tsx        Work-in-progress Today view (not yet released)
```

### 2.3 Lib Directory (Utilities)

```
lib/
├── supabase.ts        Supabase client initialization and table helpers
├── googleAuth.ts      Google Sign-In: signIn, signOut, getCurrentUser
├── notifications.ts   Schedule and cancel Expo push notifications
├── nlp.ts             NLP parser: extracts priority, due date, and time from text
└── localAi.ts         Local task generation helper (no external API calls)
```

### 2.4 Contexts and Hooks

```
contexts/
└── ThemeContext.tsx    Light / dark theme provider and useTheme hook

hooks/
└── useFrameworkReady.ts    Expo framework initialization hook
```

### 2.5 Supabase Directory

```
supabase/
├── migrations/        SQL migration files — one file per schema change
└── functions/         Supabase Edge Functions (server-side logic)
```

### 2.6 Key Configuration Files

| File | Purpose |
|------|---------|
| [app.json](../app.json) | App name, bundle IDs, version, EAS project ID, permissions |
| [eas.json](../eas.json) | Build profiles for development, preview, and production |
| [babel.config.js](../babel.config.js) | Babel preset (expo) and Reanimated plugin |
| [tsconfig.json](../tsconfig.json) | TypeScript strict mode and `@/*` path alias |
| [package.json](../package.json) | Dependencies, dev dependencies, and npm scripts |

**Path alias:** `@/*` resolves to the project root, configured in `tsconfig.json`.

---

## 3. Installation Process

### 3.1 Prerequisites

| Tool | Required Version | Install |
|------|-----------------|---------|
| Node.js | 18 LTS or 20 LTS | https://nodejs.org |
| npm | Bundled with Node | — |
| Expo CLI | Latest | `npm install -g expo-cli` |
| EAS CLI | >= 16.19.3 | `npm install -g eas-cli` |
| Xcode | 15+ | Mac App Store (iOS only) |
| Android Studio | Latest | https://developer.android.com/studio |
| Expo Go | Latest | App Store / Play Store (quick preview) |

### 3.2 Local Development Setup

```bash
# 1. Clone the repository
git clone https://github.com/qualaces/mobileAssistantApp.git
cd mobileAssistantApp

# 2. Install dependencies
npm install

# 3. Create the environment variables file
cp .env.example .env
# Open .env and fill in all values (see Section 3.3)

# 4. Start the development server
npm run dev

# 5. Open on a device or simulator
npm run ios        # iOS Simulator — macOS only
npm run android    # Android emulator or connected device
```

Alternatively, scan the QR code shown in the terminal with the **Expo Go** app on a physical device.

### 3.3 Environment Variables

Create a `.env` file in the project root. All keys prefixed with `EXPO_PUBLIC_` are bundled into the client app by Expo.

```env
# Google OAuth Client IDs — from Google Cloud Console
EXPO_PUBLIC_GOOGLE_WEB_CLIENT_ID=<your-web-client-id>
EXPO_PUBLIC_GOOGLE_IOS_CLIENT_ID=<your-ios-client-id>
EXPO_PUBLIC_GOOGLE_ANDROID_CLIENT_ID=<your-android-client-id>

# Supabase — from your Supabase project dashboard > Settings > API
EXPO_PUBLIC_SUPABASE_URL=<https://your-project-ref.supabase.co>
EXPO_PUBLIC_SUPABASE_ANON_KEY=<your-anon-public-key>
```

Never commit `.env` to the repository. It is listed in `.gitignore`.

### 3.4 Supabase Database Setup

```bash
# Install the Supabase CLI
npm install -g supabase

# Log in
supabase login

# Link to the remote project
supabase link --project-ref <your-project-ref>

# Apply all migrations to the remote database
supabase db push
```

### 3.5 Cloud (EAS) Builds

```bash
# Authenticate with Expo
eas login

# Development build — installs a dev client on device
eas build --profile development --platform ios
eas build --profile development --platform android

# Preview build — internal distribution
eas build --profile preview --platform all

# Production build — App Store / Play Store submission
eas build --profile production --platform all
```

**EAS Project ID:** `da173da0-3ef8-4941-89b2-94641ee4caf5`  
**App Store Connect App ID:** `6755730110`

### 3.6 Available Scripts

| Script | Command | Description |
|--------|---------|-------------|
| Start dev server | `npm run dev` | Starts Expo with telemetry disabled |
| iOS | `npm run ios` | Opens iOS Simulator |
| Android | `npm run android` | Opens Android emulator |
| Web export | `npm run build:web` | Exports static web build |
| Lint | `npm run lint` | Runs ESLint |
| Type check | `npm run typecheck` | Runs TypeScript compiler check |

---

## 4. Licenses

### 4.1 Project License

The project is currently private (`"private": true` in `package.json`) and contains no `LICENSE` file. All rights are reserved. Before any public release or handoff, a license should be chosen and added as a `LICENSE` file in the repository root.

### 4.2 Third-Party Library Licenses

All production dependencies use permissive open-source licenses. A summary:

| License | Libraries |
|---------|----------|
| MIT | React, React Native, Expo, Expo Router, Supabase JS, AsyncStorage, NativeWind, Reanimated, Gesture Handler, Draggable FlatList, Lucide React Native, and all other listed libraries |
| ISC | Some minor utilities |

No copyleft (GPL/AGPL) licenses are present. MIT-licensed code can be used in closed-source commercial applications without restriction beyond attribution.

To generate a full license report at any time:

```bash
npx license-checker --summary
npx license-checker --csv > licenses.csv   # full export
```

### 4.3 Third-Party Service Terms

| Service | Terms reference |
|---------|----------------|
| Google Sign-In | Google APIs Terms of Service |
| Supabase | Supabase Terms of Service |
| Expo / EAS | Expo Terms of Service |
| Apple App Store | Apple Developer Program License Agreement |
| Google Play | Google Play Developer Distribution Agreement |

---

## 5. Libraries

### 5.1 Runtime Dependencies

| Library | Version | Source | Purpose |
|---------|---------|--------|---------|
| react | 19.1.0 | npmjs.com | Core UI framework |
| react-native | 0.81.5 | npmjs.com | Mobile runtime |
| expo | ~54.0.33 | npmjs.com | Managed workflow and SDK |
| expo-router | ~6.0.23 | npmjs.com | File-based navigation |
| @react-navigation/bottom-tabs | ^7.3.10 | npmjs.com | Bottom tab navigator |
| @react-navigation/native | ^7.1.6 | npmjs.com | Navigation container |
| nativewind | ^4.1.23 | npmjs.com | Tailwind CSS for React Native |
| react-native-reanimated | ~4.1.1 | npmjs.com | Gesture-driven animations |
| react-native-gesture-handler | ~2.23.1 | npmjs.com | Touch gesture recognition |
| react-native-draggable-flatlist | ^4.0.1 | npmjs.com | Drag-to-reorder lists |
| @supabase/supabase-js | ^2.58.0 | npmjs.com | Backend: auth, database, storage |
| @react-native-async-storage/async-storage | ^2.2.0 | npmjs.com | Local on-device persistence |
| expo-notifications | ~0.31.2 | npmjs.com | Local push notifications |
| @react-native-google-signin/google-signin | ^15.0.0 | npmjs.com | Google OAuth authentication |
| expo-auth-session | ~6.1.4 | npmjs.com | OAuth session management |
| expo-image-picker | ~16.1.4 | npmjs.com | Camera roll / gallery access |
| expo-camera | ~16.1.6 | npmjs.com | In-app camera capture |
| expo-file-system | ~18.1.1 | npmjs.com | File I/O for photo storage |
| lucide-react-native | ^0.475.0 | npmjs.com | Icon library |
| @expo/vector-icons | ^14.0.4 | npmjs.com | Extended icon set |
| react-native-safe-area-context | 4.16.1 | npmjs.com | Safe area insets |
| react-native-screens | ~4.11.1 | npmjs.com | Native screen containers |
| react-native-url-polyfill | ^2.0.0 | npmjs.com | URL API polyfill (Supabase requirement) |
| expo-constants | ~17.1.6 | npmjs.com | Build-time constants |
| expo-linking | ~7.1.4 | npmjs.com | Deep linking |
| expo-splash-screen | ~0.30.8 | npmjs.com | Splash screen control |
| expo-status-bar | ~2.2.3 | npmjs.com | Status bar theming |
| expo-web-browser | ~14.1.0 | npmjs.com | OAuth web browser session |

### 5.2 Dev Dependencies

| Library | Version | Purpose |
|---------|---------|---------|
| typescript | ~5.9.2 | Static type checking |
| @babel/core | ^7.25.2 | JavaScript transpilation |
| babel-preset-expo | ~13.0.0 | Expo-specific Babel preset |

---

## 6. API / SDK

### 6.1 Supabase

| Property | Value |
|----------|-------|
| SDK | `@supabase/supabase-js` v2.58.0 |
| Source | https://supabase.com — npmjs.com |
| Config | [lib/supabase.ts](../lib/supabase.ts) |
| Services used | PostgreSQL database, Auth (JWT), Row Level Security |
| Session storage | AsyncStorage (persists sessions across restarts) |
| Tables | `tasks`, `task_photos`, `lists` |

The Supabase client is initialized once in `lib/supabase.ts` and imported wherever database access is needed.

### 6.2 Google Sign-In

| Property | Value |
|----------|-------|
| SDK | `@react-native-google-signin/google-signin` v15.0.0 |
| Source | npmjs.com — credentials from Google Cloud Console |
| Config | [lib/googleAuth.ts](../lib/googleAuth.ts) |
| Client IDs | Set via `.env` (Web, iOS, Android variants) |
| Functions exposed | `signInWithGoogle()`, `signOutGoogle()`, `getCurrentUser()` |
| Usage | Settings screen — optional, app works without sign-in |

### 6.3 Expo Notifications

| Property | Value |
|----------|-------|
| SDK | `expo-notifications` ~0.31.2 |
| Source | npmjs.com — Expo SDK |
| Config | [lib/notifications.ts](../lib/notifications.ts) |
| Behavior | Schedules a local notification at 12:00 PM on a task's due date |
| Platform notes | Android requires a notification channel (MAX importance, configured in `lib/notifications.ts`) |
| Functions exposed | `scheduleTaskNotification()`, `cancelTaskNotification()` |

### 6.4 Expo Application Services (EAS)

| Property | Value |
|----------|-------|
| CLI | `eas-cli` >= 16.19.3 |
| Source | https://expo.dev/eas — npmjs.com |
| Config | [eas.json](../eas.json) |
| Profiles | `development`, `preview`, `production` |
| Used for | Cloud builds, OTA updates, App Store / Play Store submission |
| EAS Project ID | `da173da0-3ef8-4941-89b2-94641ee4caf5` |

### 6.5 Expo Auth Session

| Property | Value |
|----------|-------|
| SDK | `expo-auth-session` ~6.1.4 |
| Source | npmjs.com — Expo SDK |
| Used for | Managing the OAuth browser session during Google Sign-In |

---

*TaskAces v1.0.5 — Technical Documentation — April 2026*
