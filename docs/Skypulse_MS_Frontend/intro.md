---
sidebar_position: 1
---


# Frontend Summary

A modern, high‑performance React application built with **TypeScript**, **TanStack Router**, and **TanStack Query**, and deployed on **Netlify**. This frontend powers the user interface for the Skypulse system, delivering real‑time visibility and role‑based access to monitoring operations.



## Overview

The frontend provides a clean, responsive, and fast UI layer for interacting with the Skypulse backend. It focuses on speed, reliability, and maintainability, leveraging proven tools and patterns:

* **React + TypeScript** for scalable, type‑safe UI development.
* **TanStack Router** for flexible, file‑based routing.
* **TanStack Query** for efficient server state caching and synchronization.
* **Netlify hosting** for global CDN performance, zero‑downtime deployments, and SSL.

The application communicates with the backend for authentication, service uptime events, SSL monitoring results, and all user‑role‑specific actions.



## Live Deployment

The frontend is deployed and accessible at:
**[https://skypulse-mss.netlify.app/](https://skypulse-mss.netlify.app/)**



## Features

### Real‑Time Monitoring UI

Displays live uptime/downtime snapshots, SSL statuses, and system health.

### Role‑Based Access Control (RBAC)

The system supports **three distinct user roles**, each with specific privileges:

#### **Admin**

* Manage users
* Manage services and service groups
* Modify system settings

#### **Operator**

* Add and edit services
* Manage reports
* Access dashboards with operational privileges

#### **Viewer**

* Read‑only access to dashboards and reports


## Tech Stack

* **React** – Component‑based UI
* **TypeScript** – Type safety and maintainability
* **TanStack Router** – Stable and powerful router
* **TanStack Query** – Server state management
* **Netlify** 


## Project Structure (Simplified)

```
src/
  components/
  hooks/
  pages/
  routes/
  services/     
  stores/
  utils/
  styles/
```



## Development

### Install Dependencies

```bash
pnpm install
```

### Start Dev Server

```bash
pnpm run dev
```

### Build for Production

```bash
pnpm run build
```



## Environment Variables

The app expects values such as:

```
VITE_API_BASE_URL=...
VITE_ENVIRONMENT=...
```




## License

Proprietary — part of the Skypulse Monitoring System.
