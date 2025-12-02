---
sidebar_position: 3
---

# Frontend Setup Guide

This README explains how to set up and run the **SkyPulse Monitoring System Frontend**, built with **Vite**, **TypeScript**, and modern web tooling.

---

## Cloning the Repository

Clone the frontend from GitHub:

```sh
git clone git@github.com:Raddames-Tonui/skypulse_monitoring_system_frontend.git
```

Navigate into the project:

```sh
cd skypulse_monitoring_system_frontend
```

You can use any modern JavaScript-capable editor such as:

* VS Code
* WebStorm
* Sublime Text

---

## Environment Variables (`.env`)

Create a `.env` file in the project root:

#### Development .env
```env
VITE_BASE_API_URL=http://localhost:8000/api/rest
```

#### Production Sample .env
```env
VITE_BASE_API_URL=https://skypulse-monitoring-system-backend.onrender.com/api/rest
```

### Explanation

* **VITE_BASE_API_URL** — This is the base URL used by the frontend to interact with the backend API.

---

## Installing Dependencies

You can use **pnpm** or **npm**.

### Using pnpm

Install dependencies:

```sh
pnpm install
```

Start the development server:

```sh
pnpm run dev
```

Build for production:

```sh
pnpm run build
```

### Using npm

Install dependencies:

```sh
npm install
```

Start development:

```sh
npm run dev
```

Build production bundle:

```sh
npm run build
```

---

## Serving the Production Build

Most hosting platforms (Netlify, Vercel, Render static hosting) automatically detect Vite builds.

To preview locally after building:

```sh
npm run preview
# or
pnpm run preview
```

---

## Additional Notes

* Ensure the backend is reachable at the URL specified in your `.env` file.
* When deploying, update `VITE_BASE_API_URL` to your production backend URL.
* Vite automatically injects environment variables prefixed with `VITE_`.

---

## Done!

Your frontend should now be ready for development or production deployment.
