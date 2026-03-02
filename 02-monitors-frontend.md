# Monitors Frontend Implementation

## Overview
Added a full CRUD Monitors page to the admin-portal, scoped per-project via the existing Axios `X-Project-Id` interceptor.

---

## New Files

### `admin-portal/src/api/monitorRequests.ts`
Axios request helpers for the monitors backend module:
- `getMonitors()` — `GET /monitors`
- `createMonitor(data)` — `POST /monitors`
- `updateMonitor(id, data)` — `PUT /monitors/:id`
- `deleteMonitor(id)` — `DELETE /monitors/:id`
- `getLogs(id, page, limit)` — `GET /monitors/:id/logs?page=&limit=`

### `admin-portal/src/components/monitors/MonitorFormModal.tsx`
`SaModal`-based create/edit form.

**Props:**
| Prop | Type | Description |
|------|------|-------------|
| `open` | `boolean` | Controls modal visibility |
| `onClose` | `() => void` | Close handler |
| `monitor` | `any \| null` | `null` = create mode, object = edit mode |
| `onSaved` | `() => void` | Callback to refresh parent list |

**Fields:** Name (required), URL (required), Frequency in minutes (1–1440, default 5), Active switch (edit mode only).

### `admin-portal/src/components/monitors/MonitorLogsModal.tsx`
`SaModal`-based paginated log viewer using `SaDataTable` with `serverSide={true}`.

**Props:**
| Prop | Type | Description |
|------|------|-------------|
| `open` | `boolean` | Controls modal visibility |
| `onClose` | `() => void` | Close handler |
| `monitor` | `any \| null` | Monitor whose logs to display |

**Columns:** Status (Chip), Response Code, Response Time (ms), Started At (datetime), Duration (endedAt − startedAt in seconds).

**Pagination:** local `page` (0-indexed for DataGrid, +1 for API), `pageSize` default 20.

### `admin-portal/src/pages/Monitors.tsx`
Main monitors list page.

- Fetches all monitors on mount (`GET /monitors` returns flat array with `latestLog` embedded)
- `SaTable` (non-paginated) with columns: Name, URL (tooltip on truncation), Frequency, Status chip, Last Checked, Active switch (inline toggle), Actions (edit/logs/delete)
- Add Monitor button opens `MonitorFormModal` in create mode
- Edit icon opens `MonitorFormModal` in edit mode
- Logs icon opens `MonitorLogsModal`
- Delete icon opens `SaConfirmDialog`, confirms then calls `deleteMonitor`
- Empty state shown when no monitors exist

---

## Modified Files

### `admin-portal/src/components/Layout.tsx`
- Added `SignalCellularAltIcon` import from `@mui/icons-material`
- Added `{ label: 'Monitors', path: '/monitors', icon: SignalCellularAltIcon }` to user menu after Dashboard

### `admin-portal/src/App.tsx`
- Added `const Monitors = React.lazy(() => import('./pages/Monitors'))`
- Added `<Route path="/monitors" element={<UserRoute><Monitors /></UserRoute>} />`

### `admin-portal/src/config/pageMetadata.ts`
- Added `/monitors` entry: title "Monitors", description "Manage uptime monitors for your project"

---

## Still To Do
- **Status-page display**: Surface monitor status (uptime percentage, incident history) on the customer-facing status page
- **Feature gating**: Optionally gate the Monitors page/feature behind a subscription plan feature flag
- **Alert notifications**: Email/Pusher alerts when a monitor transitions to `error` state
