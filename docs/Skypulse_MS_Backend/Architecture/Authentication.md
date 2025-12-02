# User Authentication & Interaction Flow (Auth Context)

## USER INTERACTION FLOW

1. **User Login**

   * User enters email/password → clicks **Login**
   * `login(email, password)` calls `/auth/login` API
   * Server validates credentials

     * Verifies password hash against database
     * Checks user account status (`is_deleted`, role assignment, etc.)
     * Sets HttpOnly cookies: `accessToken` & `refreshToken`
     * Inserts a new session in `auth_sessions` table with `user_id`, `refresh_token_hash`, `jwt_id`, timestamps, and device info
     * Returns JSON: `{ user: { uuid, fullName, roleName, email, ... }, message }`
   * On success → `setUser(user)` updates Auth Context state

2. **Profile Fetch / Token Refresh**

   * User navigates to **Dashboard** or **Profile** page
   * `fetchProfile()` calls `/auth/profile` API

     * Axios automatically sends cookies (HttpOnly tokens)
     * If accessToken valid → server returns profile JSON → updates Auth Context state
     * If accessToken expired → Auth Middleware checks refresh token cookie

       * Valid refresh token → new access token issued, request retried
       * Invalid or missing refresh token → 401 Unauthorized

3. **Logout**

   * User clicks **Logout**
   * `logout()` calls `/auth/logout` API

     * Server clears HttpOnly cookies
     * Marks session in `auth_sessions` table as revoked
   * Auth Context state cleared

## DATABASE INTERACTIONS

* **Users table**: stores user credentials, UUID, name, email, role, company, password hash, account status
* **Auth_sessions table**: stores refresh token hash, JWT ID, session timestamps, device info, session status (`active` / `revoked`)
* **Login_failures table**: records failed login attempts
* **Roles & Company tables**: enrich user profile with roleName and companyName

## STATE LOCATIONS

* **HttpOnly tokens**: browser cookies (not accessible in JS)
* **User info & auth state**: Auth Context state
* **Profile & other fetched data**: Auth Context state

## AUTH FLOW CHART

```
Client Request (Login / Profile / Logout)
      |
      v
+------------------------+
|     Auth API Handler    |  <-- Receives request
|   (/auth/login, /auth/profile, /auth/logout)
+------------------------+
      |
      v
+------------------------+
| AuthMiddleware          |  <-- Checks accessToken
|  - Validates JWT        |
|  - Checks session in DB |
|  - Refreshes accessToken|
+------------------------+
      |
      v
+------------------------+
| Database Operations     |  <-- Users / Auth_sessions / Login_failures
|  - Verify credentials   |
|  - Insert session       |
|  - Update logout status |
+------------------------+
      |
      v
+------------------------+
| Response Construction   |  <-- Sets HttpOnly cookies, returns JSON profile / message
+------------------------+
      |
      v
Client Receives Response
      |
      +--> If login failed -> Error message
      +--> If accessToken expired but refreshToken valid -> Retry with new token
      +--> If logout -> State cleared, cookies removed
```
