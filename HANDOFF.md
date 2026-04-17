# TaskAces — Project Handoff Document

**App Name:** TaskAces  
**Version:** 1.0.5  
**Platform:** iOS & Android (React Native / Expo)  
**Repository:** https://github.com/qualaces/mobileAssistantApp.git  
**Date:** April 2026  

---

# Part 1 — Technical Documentation

## 1.1 Architecture Diagrams

### System Overview

```
┌─────────────────────────────────────────────────────────────┐
│                        TaskAces App                         │
│                   (React Native / Expo)                     │
│                                                             │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────────┐  │
│  │   app/       │  │   lib/       │  │   contexts/      │  │
│  │  (Screens)   │  │ (Utilities)  │  │  (State/Theme)   │  │
│  └──────┬───────┘  └──────┬───────┘  └────────┬─────────┘  │
│         │                 │                   │             │
│         └─────────────────┴───────────────────┘            │
│                           │                                 │
│              ┌────────────┴───────────┐                    │
│              │    Local Storage       │                    │
│              │   (AsyncStorage)       │                    │
│              └────────────┬───────────┘                    │
└───────────────────────────┼─────────────────────────────────┘
                            │ (optional cloud sync)
            ┌───────────────┼───────────────┐
            │               │               │
     ┌──────▼──────┐ ┌──────▼──────┐ ┌─────▼──────┐
     │  Supabase   │ │Google OAuth │ │  Expo Push │
     │  (Postgres) │ │  Sign-In    │ │Notification│
     │  + Storage  │ │             │ │  Service   │
     └─────────────┘ └─────────────┘ └────────────┘
```

### Navigation Architecture

```
Stack Navigator (app/_layout.tsx)
│
├── / → redirects to /(tabs)/lists
│
├── (tabs)/   (Bottom Tab Navigator)
│   ├── lists        → Task List Screen
│   └── settings     → Settings Screen
│
└── task/[id]        → Task Detail/Edit Screen
```

### Data Flow

```
User Action
    │
    ▼
Screen Component (app/(tabs)/lists.tsx or app/task/[id].tsx)
    │
    ├── Reads from AsyncStorage (@tasks_storage)
    ├── Writes to AsyncStorage  (@tasks_storage)
    │
    └── [Optional] Supabase sync (lib/supabase.ts)
            │
            └── PostgreSQL (tasks, task_photos, lists tables)
```

### Database Schema (Supabase)

```
tasks
  ├── id            UUID (PK)
  ├── user_id       UUID (FK → auth.users)
  ├── task_number   INTEGER
  ├── title         TEXT
  ├── description   TEXT
  ├── priority      TEXT ('P1' | 'P2' | 'P3')
  ├── due_date      TIMESTAMP
  ├── completed     BOOLEAN
  ├── list_name     TEXT
  ├── status_emoji  TEXT
  ├── created_at    TIMESTAMP
  └── updated_at    TIMESTAMP

task_photos
  ├── id            UUID (PK)
  ├── task_id       UUID (FK → tasks.id)
  ├── photo_uri     TEXT
  └── created_at    TIMESTAMP

lists
  ├── id            UUID (PK)
  ├── user_id       UUID (FK → auth.users)
  ├── name          TEXT
  ├── color         TEXT
  └── created_at    TIMESTAMP
```

---

## 1.2 File Navigation

```
project_2/
├── app/                          # All screens (Expo Router)
│   ├── _layout.tsx               # Root Stack layout, theme/gesture setup
│   ├── index.tsx                 # Entry point — redirects to /lists
│   ├── +not-found.tsx            # 404 fallback screen
│   ├── (tabs)/
│   │   ├── _layout.tsx           # Bottom tab bar (Lists + Settings)
│   │   ├── lists.tsx             # Primary screen — task management (900+ lines)
│   │   └── settings.tsx          # Dark mode toggle & Google Sign-In
│   ├── task/
│   │   └── [id].tsx              # Dynamic task detail/edit screen (745 lines)
│   └── temp/
│       └── _today.tsx            # WIP — today view (not yet released)
│
├── lib/                          # Stateless utility modules
│   ├── supabase.ts               # Supabase client initialization
│   ├── googleAuth.ts             # Google Sign-In helpers
│   ├── notifications.ts          # Expo push notification helpers
│   ├── nlp.ts                    # NLP parser for task input
│   └── localAi.ts                # Local task generation (no API call)
│
├── contexts/
│   └── ThemeContext.tsx           # Light/dark theme provider + hook
│
├── hooks/
│   └── useFrameworkReady.ts      # Expo framework initialization hook
│
├── supabase/
│   ├── migrations/               # SQL schema migrations
│   └── functions/                # Supabase Edge Functions
│
├── assets/                       # Images, icons, splash screens
├── android/                      # Native Android project files
├── ios/                          # Native iOS project files
│
├── app.json                      # Expo app config (name, bundle IDs, version)
├── eas.json                      # EAS build profiles (dev/preview/production)
├── babel.config.js               # Babel config (expo preset + reanimated)
├── tsconfig.json                 # TypeScript config (strict, path alias @/*)
├── package.json                  # Dependencies, scripts, metadata
└── .env                          # Environment variables (not committed to git)
```

