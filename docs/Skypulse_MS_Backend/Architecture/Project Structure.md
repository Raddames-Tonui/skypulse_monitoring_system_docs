---
sidebar_position: 2
---


# Project Structure

This document provides a structured, high‑level explanation of the SkyPulse backend architecture based on the provided package layout and representative code samples. It outlines the responsibilities of each module and how the system components cooperate to deliver a fully modular, secure, and scalable backend.



## 1. Project Structure

```
java
    |__org.skypulse
    ├── config
    ├── handlers
    ├── notifications
    ├── rest
    ├── services
    │   └── tasks
    └── utils

resources
    ├── images
    ├── templates
    │   ├── emails
    │   │   └── templates
    │   └── pdf
    │       └── templates
    └── logs
        └── templates
```

---

## 2. Module Breakdown

### **2.1 `config` – Configuration, Encryption, and Database Setup**

This module centralizes runtime configuration, secure handling of secrets, and database connectivity.

**Key Responsibilities:**

* Parsing XML configuration files.
* Encrypting/decrypting sensitive fields (e.g., DB passwords) using AES‑GCM.
* Initializing & maintaining the HikariCP connection pool.
* Providing environment‑driven master keys and allowed origins.

**Representative Code:**

* `DatabaseManager` – robust pooled connection creation with SSL support.
* `ConfigEncryptor` – in‑place encryption for config.xml and secure decryption in memory.



### **2.2 `handlers` – Request Processing Logic (Controllers)**

This layer handles HTTP requests, validates data, performs DB operations, and generates responses.

**Sub‑categories:**

* `auth` – login, logout, session renewal, password resets.
* `users`, `services`, `contacts`, `reports`, `logs`, … – business‑specific handlers.

Handlers typically:

1. Parse incoming requests.
2. Validate and sanitize input.
3. Query/update database.
4. Return JSON responses.

**Representative Code:**

* `AuthMiddleware` – cookie‑based JWT authentication, auto‑refresh using DB‑backed sessions.
* CRUD‑style handlers for services, users, logs, etc.



### **2.3 `notifications` – Multi‑Channel Notifications Engine**

Manages sending messages over various channels without tying business logic to any single service.

**Key Components:**

* `NotificationSender` – a pluggable interface.
* `MultiChannelSender` – routing layer supporting EMAIL/SMS/TELEGRAM.
* Email sender implementations (with inline‑image support).

This design keeps communication channels isolated and easily extendable.

### **2.4 `rest` – API Server & Routing Layer**

Responsible for wiring all handlers into the Undertow web server.

**Key Responsibilities:**

* HTTP server initialization.
* URL routing & grouping.
* CORS enforcement.
* SSE (Server Sent Events) endpoints.

**Representative Code:**

* `RestApiServer` – bootstraps Undertow, configures threads and listener.
* `Routes` – declares REST routes using prefix grouping.

Example route grouping:

* `/auth` → authentication flow.
* `/services` → monitored services API.
* `/sse` → real‑time uptime/downtime streams.



### **2.5 `services` – Background Jobs and Business Services**

Encapsulates non‑HTTP business logic, scheduled tasks, monitors, and async processes.

Examples:

* Uptime checker.
* SSL certificate monitor.
* Notification processor.
* Housekeeping / log cleanup tasks.

`services/tasks` holds schedulable or asynchronous units.



### **2.6 `utils` – Security, Helpers, Common Tools**

A shared collection of helpers used across the system.

Key areas:

* JWT generation / validation.
* Token hashing and cookie utilities.
* Response helpers.
* Time/date utilities.
* Logging helpers.

---

## 3. `resources` Structure

Resources contain static and template‑driven assets used by the backend.

### **Images**

Inline resources used in email templates or reports.

### **Templates → Emails / PDF**

* Email HTML templates for verification, reset-password, notifications.
* PDF templates for uptime and SSL reports.

### **Logs → Templates**

The system provides two streamlined logging template categories:

**Development Templates**

* High-detail logs for debugging.
* Includes debug messages, warnings, errors, and diagnostic data.
* Ideal for active development and troubleshooting.

**Production Templates**

* Clean, controlled output.
* Includes only info, warnings, and errors.
* Uses rolling files and error-specific logs for stability and audit clarity.

These templates align with the corresponding Logback configurations for each environment.


---

## 4. Architectural Flow Summary

### **4.1 Startup Phase**

1. Load & decrypt config.xml.
2. Initialize database pool.
3. Start Undertow with configured routes.
4. Load notification channels.
5. Initialize schedulers for uptime & SSL checks.

### **4.2 Request Lifecycle**

1. Request hits Undertow.
2. CORS validated.
3. If protected route → `AuthMiddleware` checks cookies & session.
4. Routed to appropriate handler.
5. Handler executes DB logic, responds with JSON.

### **4.3 Background Tasks**

* Regular checks for service uptime.
* SSL certificate expiry scanning.
* Generating PDF reports.
* Dispatching notifications.

---

## 5. Why This Architecture Works

* **Strong modularity** – clean separation of config, HTTP logic, business logic, and background tasks.
* **Security‑first** – encrypted configs, cookie‑based JWTs, DB‑backed sessions, secure crypto.
* **Performance‑oriented** – Undertow + HikariCP + async tasks.
* **Extendable** – plug‑and‑play notification channels, easily add new handlers.
* **Production‑ready** – proper thread control, structured templates, monitoring tasks.

