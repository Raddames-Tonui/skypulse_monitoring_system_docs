---
sidebar_position: 5
---

# NotificationProcessorTask

## Overview

`NotificationProcessorTask` is a scheduled background task that processes pending events from the `event_outbox` table and sends notifications to recipients via configured channels (email, etc.). It supports retries, cooldown periods, and logs the outcomes for auditing.

The task is designed for high throughput using a thread pool and respects template availability and user notification preferences.

## Key Components

* **NotificationSender**: Handles sending messages to recipients via different channels.
* **RecipientResolver**: Resolves recipients for a given event type and service.
* **TemplateLoader**: Loads email/message templates, supporting hybrid storage.
* **JdbcUtils**: Database utility for acquiring connections.
* **SystemDefaults**: Contains configurable defaults such as retry count, cooldown, and interval.
* **ScheduledExecutorService**: Handles concurrent execution of sending tasks.

## Task Configuration

```java
NotificationProcessorTask(NotificationSender sender, SystemDefaults systemDefaults, int workerThreads)
```

* `sender`: Notification sending implementation.
* `systemDefaults`: Default system settings (retry count, cooldown, interval).
* `workerThreads`: Number of concurrent threads for processing notifications.

## Execution Flow

```
+---------------------+
| Scheduled Task Run  |  <-- Interval from systemDefaults.notificationCheckInterval
+---------------------+
           |
           v
+---------------------+
| DB Connection       |  <-- Acquire connection for transaction
+---------------------+
           |
           v
+---------------------+
| Fetch Pending Events|  <-- SELECT * FROM event_outbox WHERE status = 'PENDING' FOR UPDATE SKIP LOCKED
+---------------------+
           |
           v
+---------------------+
| Parse Payload       |  <-- Convert JSON payload to Map
+---------------------+
           |
           v
+---------------------+
| Schedule Send Task  |  <-- Submit to ScheduledExecutorService
+---------------------+
           |
           v
+---------------------+
| Send Notifications  |  <-- Resolve recipients, fetch templates, send via sender
+---------------------+
           |
           v
+---------------------+
| Log & Update Status |  <-- Mark PROCESSED / FAILED / NO_TEMPLATE_FOUND
+---------------------+
```

## Database Interactions

### Tables Accessed:

* **event_outbox**: Holds pending notification events.
* **notification_templates**: Provides subject/body templates for events.
* **notification_channels**: Holds channel configurations (e.g., EMAIL).
* **notification_history**: Stores a log of sent notifications.

### Status Workflow:

| Status            | Description                    |
| ----------------- | ------------------------------ |
| PENDING           | Event is pending processing    |
| PROCESSED         | Notification successfully sent |
| NO_TEMPLATE_FOUND | No template exists for event   |
| FAILED            | Failed after retries           |

## Notification Processing Logic

1. Fetch up to 50 pending events with `FOR UPDATE SKIP LOCKED`.
2. Parse JSON payload.
3. Schedule sending task in thread pool.
4. Fetch template from DB; load with `TemplateLoader`.
5. Resolve recipients using `RecipientResolver`.
6. Attempt sending up to `retryCount` times with `retryDelay` seconds between attempts.
7. Log each attempt in `notification_history`.
8. Mark event status based on outcome.

## Template Rendering

Uses `Mustache` templates:

```java
String rendered = renderTemplate(template, templateKey, payload);
```

* Supports hybrid storage.
* Merges payload variables into template.

## Retry & Cooldown Logic

* `retryCount`: Number of times to retry sending per recipient.
* `retryDelaySeconds`: Wait time between retries.
* `cooldownMinutes`: Suppresses notifications for recently failed events to avoid spamming.

## Logging

* Uses SLF4J `Logger`.
* Logs task start, completion, errors, and each notification sent.
* Includes detailed info on failures and retries.

## Summary

`NotificationProcessorTask` provides a robust, concurrent, and reliable mechanism to process pending system events and send notifications. It ensures:

* Event processing is atomic and transactional.
* Notifications are only sent if templates exist.
* Failed sends are retried with cooldown.
* All actions are logged for auditing.
* Supports multiple channels and payload customization.