**Key path alias:** `@/*` maps to the project root, configured in [tsconfig.json](tsconfig.json).

---

## 1.3 Installation Process

### Prerequisites

| Tool | Version | Notes |
|------|---------|-------|
| Node.js | 18 or 20 LTS | Required |
| npm | Bundled with Node | Or yarn/bun |
| Expo CLI | Latest | `npm install -g expo-cli` |
| EAS CLI | >= 16.19.3 | `npm install -g eas-cli` |
| Xcode | 15+ | iOS builds only (macOS) |
| Android Studio | Latest | Android builds only |
| Expo Go app | Latest | For quick device preview |

### Steps

```bash
# 1. Clone the repository
git clone https://github.com/qualaces/mobileAssistantApp.git
cd mobileAssistantApp

# 2. Install dependencies
npm install

# 3. Set up environment variables
cp .env.example .env   # if .env.example exists, otherwise create .env manually
# Fill in the values listed in the Environment Variables section below

# 4. Start the development server
npm run dev

# 5. Run on a device or simulator
npm run ios       # iOS Simulator (macOS only)
npm run android   # Android emulator or connected device
```

### Environment Variables

Create a `.env` file in the project root with the following keys:

```env
# Google OAuth Client IDs (from Google Cloud Console)
EXPO_PUBLIC_GOOGLE_WEB_CLIENT_ID=<your-web-client-id>
EXPO_PUBLIC_GOOGLE_IOS_CLIENT_ID=<your-ios-client-id>
EXPO_PUBLIC_GOOGLE_ANDROID_CLIENT_ID=<your-android-client-id>

# Supabase (from your Supabase project dashboard)
EXPO_PUBLIC_SUPABASE_URL=<your-supabase-url>
EXPO_PUBLIC_SUPABASE_ANON_KEY=<your-supabase-anon-key>
```

### EAS (Cloud) Builds

```bash
# Authenticate with Expo
eas login

# Build for development (device testing)
eas build --profile development --platform ios
eas build --profile development --platform android

# Build for production (App Store / Play Store)
eas build --profile production --platform all
```

**EAS Project ID:** `da173da0-3ef8-4941-89b2-94641ee4caf5`  
**App Store Connect App ID:** `6755730110`  
**Apple ID:** `saulmacias5142004@gmail.com`

---

## 1.4 Licenses

The project does not include an explicit `LICENSE` file in the repository. All first-party code is private (`"private": true` in package.json).

Third-party libraries are governed by their respective open-source licenses — primarily **MIT**. Notable exceptions:

| Library | License |
|---------|---------|
| React Native | MIT |
| Expo SDK | MIT |
| Supabase JS | MIT |
| NativeWind | MIT |
| React Native Reanimated | MIT |
| React Native Draggable FlatList | MIT |
| Lucide React Native | ISC |
| AsyncStorage | MIT |

To generate a full license report run:

```bash
npx license-checker --summary
```

---

## 1.5 Libraries

### Runtime Dependencies

| Library | Version | Source | Purpose |
|---------|---------|--------|---------|
| react | 19.1.0 | npmjs.com | UI framework |
| react-native | 0.81.5 | npmjs.com | Mobile runtime |
| expo | ~54.0.33 | npmjs.com | Managed workflow, dev tooling |
| expo-router | ~6.0.23 | npmjs.com | File-based navigation |
| @react-navigation/bottom-tabs | ^7.3.10 | npmjs.com | Bottom tab navigator |
| @react-navigation/native | ^7.1.6 | npmjs.com | Navigation container |
| nativewind | ^4.1.23 | npmjs.com | Tailwind CSS for React Native |
| react-native-reanimated | ~4.1.1 | npmjs.com | Gesture animations |
| react-native-gesture-handler | ~2.23.1 | npmjs.com | Touch gesture support |
| react-native-draggable-flatlist | ^4.0.1 | npmjs.com | Drag-to-reorder lists |
| @supabase/supabase-js | ^2.58.0 | npmjs.com | Backend client (auth, DB, storage) |
| @react-native-async-storage/async-storage | ^2.2.0 | npmjs.com | Local persistence |
| expo-notifications | ~0.31.2 | npmjs.com | Push notifications |
| @react-native-google-signin/google-signin | ^15.0.0 | npmjs.com | Google OAuth sign-in |
| expo-auth-session | ~6.1.4 | npmjs.com | OAuth session handling |
| expo-image-picker | ~16.1.4 | npmjs.com | Camera roll access |
| expo-camera | ~16.1.6 | npmjs.com | In-app camera |
| expo-file-system | ~18.1.1 | npmjs.com | File I/O for photos |
| lucide-react-native | ^0.475.0 | npmjs.com | Icon library |
| @expo/vector-icons | ^14.0.4 | npmjs.com | Extended icon set |
| react-native-safe-area-context | 4.16.1 | npmjs.com | Safe area insets |
| react-native-screens | ~4.11.1 | npmjs.com | Native screen containers |
| react-native-url-polyfill | ^2.0.0 | npmjs.com | URL API polyfill (Supabase req.) |

