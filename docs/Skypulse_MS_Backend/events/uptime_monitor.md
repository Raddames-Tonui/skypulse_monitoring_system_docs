---
sidebar_position: 2
---
# Uptime Check Task 

## Overview

`UptimeCheckTask` is a scheduled task that monitors the uptime and health of monitored services. It performs HTTP GET requests, tracks response times, records uptime logs, and triggers events when services go down or recover.

This task supports retries, configurable check intervals, and integrates with the database for state tracking and event publishing.

## Configuration

* **Service-specific settings** (from `SystemSettings.ServiceConfig`):

  * `checkInterval` (minutes)
  * `retryCount`
  * `retryDelay` (seconds)
  * `expectedStatusCode`
* **Default settings** (from `SystemSettings.SystemDefaults`) applied when service config is not provided.

## Execution Flow

```
Start Task -> Fetch Service Config
       |
       v
Perform HTTP Check -> Retry if failed -> Determine Status (UP/DOWN)
       |
       v
Update monitored_services table (status, consecutive failures, last checked)
       |
       v
Insert uptime_logs entry
       |
       v
Evaluate if event needs creation (DOWN or RECOVERED)
       |
       v
Insert into event_outbox if needed
       |
       v
Task Complete
```

## Key Methods

### `execute()`

* Opens a DB connection.
* Calls `performCheck()` within a transaction.
* Commits changes or logs errors.

### `performCheck(Connection conn)`

* Performs HTTP GET with retries.
* Determines service status (`UP` or `DOWN`).
* Updates `monitored_services` table.
* Inserts `uptime_logs` entry.
* Determines if event should be created.
* Logs summary.

### `createEvent(Connection conn, ...)`

* Creates an event in `event_outbox` for:

  * `SERVICE_DOWN` when consecutive failures equal retry count.
  * `SERVICE_RECOVERED` when service recovers.
* Payload includes service ID, name, old/new status, response time, HTTP code, retry info, and timestamp.

## Database Tables Affected

### `monitored_services`

* `last_uptime_status`
* `consecutive_failures`
* `last_checked`
* `date_modified`

### `uptime_logs`

* Tracks individual checks, including:

  * Service ID
  * Status
  * Response time (ms)
  * HTTP status
  * Error message
  * Timestamp

### `event_outbox`

* Stores events triggered by downtime or recovery:

  * `event_type` (SERVICE_DOWN / SERVICE_RECOVERED)
  * `payload` (JSON with detailed info)
  * `status` (PENDING)
  * `retries`
  * Timestamps

## Logging

Uses SLF4J `Logger`:

* Info: Summary of each check
* Error: Failures in HTTP request, DB operations, or event creation

## Concurrency & Timing

* HTTP requests are synchronous per service, but retries are handled within the loop.
* Timeout per request: 8 seconds
* Retry delay: configurable per service or default
* Check interval: configurable per service or default

## Summary

`UptimeCheckTask` ensures continuous monitoring of service availability, accurate logging, and timely alerting through events. It handles retries, tracks consecutive failures, and integrates seamlessly with the Skypulse monitoring system's DB and event mechanisms.
