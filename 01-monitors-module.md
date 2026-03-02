# Monitors Module — Build Notes

## What Was Built

### Entities
- `src/modules/monitors/entities/monitor.entity.ts` — Monitor configuration per project. Fields: id, projectId, name, url, frequencyMinutes (default 5), isActive (default true), lastCheckedAt (nullable), createdAt, updatedAt. Index on (isActive, lastCheckedAt).
- `src/modules/monitors/entities/monitor-log.entity.ts` — Smart-merged check result. Fields: id, monitorId, responseCode (nullable int), status (enum: ok/error), responseTimeMs (nullable int), startedAt, endedAt. Index on (monitorId, startedAt DESC).

### Services
- `src/modules/monitors/services/monitors.service.ts` — CRUD + paginated log listing + ownership validation via project join.
- `src/modules/monitors/services/monitor-check.service.ts` — HTTP check via axios + smart log merge algorithm + email alert dispatch.

### Processor
- `src/modules/monitors/processors/monitor.processor.ts` — `@Processor('monitors')` handles `check-monitor` jobs, delegates to MonitorCheckService.

### Controller
- `src/modules/monitors/monitors.controller.ts` — REST endpoints under `/monitors`, guarded by `IsBusinessUserGuard`. Project ID via `x-project-id` header.

### Module
- `src/modules/monitors/monitors.module.ts` — Wires TypeORM (Monitor, MonitorLog, Project, User) + Bull (monitors, emails) + providers + controller.

### DTOs
- `create-monitor.dto.ts` — { name, url, frequencyMinutes? }
- `update-monitor.dto.ts` — { name?, url?, frequencyMinutes?, isActive? }
- `monitor-log.dto.ts` — Response shape for log rows

### Migrations
- `1767200000000-CreateMonitorsTable.ts` — Creates `monitors` table with FK to projects + index.
- `1767200000001-CreateMonitorLogsTable.ts` — Creates `monitor_logs` table with enum type, FK to monitors + index.

### Email Templates
- `src/modules/mail/templates/monitor-down.hbs` — Alert for status OK → ERROR transition.
- `src/modules/mail/templates/monitor-recovery.hbs` — Alert for status ERROR → OK transition.

## Modified Files
- `src/modules/bull/bull.module.ts` — Added `{ name: 'monitors' }` to registerQueue and BullBoardModule.forFeature.
- `src/modules/app/app.module.ts` — Imported MonitorsModule; added Monitor to TypeOrmModule.forFeature.
- `src/jobs/cron.service.ts` — Injected monitorsQueue + monitorsRepo; added `@Cron(EVERY_MINUTE)` scheduleMonitorChecks().

## Smart Log Algorithm
1. Find latest log for monitor (`ORDER BY startedAt DESC LIMIT 1`)
2. No row → create first row with startedAt = endedAt = now
3. Same status → update endedAt = now, refresh responseCode + responseTimeMs
4. Different status → new row (startedAt = endedAt = now); queue notification email

## Still To Do
- **Frontend integration**: Admin-portal UI for monitor CRUD + log listing
- **Status-page display**: Show uptime/status indicators to end users
- **Feature gating**: Enforce `max_monitors` and `check_interval_minutes` plan features
- **Public status page**: Aggregate monitor statuses per project for public-facing page
- **Incident history**: Summarised downtime windows on the status page

## API Endpoints
| Method | Path | Description |
|--------|------|-------------|
| POST | /monitors | Create monitor (x-project-id header) |
| GET | /monitors | List monitors for project |
| GET | /monitors/:id | Get single monitor |
| PUT | /monitors/:id | Update monitor |
| DELETE | /monitors/:id | Delete monitor |
| GET | /monitors/:id/logs | Paginated logs (?page=1&limit=20) |