### Dev Dependencies

| Library | Version | Purpose |
|---------|---------|---------|
| typescript | ~5.9.2 | Static typing |
| @babel/core | ^7.25.2 | Transpilation |
| babel-preset-expo | ~13.0.0 | Expo Babel preset |
| expo-constants | ~17.1.6 | Build constants access |
| expo-linking | ~7.1.4 | Deep linking |
| expo-splash-screen | ~0.30.8 | Splash screen control |
| expo-status-bar | ~2.2.3 | Status bar theming |
| expo-web-browser | ~14.1.0 | OAuth web browser |

---

## 1.6 APIs and SDKs

### Supabase

| Property | Value |
|----------|-------|
| SDK | `@supabase/supabase-js` v2.58.0 |
| Source | https://supabase.com / npmjs.com |
| Auth | Row Level Security (RLS) per user |
| Config file | [lib/supabase.ts](lib/supabase.ts) |
| Services used | Auth, PostgreSQL Database, Storage |

Supabase is initialized with `AsyncStorage` as the auth session storage so sessions persist across app restarts.

### Google Sign-In

| Property | Value |
|----------|-------|
| SDK | `@react-native-google-signin/google-signin` v15.0.0 |
| Source | npmjs.com / Google Cloud Console |
| Config file | [lib/googleAuth.ts](lib/googleAuth.ts) |
| Client IDs | Set via `.env` (Web, iOS, Android) |

### Expo Notifications

| Property | Value |
|----------|-------|
| SDK | `expo-notifications` ~0.31.2 |
| Source | npmjs.com / Expo |
| Config file | [lib/notifications.ts](lib/notifications.ts) |
| Behavior | Schedules local notifications at 12:00 PM for task due dates |

### Expo Application Services (EAS)

| Property | Value |
|----------|-------|
| CLI | `eas-cli` >= 16.19.3 |
| Source | https://expo.dev/eas |
| Config file | [eas.json](eas.json) |
| Used for | Cloud builds, OTA updates, App Store submission |

---

# Part 2 — User Guide

## 2.1 How the System Works

TaskAces is a mobile task management application built for iOS and Android. It allows users to create, organize, prioritize, and track tasks across multiple lists. The app is **local-first**: all task data is saved on the device using AsyncStorage, meaning the app works fully offline without any account. An optional Google Sign-In connects to a Supabase cloud backend for future cross-device sync.

Tasks are organized by **priority tier** (P1, P2, P3) and by **time-based lists** (Inbox, Today, This Week, This Month). Users can drag tasks between priority levels, attach photos, add status emojis, and receive push notification reminders for due dates. A dark mode is available and persists between sessions.

---

## 2.2 Goals and Functionality

### EPIC 1 — Task Management

**As a user, I want to create tasks so that I can track what I need to do.**
- Create a task with a title and optional due date
- Tasks are assigned to one of three priority levels: P1 (High), P2 (Medium), P3 (Low)
- Tasks can be assigned to a named list (e.g., "Work", "Personal")

**As a user, I want to organize tasks by priority so that I focus on what matters most.**
- View tasks grouped under P1, P2, P3 headers
- Drag tasks between priority groups to re-rank them
- Priority groups are collapsible

**As a user, I want to mark tasks complete so that I can track my progress.**
- Tap a checkbox to mark a task done
- Completed tasks move to a collapsible "Completed" section at the bottom

**As a user, I want to delete tasks so that I can remove things I no longer need.**
- Swipe left on any task to reveal a delete action
- Delete from within the task detail screen

### EPIC 2 — Task Views & Filtering

**As a user, I want to filter tasks by time frame so that I focus on what's relevant now.**
- **Inbox** — all tasks
- **Today** — tasks due today
- **This Week** — tasks due within the current week
- **This Month** — tasks due within the current month

**As a user, I want to sort tasks so that I can see them in my preferred order.**
- Sort options: Created date, Priority, Due date, Alphabetical

### EPIC 3 — Task Detail & Enrichment

**As a user, I want to view and edit task details so that I can update tasks as things change.**
- Edit title, priority, and due date from the detail screen
- Access via tapping a task row

