---
sidebar_position: 2
---


# Backend Setup Guide

This README provides instructions for setting up and running the **SkyPulse Monitoring System Backend**, built with **Java 21**, **Maven**, **Undertow**, **PostgreSQL**, **HikariCP**, and several utility libraries.

---

## Cloning the Repository

To get started, clone the backend from GitHub:

```sh
git clone git@github.com:Raddames-Tonui/skypulse_monitoring_system_backend.git
```

Then navigate into the project:

```sh
cd skypulse_monitoring_system_backend
```

You can use any Java‑supported IDE, such as **IntelliJ IDEA**, **VS Code (Java extensions)**, or **Eclipse**.

---

## Requirements

* **Java 21** installed
* **Maven 3.9+** installed
* **PostgreSQL database**
* (Optional) SMTP and Telegram credentials for notifications

---

## 1. Environment Variables (`.env`)

Create a `.env` file in the project root:

```env
CONFIG_MASTER_KEY=sampleMasterKey1234567890=
JWT_SIGNING_KEY=sampleJwtSigningKey123456=
APP_ENV=DEVELOPMENT
FRONTEND_BASE_URL=http://localhost:5173
FRONTEND_ALLOWED_ORIGINS=http://localhost:5173,https://example-frontend.app
```

###  Explanation of Variables

* **CONFIG_MASTER_KEY** — Used to encrypt and decrypt the XML configuration before booting the server.
* **JWT_SIGNING_KEY** — Secret used for signing access/refresh tokens.
* **APP_ENV** — `DEVELOPMENT` or `PRODUCTION` mode.
* **FRONTEND_BASE_URL** — Fallback URL mainly for development.
* **FRONTEND_ALLOWED_ORIGINS** — CORS configuration; must list all allowed frontend domains.

---

## 2. Main System Configuration (`config.xml`)

Below is an example with **sample credentials** only. Update these values with your own production/deployment credentials.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <server>
        <host mode="TEXT">0.0.0.0</host>
        <port mode="TEXT">8000</port>
        <ioThreads mode="TEXT">10</ioThreads>
        <workerThreads mode="TEXT">100</workerThreads>
        <basePath mode="TEXT">/api/rest</basePath>
    </server>

    <jwtConfig>
        <accessToken mode="TEXT">60</accessToken>
        <refreshToken mode="TEXT">30</refreshToken>
    </jwtConfig>

    <dataSource>
        <driverClassName mode="TEXT">org.postgresql.Driver</driverClassName>
        <jdbcUrl mode="TEXT">jdbc:postgresql://your-host/db_name?user=dbuser&amp;password=dbpassword&amp;sslmode=require</jdbcUrl>
        <username mode="TEXT">dbuser</username>
        <password mode="TEXT">samplePassword123!</password>
        <encrypt mode="TEXT">false</encrypt>
        <trustServerCertificate mode="TEXT">false</trustServerCertificate>
    </dataSource>

    <connectionPool>
        <maximumPoolSize mode="TEXT">10</maximumPoolSize>
        <minimumIdle mode="TEXT">3</minimumIdle>
        <idleTimeout mode="TEXT">60000</idleTimeout>
        <connectionTimeout mode="TEXT">30000</connectionTimeout>
        <maxLifetime mode="TEXT">1800000</maxLifetime>
    </connectionPool>

    <logging>
        <level mode="TEXT">INFO</level>
        <logFile mode="TEXT">logs/skypulse-db.log</logFile>
    </logging>

    <notification enabled="true">
        <threads>
            <workerThreads mode="TEXT">10</workerThreads>
        </threads>

        <email>
            <smtpHost>smtp.gmail.com</smtpHost>
            <smtpPort mode="TEXT">587</smtpPort>
            <useTLS mode="TEXT">true</useTLS>
            <username mode="TEXT">example.mailer@gmail.com</username>
            <password>app-password-sample</password>
            <fromName mode="TEXT">SkyPulse Monitor</fromName>
            <fromAddress mode="TEXT">example.mailer@gmail.com</fromAddress>
            <connectionTimeout mode="TEXT">10000</connectionTimeout>
            <retryAttempts mode="TEXT">3</retryAttempts>
        </email>

        <telegram enabled="true">
            <botToken mode="TEXT">123456789:sample-bot-token</botToken>
            <defaultChatId mode="TEXT">-1001234567890</defaultChatId>
            <apiUrl mode="TEXT">https://api.telegram.org</apiUrl>
            <parseMode>HTML</parseMode>
            <connectionTimeout mode="TEXT">5000</connectionTimeout>
        </telegram>

        <sms enabled="false">
            <provider mode="TEXT">twilio</provider>
            <accountSid mode="TEXT">ACxxxxxxx</accountSid>
            <authToken mode="TEXT">xxxxxxx</authToken>
            <senderNumber mode="TEXT">+12025550123</senderNumber>
            <defaultCountryCode mode="TEXT">+254</defaultCountryCode>
            <apiUrl mode="TEXT">https://api.twilio.com</apiUrl>
            <maxRetries mode="TEXT">2</maxRetries>
        </sms>
    </notification>
</configuration>
```

---

## 3. Maven Project Structure

This backend uses **Maven** for dependency management and packaging.

### Key Tech Used (from `pom.xml`)

* **Undertow** → Lightweight high‑performance HTTP server
* **HikariCP** → Fast database connection pooling
* **PostgreSQL JDBC** → Database connectivity
* **JAXB** → XML parsing
* **Jackson** → JSON serialization/deserialization
* **JFreeChart** → Generating charts
* **SLF4J + Logback** → Logging
* **java-dotenv** → Reads `.env` file
* **Email (Jakarta Mail)** → SMTP notifications
* **Template engines** → Thymeleaf, FreeMarker, Mustache
* **OpenHTMLToPDF** → PDF generation
* **BouncyCastle** → Cryptography support
* **jBCrypt** → Password hashing
* **JJWT** → Token generation

The build uses:

* **maven-compiler-plugin** → Compiles Java 21
* **maven-shade-plugin** → Packages everything into a fat JAR with main class `org.skypulse.Main`

---

##  4. Building the Backend

Run:

```sh
mvn clean package
```

This generates:

```
target/skypulse-monitoring-system.jar
```

---

## 5. Running the Backend

Ensure your `.env` and `config.xml` are properly configured, then run:

```sh
java -jar target/skypulse-monitoring-system.jar
```

---

## 6. Default API Route

All endpoints are served from:

```
http://localhost:8000/api/rest/
```

---

## 7. Important Security Notes

* Never commit real credentials.
* ALWAYS rotate JWT keys before production.
* Enable encryption for config files using `CONFIG_MASTER_KEY`.
* Use environment‑specific databases.

---

## Additional Notes

### Recommended IDEs

* IntelliJ IDEA (best support for Maven + Java)
* VS Code with Java extensions
* Eclipse IDE

### Project Folder Structure (Typical)

```
src/
 └── main/
     ├── java/               # Source code
     ├── resources/          # Templates, XML config, email templates
 └── test/                   # Unit tests
config.xml                   # System configuration
.env                         # Environment variables
pom.xml                      # Maven build file
```

### Troubleshooting

* Ensure Java 21 is correctly installed.
* Verify PostgreSQL credentials and network access.
* If configuration is encrypted, ensure CONFIG_MASTER_KEY is correct.
<!-- * Use `mvn -X` for verbose Maven debugging. -->

## Done!

Your backend should now be ready for development or production setup.
