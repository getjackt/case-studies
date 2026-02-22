# MyCondoPro - MLS data platform

Six WordPress plugins that power a real estate search platform. Two do the heavy lifting (MLS sync engine and a React map UI), backed by four shared plugins handling auth, forms with CRM sync, core infrastructure, and templating. The hard part is scale. 120k active listings, 1.2M historical, and it all needs to stay current.

## Plugin 1: The sync engine

The MLS plugin pulls property data from the RESO Web API and keeps a local database in sync. It runs continuously, not a one-time import.

**Batched jobs with checkpointing.** The sync breaks into batches of 100 records via Action Scheduler. Each batch wraps inserts/updates in DB transactions and saves a checkpoint (last timestamp + listing key). If a job dies at record 47,000, the next one picks up at 47,001. No data loss, no restart from zero.

**Self-healing watchdog.** Runs every 5 minutes on its own schedule. Checks for stuck jobs (claimed for 20+ minutes), releases their locks, and reschedules. Detects missing recurring jobs and recreates them. Retries failed jobs with backoff. The sync basically maintains itself.

**Graceful degradation.** Media sync (images, documents) runs in separate jobs from the property import. If image processing fails, it doesn't block listings from appearing. Failed geocodes retry hourly in bulk. Each subsystem can fail independently without taking the rest down.

**Architecture.** PHP-DI container with auto-wiring, service-oriented layers (API > replication > repository > database), CQRS-style dual repository for reads vs writes. 185 classes across 53 directories. Full structured logging by component (API calls, imports, geocoding, media), all queryable from an admin dashboard.

## The shared platform

All six plugins share a PHP-DI container through a core plugin that handles dependency injection, structured logging, email rendering, and reCAPTCHA. The auth plugin manages registration, login, password reset, email verification. Rate limiting (IP and per-user), constant-time token comparison, and a token migration service for rolling security upgrades without logging everyone out.

The form plugin handles contact submissions with OAuth2 sync to Keap CRM. Token refresh, contact upserts (create vs update based on email match), retry on sync failure. Email alerts fire on new leads or failed syncs. Each trigger type plugs in without touching the core form logic.

None of this is the flashy part of the project, but it's what makes the platform work as a product instead of just two standalone plugins.

## Plugin 2: The map

React 18 + TypeScript frontend that renders those listings on an interactive Google Maps view with filtering.

**Server-side clustering.** You can't render 120k markers in a browser. The backend clusters markers by tile and zoom level before sending them. 100k listings become 50-200 cluster objects. Configurable grid density per zoom level from the admin panel.

**The filter system.** Nine specialized Zustand stores, one per filter type (city, price, bedrooms, property type, etc.) instead of one big store. Each store manages its own localStorage persistence and only triggers re-renders for components that subscribe to that specific filter. Had to research how people actually search and designed it around that.

**Sold listings.** Two separate database tables (available vs sold/historical), queried independently with different indexes. Date range filtering (3 months, 6 months, 1 year, all time) with a zoom gate on "all time" queries to prevent performance disasters. Available markers load first, sold fade in 300ms later.

**Coordinates.** Shared table between both plugins. Addresses are hashed for deduplication so multiple listings at the same address share one geocoded coordinate row. Spatial indexes (MySQL POINT columns) for distance queries. Bulk retry service handles failed geocodes automatically.

## Scale

- 6 plugins, ~530 PHP classes total
- ~120,000 active listings synced continuously
- ~1,200,000 historical records (sold/expired/closed)
- Map queries: <1.5s typical, <2s at P95 for "all time" sold data

## Tech

**Backend:** PHP 8, WordPress, MySQL 8, PHP-DI, Action Scheduler, RESO Web API, Google Geocoding API, OAuth2 (Keap CRM)

**Frontend:** React 18, TypeScript, Vite 5, Google Maps, TanStack Query v5, Zustand, Tailwind v4, WordPress REST API

**Auth & security:** Rate limiting, constant-time token comparison, token migration, reCAPTCHA

## Access

Live site and demo available on request.
