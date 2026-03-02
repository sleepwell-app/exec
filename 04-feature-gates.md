# Feature Gates — Build Notes

## What Was Built

Wired up enforcement of subscription plan feature limits end-to-end across the backend (API guards) and frontend (UX gates), reading values dynamically from the plan feature system already present in the database. No hardcoded limits — all values are managed via the admin portal.

---

## Features Gated

| Feature Code | Enforced Where |
|---|---|
| `max_projects` | `POST /projects` + "New Project" UI |
| `max_monitors` | `POST /monitors` + "Add Monitor" UI |
| `check_interval_minutes` | `POST /monitors`, `PUT /monitors/:id` + frequency form field |
| `data_retention_days` | Daily cron job that purges old `monitor_logs` |

---

## Backend

### Modified: `projects.controller.ts`
- Injected `UserSubscriptionService`
- In `create()`: calls `getCurrentFeatures(user.id)` → counts existing projects via `getProjectsByUserId()` → throws `ForbiddenException` if `count >= max_projects`

### Modified: `monitors.service.ts`
- Added `countForUser(userId)` — counts monitors across all of a user's projects via an `INNER JOIN` on `project.userId`

### Modified: `monitors.controller.ts`
- Injected `UserSubscriptionService`
- Added `@CurrentUser()` to `create()`
- In `create()`: enforces `max_monitors` (403) and `check_interval_minutes` (400)
- In `update()`: validates `frequencyMinutes >= check_interval_minutes` when frequency is provided

### Modified: `monitors.module.ts`
- Imported `SubscriptionsModule` to make `UserSubscriptionService` available

### Modified: `projects.module.ts`
- Imported `SubscriptionsModule`

### Modified: `cron.service.ts`
- Injected `UserSubscriptionService` and `MonitorLog` repository
- Added `@Cron(EVERY_DAY_AT_2AM) purgeOldMonitorLogs()`:
  1. Fetches all distinct `userId` values that have at least one monitor
  2. For each user: reads `data_retention_days` from their plan features
  3. Deletes `monitor_logs` with `started_at < NOW() - INTERVAL 'N days'` scoped to that user's monitors
  4. Logs total deleted count

### Modified: `app.module.ts`
- Added `MonitorLog` to `TypeOrmModule.forFeature()` so it can be injected into `CronService`

---

## Frontend (admin-portal)

### Modified: `hooks/useFeatureChecker.ts`
- Added `MAX_MONITORS` and `CHECK_INTERVAL_MINUTES` to `FeatureCodes`
- Extended `FeatureCheckContext` with `monitors` and `frequencyMinutes` fields
- Added switch cases for both new feature codes

### Modified: `components/projects/ProjectsSelect.tsx`
- Calls `checkFeature(FeatureCodes.MAX_PROJECTS, { projects })` on render
- When at limit: "New Project" item shows a `WorkspacePremiumIcon` (crown) instead of `AddIcon` and navigates to `/subscription-plans` on click

### Modified: `pages/Monitors.tsx`
- Reads `max_monitors` and `check_interval_minutes` from Redux subscription features
- Derives `atMonitorLimit` flag
- "Add Monitor" button shows `WorkspacePremiumIcon` and navigates to `/subscription-plans` when at limit
- Passes `minFrequencyMinutes` prop to `MonitorFormModal`

### Modified: `components/monitors/MonitorFormModal.tsx`
- Accepts `minFrequencyMinutes` prop (default `1`)
- Frequency field `min` and `helperText` reflect the plan's minimum
- Validates `frequencyMinutes >= minFrequencyMinutes` on submit before calling the API

---

## How Limits Are Managed

Plan feature values are set and updated via the admin portal's Monetization → Plans screen. No code changes are required to adjust limits.
