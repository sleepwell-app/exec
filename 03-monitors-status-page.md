# 03 – Public Monitors Status Page

## What Was Built

Transformed the status-page from a B2B2C authenticated customer portal into a **public, read-only status page** showing all monitors for a project with a 30-day uptime history.

---

## Backend

### New: `MonitorsStatusService`
**File:** `backend/src/modules/monitors/services/monitors-status.service.ts`

Computes per-monitor uptime timelines for a given project:

1. Fetches all monitors for the project.
2. For each monitor fetches the latest log (current status) and all logs overlapping the time window `[now - days*24h, now]`.
3. Builds N daily buckets. For each bucket, overlapping logs are found (`startedAt < bucketEnd && endedAt >= bucketStart`) and OK milliseconds are summed (clamped to bucket bounds). `uptimePercent = okMs / bucketMs * 100`. Buckets with no logs get `null`.
4. Overall `uptimePercent` is the mean of non-null buckets, rounded to 1 decimal.

### New: `StatusPageMonitorsController`
**File:** `backend/src/modules/monitors/status-page-monitors.controller.ts`

```
GET /status-page/monitors?days=30
  Headers: x-project-id: <projectId>
  Auth: none (@Public() decorator bypasses BetterAuthGuard)
```

### Modified: `monitors.module.ts`
Registered `StatusPageMonitorsController` and `MonitorsStatusService`.

---

## Example API Response

```json
[
  {
    "id": 1,
    "name": "API",
    "url": "https://api.example.com/health",
    "isActive": true,
    "latestStatus": "ok",
    "uptimePercent": 99.8,
    "timeline": [
      { "date": "2026-01-30", "uptimePercent": 100 },
      { "date": "2026-01-31", "uptimePercent": 98.5 },
      ...
      { "date": "2026-02-28", "uptimePercent": null }
    ]
  }
]
```

---

## Frontend

### New: `status-page/src/api/monitorRequests.ts`
Axios call to `GET /status-page/monitors?days=30`. Exports `MonitorStatus` and `DayBucket` TypeScript interfaces. `x-project-id` injected by existing axios interceptor.

### New: `status-page/src/api/hooks/useMonitors.ts`
React Query hook `useMonitorsStatus(days = 30)` with `staleTime: 60s`, `refetchInterval: 60s`, `retry: 2`.

### Replaced: `status-page/src/components/pages/Dashboard.tsx`
Public status page UI:
- Centered `max-w-3xl` container.
- Header with project title and "All systems operational" / "Issues detected" banner.
- Per-monitor row: status dot, name, URL link, status badge, uptime %, 30-column CSS Grid timeline bars.
- Bar colors: green (≥99%), amber (75–98%), red (<75%), gray (no data).
- Legend at bottom.

### Simplified: `status-page/src/App.tsx`
- Removed all auth logic: `useInitializeAuth`, `useAuthStore`, `handleSignOut`, `handleLogout`, user dropdown, sign-in button, sidebar, `isAuthLoading` screen, `AuthGuard`, `SignIn`/`VerifyEmail` routes.
- Kept all multi-tenant project-loading logic unchanged.
- `document.title` set to `${title} Status`.
- Simplified top bar: project title + `ThemeToggle` only.
- Routes: `/` and `/dashboard` both render `<Dashboard />`, `*` renders `<PageNotFound />`.

---

## Unused Auth Files

The following files remain but are no longer used by the simplified App. They can be cleaned up in a future pass:
- `src/hooks/useInitializeAuth.ts`
- `src/store/useAuthStore.ts`
- `src/hooks/useAuthModal.ts`
- `src/api/authRequests.ts`
- `src/lib/auth-client.ts`
- `src/components/AuthGuard.tsx`
- `src/components/SignInForm.tsx`
- `src/components/google-sso/`
- `src/components/pages/SignIn.tsx`
- `src/components/pages/VerifyEmail.tsx`
