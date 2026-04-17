# TaskAces — User Guide

**App Name:** TaskAces  
**Version:** 1.0.5  
**Platform:** iOS & Android  
**Date:** April 2026  

---

## 1. How the System Works

TaskAces is a mobile task management application built for iOS and Android. It gives users a structured way to capture, prioritize, and track tasks across their daily life and work.

The app is **local-first**: every task is saved directly on the user's device using on-device storage (AsyncStorage). This means the app works completely offline — no account, no internet connection, and no server is required for any core feature. Tasks are available instantly, without network delays, and survive app restarts.

Tasks are organized by a **three-tier priority system** (P1 High, P2 Medium, P3 Low) and by **time-based views** (Inbox, Today, This Week, This Month). Users can drag tasks between priority levels, swipe to delete, attach photos, set status emojis, and receive push notification reminders for due dates.

An optional **Google Sign-In** connects to a Supabase cloud backend. This lays the groundwork for cross-device sync in future releases, but signing in is not required — all features are available without an account.

A **dark mode** is available and persists between sessions.

---

## 2. Goals and Functionality

### EPIC 1 — Task Creation and Management

**Goal:** Allow users to create and manage tasks as quickly as possible.

---

**User Story 1.1 — Create a task**  
*As a user, I want to create a task so that I can track something I need to do.*

- Tap the **+** button on the Lists screen to open the new task modal
- Enter a task title (required)
- Optionally set a priority (P1 / P2 / P3) and due date
- Optionally assign the task to a named list (defaults to "Inbox")
- Tap **Save** — the task appears immediately in the list

---

**User Story 1.2 — Mark a task complete**  
*As a user, I want to mark tasks done so that I can see my progress.*

- Tap the circle checkbox on any task row
- The task moves to the **Completed** section at the bottom of the screen
- The completed section is collapsible — tap its header to show or hide it

---

**User Story 1.3 — Delete a task**  
*As a user, I want to delete tasks I no longer need.*

- **From the list:** Swipe a task row to the left to reveal the red delete button
- **From the detail screen:** Tap the trash icon in the top-right corner
- Deletion is immediate and cannot be undone

---

**User Story 1.4 — Edit a task**  
*As a user, I want to update a task's details as things change.*

- Tap any task row to open the task detail screen
- Edit the title, priority, or due date inline
- Changes are saved automatically on exit

---

### EPIC 2 — Priority and Organization

**Goal:** Help users focus on what matters by organizing tasks into clear tiers.

---

**User Story 2.1 — View tasks by priority**  
*As a user, I want to see my tasks grouped by priority so I know what to work on first.*

- The Lists screen groups tasks under **P1**, **P2**, and **P3** section headers
- Each section header shows the count of tasks in that group
- Sections are collapsible — tap a header to expand or collapse it

---

**User Story 2.2 — Reorder tasks by dragging**  
*As a user, I want to drag tasks between priority groups so I can re-rank them quickly.*

- Long-press a task row until it lifts
- Drag it into a different priority section
- Release to drop — the task's priority updates and is saved

---

### EPIC 3 — Time-Based Views

**Goal:** Surface the right tasks at the right time.

---

**User Story 3.1 — Filter tasks by time frame**  
*As a user, I want to see only the tasks relevant to a specific time window.*

| View | Shows |
|------|-------|
| Inbox | All tasks regardless of due date |
| Today | Tasks with a due date of today |
| This Week | Tasks due within the current week |
| This Month | Tasks due within the current month |

- Tap the view name in the tab bar at the top of the Lists screen to switch between views

---

**User Story 3.2 — Sort tasks**  
*As a user, I want to sort tasks in my preferred order.*

- Tap the sort icon on the Lists screen toolbar
- Available sort options:
  - Created date (newest first)
  - Priority (P1 → P3)
  - Due date (soonest first)
  - Alphabetical (A → Z)

---

### EPIC 4 — Task Enrichment

**Goal:** Let users add context and visual cues to their tasks.

---

**User Story 4.1 — Add a status emoji**  
*As a user, I want to add an emoji to a task so I can see its status at a glance.*

- Open a task detail screen
- Tap the emoji area to open the picker
- Choose from 4 quick-pick presets or enter any custom emoji
- The emoji appears on the task row in the list

---

**User Story 4.2 — Attach photos**  
*As a user, I want to attach photos to a task so I can store visual context.*

- Open a task detail screen
- Tap **Take Photo** to use the camera, or **Choose from Gallery** to select from the device's photo library
- Photos appear in a scrollable row at the bottom of the detail screen
- Tap a photo to open a full-screen viewer with pinch-to-zoom

---

### EPIC 5 — Notifications

**Goal:** Remind users about tasks before they miss a deadline.

---

**User Story 5.1 — Receive a reminder for a due task**  
*As a user, I want a notification on the day a task is due so I don't forget it.*

- When a task is saved with a due date, a local notification is scheduled for **12:00 PM** on that date
- The notification appears in the device notification tray even if the app is closed
- If the due date is changed, the notification is automatically rescheduled
- Grant notification permissions when prompted on first launch

---

### EPIC 6 — Appearance

**Goal:** Make the app comfortable to use in all lighting conditions.

---

**User Story 6.1 — Use dark mode**  
*As a user, I want a dark mode so the app is easier on my eyes at night.*

- Open the **Settings** tab
- Toggle **Dark Mode** on or off
- The app instantly switches between light and dark themes
- The preference is saved and remembered the next time the app opens

---

### EPIC 7 — Account (Optional)

**Goal:** Allow users to connect a Google account for future sync features.

---

