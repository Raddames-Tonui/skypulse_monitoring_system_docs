# Logging and Audit

## Overview

The Skypulse system implements a comprehensive logging and audit framework to track user actions, system errors, and runtime events. Logging leverages **SLF4J** with **Logback**, while user actions are captured in the `audit_log` table for traceability and compliance.

## Components

### 1. AuditLogger

Responsible for logging user actions:

* Captures CRUD operations and rollbacks.
* Logs both `beforeData` and `afterData` as JSON.
* Associates each log with a user, action, entity, and source IP.

**Usage:**

```java
AuditLogger.log(exchange, "services", serviceId, "UPDATE", beforeService, updatedService);
```

**Notes:**

* Requires a valid `HttpServerExchange` and attached `UserContext`.
* Failures in logging are captured in SLF4J without disrupting the main flow.

### 2. User Login Logging

`UserLoginHandler` logs login attempts:

* Successful login: generates JWT, refresh tokens, sets cookies.
* Failed login: recorded in `login_failures` table with email, IP, user-agent, and reason.

**Notes:**

* Ensures failed login attempts are traceable for security monitoring.
* Works alongside `AuditLogger` for full user activity tracking.

### 3. SLF4J + Logback Logging

Centralized logging configuration:

* **Console logging**: for development/debug.
* **File logging**: rolling daily logs (`skypulse.log`) with 7-day retention.
* **Error-specific logging**: separate `errors.log` with 30-day retention.
* **Async logging**: improves performance by decoupling logging from request processing.



### 4. KeyProvider & Environment-based Logging

* Loads the correct logback configuration based on environment (`DEVELOPMENT` or `PRODUCTION`).
* Ensures secrets and environment variables are securely loaded before initializing logging.
* Automatically switches logging verbosity (DEBUG for dev, INFO/WARN for production).

### 5. Logging Rules & Best Practices

| Action                 | Rule                                                      |
| ---------------------- | --------------------------------------------------------- |
| Start of any operation | `LogContext.start("ComponentName")`                       |
| End of operation       | `LogContext.clear()` in finally block                     |
| Async jobs             | Call `LogContext.start()` inside runnable/async job       |
| Error logging          | Use `logger.error()` with exceptions included             |
| Audit logging          | Always log user actions to `audit_log` with JSON payloads |

## Tables

* **audit_log**: captures user actions, before/after state, IP, timestamps.
* **login_failures**: captures failed login attempts, IP, user-agent, and reason.
* **background_tasks**: logs scheduled task executions and errors.

## Best Practices

* Always include `trace.id` to correlate logs across services.
* Separate error logs from normal logs for quick monitoring.
* Audit all critical user actions to maintain accountability.
* Use async logging to reduce blocking during high-load operations.

## Example Integration

```java
// Example user action logging
AuditLogger.log(exchange, "users", userId, "UPDATE", oldUser, updatedUser);

// Logging with context
LogContext.start("UserLogin");
try {
    // handle login
} finally {
    LogContext.clear();
}
```

## Conclusion

Skypulse logging framework provides a robust, scalable, and secure method to capture system events, errors, and user activities. The combination of SLF4J, Logback, MDC, and audit logging ensures traceability, operational insight, and security compliance.
