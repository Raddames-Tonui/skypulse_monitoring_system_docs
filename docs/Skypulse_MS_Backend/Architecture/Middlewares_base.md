---
sidebar_position: 3
---


# Middleware

## Overview

SkyPulse REST API is built using Undertow and provides role-based access to system settings, monitored services, and other core resources. It uses JWT for authentication and includes asynchronous request handling for performance.

## Request Flow

The following handlers manage requests:

```
Client Request
      |
      v
+--------------------+
|   CORSHandler      |  <-- Adds CORS headers
|  (OPTIONS? 204)    |
+--------------------+
      |
      v
+--------------------+
|    Dispatcher      |  <-- Async dispatch to worker thread
+--------------------+
      |
      v
+--------------------+
|  AuthMiddleware    |  <-- Validates JWT, session, attaches UserContext
+--------------------+
      |
      v
+--------------------+
| Business Handler   |  <-- e.g., UpsertSystemSettingHandler
| 
+--------------------+
      |
      v
  Response sent
      |
      +--> If HTTP method invalid -> InvalidMethod
      |
      +--> If unknown URI -> FallBack
```

### Handler Descriptions

1. **CORSHandler**

    * Adds `Access-Control-*` headers.
    * Handles OPTIONS preflight requests immediately (204).

2. **Dispatcher**

    * Dispatches request to a worker thread.
    * Prevents blocking I/O threads for database/network operations.

3. **AuthMiddleware**

    * Validates JWT token and session.
    * Checks user is active and not deleted.
    * Attaches `UserContext` with `userId` and `roleName`.

    Works with  `RoutesUtil` to check if Endpoint requires user to be authenticated or not.  ie `open` not needed, `secure` authentication is needed. 

4. **Business Handlers**

    * Use `UserContext` for role-based access.
    * Perform database operations (e.g., create/update system settings).
    * Respond with JSON via `ResponseUtil`.

5. **InvalidMethod**

    * Returns 405 with JSON when HTTP method is not allowed.

6. **FallBack**

    * Returns 404 with JSON when URI does not exist.


## Conclusion

* All endpoints return consistent JSON responses.
* Unauthorized or insufficient role requests return 401/403 JSON errors.
* Invalid methods and unknown URIs return 405/404 JSON errors.
* Dispatcher ensures async handling and high throughput.
