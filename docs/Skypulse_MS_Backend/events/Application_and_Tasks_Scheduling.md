---
sidebar_position: 1
---

# ApplicationTasks and TaskScheduler 

## Overview

The `ApplicationTasks` class and the `TaskScheduler` manage the registration, scheduling, and execution of application-level tasks within the Skypulse system. These tasks include uptime checks, SSL monitoring, and notification processing.

This module ensures tasks run periodically, handle errors gracefully, and log execution history in the database.

## Key Classes

### ApplicationTasks

Responsible for initializing and registering system-level tasks:

* **init()**: Tracks initialization count.
* **registerApplicationTasks(boolean dbAvailable, XmlConfiguration cfg)**: Registers tasks dynamically if the database is available.

Registered tasks include:

1. **UptimeCheckTask**: Periodically checks the status of monitored services.
2. **NotificationProcessorTask**: Processes pending notifications and sends them via supported channels (Email, Telegram, etc.).
3. **SslExpiryMonitorTask**: Monitors SSL certificate expiry for registered services.

### TaskScheduler

Manages the periodic execution of `ScheduledTask`s.

* **register(ScheduledTask task)**: Registers a task to be scheduled.
* **start()**: Schedules all registered tasks using a thread pool executor.
* **shutdown()**: Cancels scheduled tasks and shuts down the executor.
* **reload()**: Reloads tasks, clearing existing ones and re-running the task loader.

#### Task Execution Flow

1. Tasks are scheduled at fixed intervals using `ScheduledThreadPoolExecutor`.
2. Each task execution is wrapped in logging, recording start and end times.
3. Execution results and errors are logged into the `background_tasks` table in the database.

## Features

* Dynamic task registration based on database availability.
* Concurrent execution with configurable thread pool size.
* Automatic logging of task results with timestamps.
* Graceful shutdown and reload of all tasks.
* Extensible for adding additional scheduled tasks.

## Usage Example

```java
// Initialize and register tasks
ApplicationTasks.registerApplicationTasks(dbAvailable, cfg);

// Start the scheduler
appScheduler.start();

// Reload tasks dynamically
appScheduler.reload();

// Shutdown scheduler
appScheduler.shutdown();
```

## Notes

* Ensure `SystemSettings` tables contain active services and default configuration.
* Use `ScheduledTask` interface for all tasks to be compatible with `TaskScheduler`.
* Task logging uses `background_tasks` table with upsert (`ON CONFLICT`) to track each task's last run and status.
* `NotificationProcessorTask` relies on `MultiChannelSender` to send notifications via multiple channels.
* `TaskScheduler` ensures tasks are safely executed even in case of individual task failure, with errors logged but not interrupting other tasks.
