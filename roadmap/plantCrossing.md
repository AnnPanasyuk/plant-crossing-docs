# Plant Crossing: Step-by-Step Development Roadmap
**Last updated:** April 22, 2026
**Status:** Active development
**Goal:** Building a plant exchange platform in Ukraine; demonstrating Fullstack expertise for Scalr (summer 2026).

## "Walking Skeleton" Strategy
Given the tight timeline (1.5–2 months until summer), development follows the "Walking Skeleton" principle: priority on rapid deployment of a minimally working infrastructure, with complex functionality added incrementally.

### Phase 1: Infrastructure Setup (Week 1)
* **Development environment:** Configuring WebStorm with the Continue.dev / Cody plugin (Claude Sonnet integration for AI-assisted coding).
* **Repository:** Initializing a Next.js monorepo (App Router).
* **Database:** Creating a PostgreSQL instance (Neon or Supabase).
* **ORM:** Configuring Prisma, converting Mermaid UML diagrams into Prisma schema, generating the first migration via `prisma migrate dev`.
* **Image storage:** Configuring Vercel Blob (`@vercel/blob`) for `PLANT_PHOTOS`.

### Phase 2: Core MVP Functionality (Weeks 2–3)
* **Authentication:** Integrating Auth.js v5 (NextAuth) for Google/Email login with session storage in the database.
* **Entity management (CRUD):** Building profile pages (Collector/Seeker) and management of user plant instances (`UserPlant`), including photo upload via Vercel Blob.
* **Exchange feed:** Creating a general marketplace/feed with basic filters for displaying available listings.

### Phase 3: Core Transactional Feature (Weeks 4–5)
* **Transactional Swap:** Implementing the plant exchange mechanic (request → confirmation → logging in `ActionLog` with ACID compliance). This is the core mechanic of the platform and the primary focus of this phase.
* **Interactive map:** Connecting Mapbox GL JS or Google Maps API for clustered display of available plants across Ukrainian cities.

### Phase 4: Polish & Deployment (Week 6)
* **UI/UX:** Building responsive components with Tailwind CSS (converting Figma designs).
* **AI Assistant (stub):** Integrating Claude API as a plant care consultant. Delivered as a functional stub with real API integration — full feature scope deferred post-launch to avoid timeline risk.
* **CI/CD & Deployment:** Configuring GitHub Actions, deploying to Vercel (verifying LCP/SEO).
* **Review:** Final code and architecture audit before the presentation.