**As a user, I want to attach photos to a task so that I can store visual context.**
- Take a photo with the camera or select from the device gallery
- Photos appear in a scrollable list on the task detail screen
- Pinch-to-zoom photo viewer

**As a user, I want to add a status emoji to a task so that I can communicate its state at a glance.**
- Quick-pick from 4 preset emojis
- Or enter any custom emoji

### EPIC 4 — Notifications

**As a user, I want to receive reminders for upcoming tasks so that I don't miss due dates.**
- When a due date is set, a local notification is scheduled for 12:00 PM on that day
- Notifications are rescheduled when due dates change

### EPIC 5 — Appearance

**As a user, I want a dark mode so that I can use the app comfortably at night.**
- Toggle dark/light mode from the Settings screen
- Preference persists across app restarts

### EPIC 6 — Account (Optional)

**As a user, I want to sign in with Google so that my tasks can sync across devices in the future.**
- Google Sign-In from the Settings screen
- App works fully without an account (local storage fallback)

---

## 2.3 Inputs and Outputs

### Inputs

| Input | Where | Description |
|-------|-------|-------------|
| Task title | New task modal / Task detail | Text field, required |
| Priority | New task modal / Task detail | P1, P2, or P3 selector |
| Due date | New task modal / Task detail | Date picker |
| List name | New task modal | Text field (defaults to "Inbox") |
| Status emoji | Task detail | Quick emoji picker or custom |
| Photos | Task detail | Camera or gallery picker |
| Sort preference | Lists screen toolbar | Picker (date/priority/due/alpha) |
| Filter (list view) | Lists screen tab bar | Inbox / Today / This Week / This Month |
| Dark mode toggle | Settings screen | On/off switch |
| Google account | Settings screen | OAuth sign-in flow |

### Outputs

| Output | Where | Description |
|--------|-------|-------------|
| Task list | Lists screen | Priority-grouped, filtered task rows |
| Task count badges | Priority section headers | Count of tasks per priority |
| Completed section | Lists screen (bottom) | Collapsed list of finished tasks |
| Task detail view | Task detail screen | All task fields + photos |
| Push notification | Device notification tray | Reminder at 12:00 PM on due date |
| User profile | Settings screen | Avatar + name when signed into Google |
| Theme | App-wide | Light or dark color scheme |
| Persisted data | AsyncStorage | Survives app restarts and reboots |

---

## 2.4 Services and Features

### Local Storage (AsyncStorage)

All tasks are stored locally under the key `@tasks_storage` as a JSON array. This is the primary datastore. No internet connection is required for core functionality. Theme preference is stored under a separate key and persists independently.

### Cloud Backend (Supabase — Optional)

Supabase provides a PostgreSQL database, authentication, and file storage for future multi-device sync. Row Level Security (RLS) ensures each user can only access their own data. The integration is initialized in [lib/supabase.ts](lib/supabase.ts) but the app falls back gracefully to local storage when the user is not signed in.

### Google Authentication

Handled by `@react-native-google-signin/google-signin`. Supports sign-in, sign-out, and fetching the current user profile. Authentication state is surfaced in the Settings screen and is not required to use the app. See [lib/googleAuth.ts](lib/googleAuth.ts).

### Push Notifications

Implemented via `expo-notifications`. When a task is created or updated with a due date, a local notification is scheduled for 12:00 PM on that date. Notifications are cancelled and re-created when due dates change. Android uses a high-importance notification channel. See [lib/notifications.ts](lib/notifications.ts).

### NLP Task Parsing

A lightweight, no-API natural language parser lives in [lib/nlp.ts](lib/nlp.ts). It can extract:
- **Priority** from words like "urgent", "high", "P1"
- **Due dates** from phrases like "today", "tomorrow", "next week"
- **Times** from patterns like "3pm", "10:30 AM"

This module is implemented but not yet wired to the main task creation UI.

### Theme System

A React Context ([contexts/ThemeContext.tsx](contexts/ThemeContext.tsx)) provides light/dark theming to the entire app. Colors are defined as a static palette (primary blue `#2196F3`, secondary green `#4CAF50`, and surface colors per theme). The user's choice is read from AsyncStorage on launch and defaults to the system color scheme.

### Drag-to-Reorder

Powered by `react-native-draggable-flatlist`, tasks within each priority group can be reordered by long-pressing and dragging. Order is persisted to AsyncStorage.

### Swipe-to-Delete

Implemented with `react-native-gesture-handler`. Swiping a task row to the left reveals a red delete action. The deletion is immediate and updates AsyncStorage.

### Photo Attachments

Users can attach photos to tasks from the camera (`expo-camera`) or the device gallery (`expo-image-picker`). Photos are stored as local file URIs on the device and displayed in a scrollable row on the task detail screen with a pinch-to-zoom viewer.

---

*Generated April 2026 — TaskAces v1.0.5*