**User Story 7.1 — Sign in with Google**  
*As a user, I want to sign in with Google so my tasks can sync in the future.*

- Open the **Settings** tab
- Tap **Sign in with Google**
- Complete the standard Google sign-in flow in the browser
- Your name and profile photo appear in Settings once signed in
- Tap **Sign Out** to disconnect the account
- The app works identically with or without a Google account

---

## 3. Inputs and Outputs

### 3.1 Inputs

| Input | Screen | Type | Required |
|-------|--------|------|----------|
| Task title | New task modal / Task detail | Text field | Yes |
| Priority | New task modal / Task detail | Segmented picker (P1 / P2 / P3) | No (defaults to P3) |
| Due date | New task modal / Task detail | Date picker | No |
| List name | New task modal | Text field | No (defaults to Inbox) |
| Status emoji | Task detail | Emoji picker / text input | No |
| Photos | Task detail | Camera or gallery picker | No |
| Sort preference | Lists screen | Dropdown picker | No (defaults to created date) |
| List view filter | Lists screen | Tab selector | No (defaults to Inbox) |
| Dark mode toggle | Settings screen | Switch | No |
| Google account | Settings screen | OAuth sign-in | No |
| Task completion | Lists screen | Checkbox tap | — |
| Task reorder | Lists screen | Long-press drag | — |
| Task delete (swipe) | Lists screen | Left swipe gesture | — |
| Task delete (button) | Task detail | Trash icon tap | — |

### 3.2 Outputs

| Output | Where it appears | Description |
|--------|-----------------|-------------|
| Task list | Lists screen | Tasks grouped by priority and filtered by view |
| Section task count | Priority section headers | Number of tasks per priority group |
| Completed section | Bottom of Lists screen | Collapsible list of finished tasks |
| Task row | Lists screen | Shows title, priority badge, due date, and status emoji |
| Task detail view | Task detail screen | All fields, photos, and action buttons |
| Photo viewer | Full screen overlay | Full-screen photo with pinch-to-zoom |
| Push notification | Device notification tray | Task reminder at 12:00 PM on due date |
| User profile | Settings screen | Google avatar and display name when signed in |
| Dark / light theme | App-wide | Entire app re-themes instantly on toggle |
| Persisted data | On-device storage | All tasks survive app restarts and device reboots |

---

## 4. Services and Features

### 4.1 Local Storage

All tasks are stored on the device in **AsyncStorage** under the key `@tasks_storage` as a JSON array. This is the primary and authoritative datastore. No internet connection is required. Tasks are loaded instantly without any network delay. Theme preferences are stored separately and persist independently.

### 4.2 Cloud Backend (Supabase)

Supabase provides a **PostgreSQL database**, authentication, and file storage as the cloud layer for future cross-device sync. Row Level Security (RLS) enforces that each user can only access their own data. The cloud backend is initialized and ready, but the app falls back gracefully to local storage for users who are not signed in. The Supabase client is configured in [lib/supabase.ts](../lib/supabase.ts).

### 4.3 Google Authentication

Google Sign-In is handled by `@react-native-google-signin/google-signin`. It supports sign-in, sign-out, and fetching the current user profile (name, email, avatar). Authentication is surfaced on the Settings screen and is entirely optional. Helper functions are in [lib/googleAuth.ts](../lib/googleAuth.ts).

### 4.4 Push Notifications

Local push notifications are scheduled through `expo-notifications`. When a user sets a due date on a task, the app schedules a notification for **12:00 PM** on that date. The notification is cancelled and rescheduled whenever the due date is updated. On Android, a high-importance notification channel is created on first launch. Notification logic is in [lib/notifications.ts](../lib/notifications.ts).

### 4.5 Drag-to-Reorder

Powered by `react-native-draggable-flatlist`, users can long-press any task row and drag it to a new position — including into a different priority group. The updated order is persisted to AsyncStorage immediately.

### 4.6 Swipe-to-Delete

Implemented with `react-native-gesture-handler`. Swiping a task row to the left reveals a red **Delete** button. Confirming deletion removes the task from AsyncStorage immediately. This gesture works across all list views.

### 4.7 Photo Attachments

Users can attach an unlimited number of photos to any task. Photos are captured via the in-app camera (`expo-camera`) or selected from the device gallery (`expo-image-picker`). Photos are stored as local file URIs on the device. The task detail screen displays photos in a horizontal scrollable row. Tapping a photo opens a full-screen pinch-to-zoom viewer.

### 4.8 NLP Task Parser

A lightweight natural language processing module (`lib/nlp.ts`) can extract structured task data from free-form text:

- **Priority** — from words like "urgent", "high priority", "P1", "low"
- **Due dates** — from phrases like "today", "tomorrow", "next week"
- **Times** — from patterns like "3pm", "10:30 AM"

This module is implemented and tested but is not yet connected to the main task creation UI. It is available for integration in a future release.

### 4.9 Theme System

A React Context ([contexts/ThemeContext.tsx](../contexts/ThemeContext.tsx)) manages the app-wide color scheme. Themes define:

| Token | Light | Dark |
|-------|-------|------|
| Background | White (`#FFFFFF`) | Black (`#000000`) |
| Surface | Light gray | Dark gray |
| Text | Dark | Light |
| Primary | Blue (`#2196F3`) | Blue (`#2196F3`) |
| Secondary | Green (`#4CAF50`) | Green (`#4CAF50`) |

On first launch, the theme defaults to the device's system color scheme. The user's manual choice overrides this and is persisted via AsyncStorage.

---

*TaskAces v1.0.5 — User Guide — April 2026*
