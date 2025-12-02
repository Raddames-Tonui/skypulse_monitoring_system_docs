---
sidebar_position: 3
---

# SSL Expiry Monitor Task

## Overview

The `SslExpiryMonitorTask` is a scheduled background task in the SkyPulse Monitoring System responsible for monitoring the SSL certificates of active services and generating alerts before certificates expire. It uses multi-threading to check multiple services in parallel and retries failed checks according to system configuration.

## Configuration

The task reads configuration values from `SystemSettings.SystemDefaults`:

* **sslCheckIntervalSeconds**: Interval between SSL checks (default fallback: 6 hours)
* **alertThresholds**: Days before expiry to trigger alerts (default: 30, 14, 7, 3)
* **sslRetryCount**: Number of retries for failed SSL checks
* **sslRetryDelaySeconds**: Delay between retries

## Execution Flow

```
Scheduled Task Trigger
        |
        v
+---------------------+
|  Load Active Services | <-- Query DB for SSL-enabled, active services
+---------------------+
        |
        v
+---------------------+
|  Load Retry Services  | <-- Retry services with previous failures
+---------------------+
        |
        v
+---------------------+
|   Thread Pool (5)   | <-- Parallel SSL checks
+---------------------+
        |
        v
+---------------------+
|  Check SSL Certificate | <-- Extract cert info, calculate days remaining
+---------------------+
        |
        v
+---------------------+
|   Upsert SSL Logs    | <-- Insert or update `ssl_logs` table
+---------------------+
        |
        v
+---------------------+
|   Generate Alerts    | <-- Check thresholds, insert into `ssl_alerts` & `event_outbox`
+---------------------+
        |
        v
 Task Completed
```

## Database Interaction

* **Tables Updated**:

  * `monitored_services`: Reads services to monitor
  * `ssl_logs`: Stores certificate info and days remaining
  * `ssl_alerts`: Tracks sent alerts per threshold
  * `event_outbox`: Queues SSL expiry events for notification

* **Upsert Logic**:

  * If service+domain exists in `ssl_logs`, update it; otherwise, insert new record
  * Failed checks increment `retry_count` and mark `days_remaining` as `-1`

## Certificate Handling

* Extracts host from service URL
* Retrieves SSL certificate using `SslUtils.getServerCert`
* Extracts certificate details:

  * Issuer
  * Serial Number
  * Signature Algorithm
  * Public Key Algorithm & Length
  * Subject Alternative Names (SANs)
  * Chain Validity
  * Subject
  * Fingerprint
  * Issued & Expiry Dates
* Calculates days remaining until expiry

## Alert Generation

* Compares `days_remaining` with configured thresholds
* Checks if alert for threshold already sent
* Inserts alert into `ssl_alerts` table if not already sent
* Creates a corresponding event in `event_outbox` for notifications

## Logging

* Uses SLF4J for logging:

  * Initialization
  * SSL check start & completion
  * Failures and retries
  * Alert creation

## Concurrency

* Uses a fixed thread pool (`THREADS = 5`) for parallel SSL checks
* Each service check runs in its own thread
* Futures ensure completion and error handling for each service check

## Retry Mechanism

* Services that previously failed SSL checks (`days_remaining = -1`) are retried up to `sslRetryCount`
* Only retried if last check was older than `sslRetryDelaySeconds`
* Failed retry attempts are logged and increment `retry_count`

## Example Config (SystemDefaults)

```json
{
  "sslCheckInterval": 21600,
  "sslAlertThresholds": [30, 14, 7, 3],
  "uptimeRetryCount": 3,
  "uptimeRetryDelay": 300
}
```

## Summary

The `SslExpiryMonitorTask` ensures SSL certificate validity is continuously monitored, alerts are generated before expiry, and failures are retried automatically. It integrates tightly with the database and event system for seamless notifications and logging.
