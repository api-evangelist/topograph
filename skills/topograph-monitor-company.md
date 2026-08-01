---
name: Monitor a company for register changes
description: Start continuous monitoring on a company and receive real-time monitor.notification webhooks when meaningful changes are detected.
api: openapi/topograph-openapi-original.json
operations:
  - MonitoringController_createMonitor_v2
  - MonitoringController_listMonitors_v2
  - MonitoringController_getMonitorLogs_v2
  - MonitoringController_stopMonitoring_v2
---

# Monitor a company for register changes

Track legal-lifecycle and other meaningful changes to a company; Topograph detects and categorizes
changes with AI and pushes webhook notifications.

## Auth
`x-api-key` header; base URL `https://api.topograph.co`. Webhook endpoints are configured once at
the **account** level (all monitors share them).

## Steps

1. **Start monitoring.** Call `POST /v2/monitors` (`MonitoringController_createMonitor_v2`) with
   `companyId`, `countryCode`, and an optional `metadata` object (e.g. `{ "caseId": "case-123" }`,
   max 50 keys). Keep the returned `id`. Re-calling for the same company replaces the stored
   config/metadata (idempotent upsert).

2. **List / audit.** `GET /v2/monitors` (`MonitoringController_listMonitors_v2`) lists monitored
   companies (cursor paginate with `after`); `GET /v2/monitors/{id}/logs`
   (`MonitoringController_getMonitorLogs_v2`) returns the change log for one monitor.

3. **Handle webhooks.** On a change you receive a `monitor.notification` event containing `type`,
   `monitorId` (matches the `id` from step 1 — use it to order and deduplicate), `companyId`, your
   `metadata`, and `monitorHasBeenDeactivated`. If `monitorHasBeenDeactivated` is true, treat it as
   the final notification (e.g. company no longer found) and clean up. Categories include `status`
   (bankruptcy, liquidation, dissolution).

4. **Stop monitoring.** `DELETE /v2/monitors/{id}` (`MonitoringController_stopMonitoring_v2`) →
   `204`.

## Errors
`401` invalid/missing API key; `404` monitor not found or not owned by this account; `400`
monitoring not supported for the company. Webhook catalog:
`asyncapi/topograph-monitoring-webhooks.yml`.
