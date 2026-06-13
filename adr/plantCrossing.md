# Plant Crossing: Architecture Decision Records (ADR)
**Last updated:** June 13, 2026

This document captures the key architectural decisions made during the development of the Plant Crossing platform. Technology choices are based on requirements for reliability, time-to-market, and the need to build a scalable ecosystem.

### 1. Framework: Next.js (App Router)
* **Alternatives:** Remix, Vite + Express.
* **Rationale:** The project requires a high level of SEO (searching for specific plant species) and an optimized first screen load (LCP) for mobile devices. Next.js with Server Components allows heavy logic to run on the server, sending minimal JS to the client.
* **Competitive advantages:** Largest community, reference integration with Vercel, de facto industry standard for React development in 2026.

### 2. Database: PostgreSQL
* **Alternatives:** MongoDB (NoSQL), MySQL.
* **Rationale:** The core of the platform is the asset exchange mechanic (plants) between users. This requires strict transactional guarantees (ACID). If an exchange is interrupted, data must be guaranteed to roll back.
* **Competitive advantages:** The `JSONB` type allows storing dynamic characteristics of unique plants without breaking the rigid relational structure of users and transactions.

### 3. ORM: Drizzle ORM
* **Alternatives:** Prisma, TypeORM.
* **Rationale:** Drizzle generates lighter, more predictable SQL with no runtime query engine — critical for serverless cold starts on Vercel. Schema-as-code in TypeScript gives full type inference without a separate generation step. `drizzle-kit` covers migrations (generate/migrate/studio) with less abstraction overhead than Prisma's migration toolchain.
* **Competitive advantages:** No binary query engine to bundle — smaller deployment size, faster cold starts on Vercel serverless. Direct SQL-like query builder keeps full control over generated queries. Native Neon HTTP driver support (`db/index.ts` exports `db` via Drizzle + Neon HTTP driver).

### 4. Image Storage: Vercel Blob
* **Alternatives:** Cloudflare R2, Supabase Storage, AWS S3.
* **Rationale:** `PLANT_PHOTOS` requires a file storage solution. Vercel Blob is the zero-config choice within the existing Vercel deployment, with no additional infrastructure to manage.
* **Competitive advantages:** Native integration with Next.js API routes. Simple SDK (`@vercel/blob`). Automatic CDN delivery. If storage costs become a concern at scale, migration to Cloudflare R2 is straightforward as both expose S3-compatible APIs.

### 5. Authentication: Auth.js v5 (NextAuth)
* **Alternatives:** Clerk, Firebase Auth.
* **Rationale:** SaaS solutions like Clerk create vendor lock-in and can exponentially increase costs as the user base grows (MAU).
* **Competitive advantages:** Full ownership of user data, sessions stored directly in PostgreSQL. Open source. Note: Auth.js v5 API was in beta for an extended period — expect some discrepancy between community examples and current docs. Use the official docs at `authjs.dev` as the single source of truth.

### 6. Infrastructure & Deployment: Vercel + GitHub Actions
* **Alternatives:** AWS (EC2/S3/Amplify), DigitalOcean (Droplets).
* **Rationale:** Managing a VPS or complex AWS configuration takes time away from product development. Vercel handles the infrastructure layer, allowing focus on business logic.
* **Competitive advantages:** Zero-config CI/CD. Preview Deployments — automatic creation of isolated test environments for every Pull Request — radically accelerates feature testing without risk to production.

### 7. State Management: Zustand + TanStack Query
* **Alternatives:** Redux Toolkit, React Context + useState, Jotai.
* **Rationale:** Стан розділено на три чіткі шари з різними інструментами:
  — Server Components — серверний статичний стан (каталог, деталі, профіль) без будь-яких бібліотек.
  — Zustand — глобальний UI стан: кошик з `persist` middleware (виживає після перезавантаження), wishlist count badge, drawer/popup стани.
  — TanStack Query — динамічний серверний стан: polling статусу обміну (10s interval), оптимістичні апдейти wishlist.
  Redux відхилено як overkill — App Router з Server Components вирішує серверний стан без стору.
* **Rollout:** Zustand додається в Phase 2 (Core MVP). TanStack Query — в Phase 3 (Swap flow), коли з'являється потреба в polling і оптимістичних апдейтах.
