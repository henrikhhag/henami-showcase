# Architecture

A technical deep dive into how henami is built.

## Overview

henami consists of two clients sharing a single backend:

```
┌─────────────────┐     ┌─────────────────┐
│  Web (React)    │     │  iOS (Expo)     │
│  Vite · Vercel  │     │  React Native   │
└────────┬────────┘     └────────┬────────┘
         │                       │
         └───────────┬───────────┘
                     ▼
         ┌───────────────────────┐
         │       Supabase        │
         │  Auth · Postgres ·    │
         │   Row-Level Security  │
         └───────────────────────┘
```

## Frontend (web)

- **React 19 + Vite** — fast HMR in development, optimized production build.
- **Code splitting** — every page loads as its own chunk via `React.lazy`. Especially important for heavy dependencies like Mapbox (maps) and Recharts (charts), so they aren't downloaded until you actually visit the page.
- **Routing** — React Router v7, all routing behind an auth gate.
- **Theming** — a custom theme system based on CSS variables on `<html>`. Lets the user customize colors live, stored in `localStorage`.

## Mobile (iOS)

- **React Native + Expo (SDK 54)** with **Expo Router** for file-based routing.
- Reuses pure business logic (date helpers, calculations) from web, but has its own UI components (`View`/`Text` instead of DOM).
- The auth session is stored in `AsyncStorage`; built and distributed via EAS / TestFlight.

## Backend (Supabase)

- **Postgres** with **row-level security** on every table. The pattern is `auth.uid() = bruker_id`, so each user only sees their own rows.
- **Sharing** — features like a shared calendar and joint finances are handled with dedicated `*_deling` (sharing) tables, so you only ever expose what you explicitly share.
- **Trigger** — a new user automatically gets a profile row created.

## Why these choices?

| Choice | Reasoning |
|---|---|
| Supabase | Auth, database and serverless functions in one package — fast to build on as a solo developer |
| RLS in the database | Security is enforced in the database, not in client code — harder to bypass |
| Expo (not just React Native CLI) | TestFlight/App Store distribution via EAS without local Xcode configuration |
| CSS variables for theming | Live color switching without re-rendering the React tree |
