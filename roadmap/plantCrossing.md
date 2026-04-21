# Plant Crossing: Step-by-Step Development Roadmap
**Last updated:** April 20, 2026
**Status:** Active development
**Goal:** Building a plant exchange platform in Ukraine; demonstrating Fullstack expertise for Scalr (summer 2026).

## "Walking Skeleton" Strategy
Given the tight timeline (1.5–2 months until summer), development follows the "Walking Skeleton" principle: priority on rapid deployment of a minimally working infrastructure, with complex functionality added incrementally.

### Phase 1: Infrastructure Setup (Week 1)
* **Development environment:** Configuring PhpStorm/WebStorm with the Continue.dev / Cody plugin (Claude 3.5 Sonnet integration for AI-assisted coding).
* **Repository:** Initializing a Next.js monorepo (App Router).
* **Database:** Creating a PostgreSQL instance (Neon or Supabase).
* **ORM:** Configuring Drizzle ORM, converting Mermaid UML diagrams into TS schemas, generating the first migration.

### Phase 2: Core MVP Functionality (Weeks 2–3)
* **Authentication:** Integrating Auth.js (NextAuth) for Google/Email login with session storage in the database.
* **Entity management (CRUD):** Building profile pages (Collector/Seeker) and management of user plant instances (`UserPlant`).
* **Exchange feed:** Creating a general marketplace/feed with basic filters for displaying available listings.

### Phase 3: Complex Feature Integration (Weeks 4–5)
* **Transactional Swap:** Implementing the plant exchange mechanic (request → confirmation → logging in `ActionLog` with ACID compliance).
* **AI Assistant:** Integrating the Claude API into the backend to build a plant care consultant bot based on system prompts and plant parameters.
* **Interactive map:** Connecting Mapbox GL JS or Google Maps API for clustered display of available plants across Ukrainian cities.

### Phase 4: Polish & Deployment (Week 6)
* **UI/UX:** Building responsive components with Tailwind CSS (converting Figma designs).
* **CI/CD & Deployment:** Configuring GitHub Actions, deploying to Vercel (verifying Edge functions, optimizing LCP/SEO).
* **Review:** Final code and architecture audit before the presentation.