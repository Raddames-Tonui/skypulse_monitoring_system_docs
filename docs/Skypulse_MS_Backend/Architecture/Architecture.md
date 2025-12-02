---
sidebar_position: 1
---

# Architecture Summary

SkyPulse Backend powers the monitoring infrastructure for uptime tracking, SSL expiry supervision, and event‑driven notifications. It is implemented in **Java**, backed by a **PostgreSQL** database, and deployed on **Render**.

## Overview

SkyPulse collects real‑time availability data from configured services, processes SSL certificate metadata, and streams insights to clients through **Server‑Sent Events (SSE)**. It incorporates scheduled background processes, secure authentication mechanisms, and a modular notification pipeline to keep stakeholders informed.


## Folder Structure

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
   |---reources 
        |---images
        |---templates
            | -- emails
                |--- templates
            | --- pdf
                |---templates
        |--logs templates
```

---

### Core Capabilities

* **Uptime Monitoring** using scheduled HTTP checks with retries.
* **SSL Certificate Monitoring** for expiry and trust‑chain validation.
* **Event Outbox Pattern** for reliable notification dispatch.
* **Server‑Sent Events** to emit live uptime/downtime snapshots.
* **REST Endpoints** for data access and configuration.
* **Cookie‑based Authentication** with HttpOnly and SameSite controls.
* **Configurable Email Notifications** using Mustache template rendering.


## Architecture Summary

The backend is composed of interconnected subsystems:

### 1. Uptime Check Process

Executes periodic HTTP requests to each monitored service, measuring:

* Response code
* Latency
* Error details
* Consecutive failures

Events are generated when:

* A service recovers from DOWN → UP
* A service crosses its failure threshold → DOWN

### 2. SSL Expiry Monitoring

Extracts certificate metadata from service domains, including:

* Issuer and subject
* SAN list
* Signature algorithm
* Public key details
* Expiry date and days remaining

*THRESHOLD‑based alerts* (30, 14, 7, 3 days, or configured) trigger event creation.

### 3. Notification Processor

Consumes events from the outbox table and resolves recipients. It renders templates and dispatches messages through the configured notification channel.

### 4. Real‑Time Streaming

SSE endpoints expose live updates on service health, enabling dashboards to update immediately without polling.

### 5. REST API Layer

* CRUD for monitored services
* Logs and statistics
* Settings and configuration
* Authentication endpoints



## Technologies

| Component          | Technology                  |
| ------------------ | --------------------------- |
| Language           | Java                        |
| Database           | PostgreSQL                  |
| Deployment         | Render                      |
| Data Serialization | Jackson (JSON)              |
| Templating         | Mustache                    |
| HTTP Client        | Java HttpClient             |
| Security           | Cookie‑based session tokens |
| Background Jobs    | Custom ScheduledTask system |


## Folder Structure




It stores:

* Status transitions
* Response metrics
* Error messages
* Failure counts



## Security

* HttpOnly, Secure, SameSite cookies
* Role‑based authorization
* Controlled CORS exposure for trusted frontends


