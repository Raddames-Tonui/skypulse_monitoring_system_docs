---
sidebar_position: 3
---


# RBAC & Authentication Architecture (Frontend)

This document explains how **Authentication** and **Role‑Based Access Control (RBAC)** work in your frontend. It also includes explanations of the code snippets you provided.

---

## 1. Authentication Flow Overview

The frontend uses a combination of:

* **React Context** → Stores the logged‑in user
* **TanStack Router** → Protects routes and performs pre‑loading authentication
* **Axios Client** → Sends authenticated requests to backend (`/auth/login`, `/auth/profile`, `/auth/logout`)
* **React Hooks** → Access auth data globally
* **Role-Based Routing** → Allows pages based on user.role

### Lifecycle

1. App loads → `AuthProvider` runs `fetchProfile()` to check if the user is logged in.
2. If authenticated → user is stored in context and protected pages unlock.
3. If not authenticated → user is redirected to login.
4. If authenticated but lacks role permissions → redirect to Unauthorized page.

---

## 2. AuthProvider Explained

This is the global authentication manager.

### Key Responsibilities

* Store `user` + loading state
* Login
* Logout
* Fetch profile
* Map backend user object to frontend-friendly object
* Provide these functions to the whole app

### Breakdown

```ts
const [user, setUser] = useState<UserProfile | null>(null);
const [isLoading, setIsLoading] = useState(true);
```

Stores who is logged in + whether auth is still loading.

```ts
useEffect(() => { initProfile(); }, []);
```

When the app loads, it tries to fetch the profile.

```ts
const login = async (email, password) => {...}
```

* Calls `/auth/login`
* Saves profile into context
* Handles errors with toast

```ts
const fetchProfile = async () => {...}
```

Makes a request to `/auth/profile` using stored cookies.

```ts
const logout = async () => {...}
```

* Calls `/auth/logout`
* Clears user

```ts
<AuthContext.Provider value={{ user, isLoading, login, logout, fetchProfile, error }}>
```

Allows the rest of the app to consume auth state.

---

## 3. Route Protection (TanStack Router)

Protected routes are enforced at two levels:

* **beforeLoad** (route-level auth before rendering)
* **ProtectedRouteComponent** (UI-based enforcement)

### beforeLoad Example

```ts
beforeLoad: async () => {
  try {
    await axiosClient.get('/auth/profile');
  } catch {
    return redirect({ to: '/auth/login' });
  }
}
```

This prevents loading protected pages if the user has no active session.

### ProtectedRouteComponent

Once the route renders, it double‑checks the user in context:

```ts
if (isLoading || !user) return <Loader />;
```

Guarantees no page renders before auth is ready.

It then renders:

* Navbar
* Sidebar
* Page Layout (Outlet)

---

## 4. RBAC (Role-Based Access Control)

Roles such as **ADMIN** and **OPERATOR** determine access.

### How It Works

Special routes use:

```ts
<ProtectedRoute allowed={['ADMIN']} />
```

Or multiple roles:

```ts
<ProtectedRoute allowed={['OPERATOR', 'ADMIN']} />
```

The `ProtectedRoute` component checks:

```ts
if (!allowed.includes(user.roleName)) {
  return <UnauthorizedPage />;
}
```

If the user's role does not match → redirect to Unauthorized.

---

## 5. Unauthorized Page

If users try accessing forbidden routes, they see:

* A clean access denied message
* Option to return to dashboard

This improves UX and clearly communicates role limitations.

---

## 6. Summary of System Behavior

| Feature               | Description                                         |
| --------------------- | --------------------------------------------------- |
| Login                 | Saves user & session cookies automatically          |
| Auto‑login            | On page refresh, `fetchProfile()` restores session  |
| Logout                | Clears user, removes UI, server clears cookies      |
| Route protection      | beforeLoad + UI guard                               |
| RBAC                  | Allows rendering only if `user.roleName` is allowed |
| Unauthorized fallback | Shows friendly error page                           |

---

## 7. Why This Architecture Is Strong

* **Secure**: Backend controls session via httpOnly cookies
* **Clean UI**: No flickers because of loaders
* **Scalable**: Adding new roles is easy
* **Centralized Auth**: One context handles everything
* **Reusable Guards**: Role-based wrappers work across many pages

---

If you'd like, I can also add:

* Sequence diagrams (Auth flow)
* RBAC table showing roles & permissions
* Backend pairing explanation (JWT/refresh tokens)
* How to extend RBAC for more roles
