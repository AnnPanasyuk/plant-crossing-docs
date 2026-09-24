# Plant Crossing: Architecture Decision Records (ADR)
**Last updated:** September 23, 2026

This document captures the key architectural decisions made during the development of the Plant Crossing platform. Technology choices are based on requirements for reliability, time-to-market, and the need to build a scalable ecosystem.

### 1. Framework: Next.js (App Router)
* **Alternatives:** Remix, Vite + Express.
* **Rationale:** The project requires a high level of SEO (searching for specific plant species) and an optimized first screen load (LCP) for mobile devices. Next.js with Server Components allows heavy logic to run on the server, sending minimal JS to the client.
* **Competitive advantages:** Largest community, reference integration with Vercel, de facto industry standard for React development in 2026.
* **Note:** Next.js is also the backend of the platform — there is no separate API service. How server logic is organized is defined in ADR 8.

### 2. Database: PostgreSQL
* **Alternatives:** MongoDB (NoSQL), MySQL.
* **Rationale:** The core of the platform is the asset exchange mechanic (plants) between users. This requires strict transactional guarantees (ACID). If an exchange is interrupted, data must be guaranteed to roll back.
* **Competitive advantages:** The `JSONB` type allows storing dynamic characteristics of unique plants without breaking the rigid relational structure of users and transactions.
* **Note:** The rollback guarantee depends on the driver used for the write — see ADR 3, "Drivers".

### 3. ORM: Drizzle ORM
* **Alternatives:** Prisma, TypeORM.
* **Rationale:** Drizzle generates lighter, more predictable SQL with no runtime query engine — critical for serverless cold starts on Vercel. Schema-as-code in TypeScript gives full type inference without a separate generation step. `drizzle-kit` covers migrations (generate/migrate/studio) with less abstraction overhead than Prisma's migration toolchain.
* **Competitive advantages:** No binary query engine to bundle — smaller deployment size, faster cold starts on Vercel serverless. Direct SQL-like query builder keeps full control over generated queries. Native support for both Neon drivers (see below).
* **Drivers:** Two Drizzle clients, each for its own job.
  * `drizzle-orm/neon-http` — exported as `db` from `db/index.ts`. Used for reads and single-statement writes. **Does not support interactive transactions**: `db.transaction()` throws on this driver. `db.batch()` executes several statements atomically, but without logic between them (cannot read a row, check a condition, then write).
  * `drizzle-orm/neon-serverless` (WebSocket `Pool`) — used only for interactive transactions, exposed via `runInTransaction(callback)` from `db/index.ts`. Required by the swap flow (check both plants are still available → change their statuses → create the swap record, all-or-nothing), which is what ADR 2 promises.
  * **Verify before implementing:** Pool lifecycle in Vercel serverless functions and whether the runtime needs the `ws` package as the WebSocket constructor — check the current Neon serverless driver docs, not community examples.

### 4. Image Storage: Vercel Blob
* **Alternatives:** Cloudflare R2, Supabase Storage, AWS S3.
* **Rationale:** `PLANT_PHOTOS` requires a file storage solution. Vercel Blob is the zero-config choice within the existing Vercel deployment, with no additional infrastructure to manage.
* **Competitive advantages:** Native integration with Next.js API routes. Simple SDK (`@vercel/blob`). Automatic CDN delivery. If storage costs become a concern at scale, migration to Cloudflare R2 is straightforward as both expose S3-compatible APIs.
* **⚠️ Open issue (unresolved): 4.5 MB request body limit.** Vercel Functions reject request bodies over 4.5 MB with `413 FUNCTION_PAYLOAD_TOO_LARGE`. The current design sends all photos in one `multipart/form-data` `POST /api/listings` (`image-uploader-spec.md`, `listing-form-spec-v2.md` → "Публікація — одна дія"). Two options:
  * **A. Client-side size budget.** Resize (longest edge) + JPEG quality in the existing canvas step so that the *maximum* number of photos always fits under 4.5 MB together with text fields. Keeps the current architecture. Budget must be calculated from the max photo count, not the average.
  * **B. Vercel Blob client uploads.** `upload()` from `@vercel/blob/client` + a Route Handler with `handleUpload` issuing upload tokens; this is the path Vercel Blob docs recommend for files over 4.5 MB. `POST /api/listings` then receives JSON with blob URLs; moderation reads photos from Blob. Cost: blobs of rejected listings become orphans and need cleanup.
  * **Decision input needed:** max photos per listing and target size per photo after canvas. If option B is chosen, `image-uploader-spec.md` and `listing-form-spec-v2.md` must be updated.

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
* **Server side:** polling статусу обміну йде в `GET /api/swaps/[id]` (Route Handler); мутації wishlist — Server Action, яку TanStack Query викликає як `mutationFn`. Див. ADR 8.

### 8. Backend Layer: Next.js server runtime + service layer
* **Alternatives:** Separate API service (Express / NestJS); business logic written directly inside Route Handlers and Server Actions.
* **Rationale:**
  * A separate API service duplicates auth, deployment and types, and removes the main benefit of Server Components — reading data on the server without an HTTP hop. Single Vercel deployment (ADR 6) stays intact.
  * Logic inside Route Handlers / Server Actions was rejected: Server Actions are reachable by a direct POST request, so Next.js docs require treating them as public API endpoints. Access checks scattered across entry points will eventually be missed in one of them. The same operation is also often needed from more than one entry point (a page read and an API route).
  * Follows the **Data Access Layer** pattern from the Next.js authentication / data security docs: one server-only module owns DB access and authorization; entry points are thin adapters.

#### Layers

| Layer | Location | Used for | Not used for |
|---|---|---|---|
| Server Components | `app/**/page.tsx`, `layout.tsx` | Reads during page render: catalog, listing detail, profile. Call services directly. | Mutations |
| Server Actions | `lib/actions/<entity>.ts` (`'use server'`) | Mutations from UI: wishlist toggle, accept/decline swap, profile edit. | Reads, polling (POST-only); file uploads (default body limit 1 MB) |
| Route Handlers | `app/api/**/route.ts` | Anything that needs a real HTTP contract: file upload, GET for polling, Auth.js handlers, webhooks, calls not initiated by a React form. | Business logic |
| Services (DAL) | `lib/services/<entity>.ts` | All business logic, authorization, validation of input, DB writes, transactions (`runInTransaction`). | HTTP / request parsing, UI concerns |

#### Rules
* Route Handlers and Server Actions only: parse the request → call one service function → map the result to a response. No DB access, no business rules inside them.
* Every service function that reads private data or mutates checks the session and resource ownership itself (via `lib/services/access.ts`). Middleware and UI-level hiding are not a security boundary.
* Every module in `lib/services/`, `lib/claude/` and `db/` starts with `import 'server-only'`, so an accidental import into a Client Component fails at build time.
* Interactive transactions only through `runInTransaction` (ADR 3). The `db` (neon-http) client is never used for multi-step writes.

#### Recommended structure

```
auth.ts                              # Auth.js v5 config: exports auth, handlers, signIn, signOut
db/
  index.ts                           # db (neon-http) + runInTransaction (neon-serverless Pool)
  schema.ts
drizzle/                             # generated migrations
lib/
  claude/                            # existing: analyzePlant, identifyPlant, checkForScam, SEO, tags
  services/                          # DAL — the only place that touches db/
    access.ts                        # getCurrentUser, requireUser, requireListingOwner
    listings.ts                      # getListings, getListingById, createListing (moderation gate → insert)
    swaps.ts                         # getSwapStatus, proposeSwap, acceptSwap, declineSwap (transactions)
    wishlist.ts                      # getWishlist, toggleWishlistItem
    profile.ts                       # getProfile, updateProfile
  actions/                           # 'use server' — thin adapters over services
    wishlist.ts
    swaps.ts
    profile.ts
  utils/                             # existing: domain-free utilities
app/
  api/
    auth/[...nextauth]/route.ts      # export const { GET, POST } = handlers
    ai/                              # existing AI routes (thin wrappers over lib/claude)
    listings/route.ts                # POST — multipart or JSON, depends on ADR 4 open issue
    swaps/[id]/route.ts              # GET — swap status for TanStack Query polling (ADR 7)
  (auth)/                            # existing
  (marketplace)/                     # existing
```

Imports follow the existing alias groups: `@plant-crossing/lib/services/...`, `@plant-crossing/lib/actions/...` — no new Prettier `importOrder` group is needed.

#### Current features mapped

| Feature | Entry point | Service |
|---|---|---|
| Publish listing (moderation gate) | `POST /api/listings` | `listings.createListing` |
| Catalog, listing detail | Server Component | `listings.getListings`, `listings.getListingById` |
| Wishlist toggle (optimistic) | Server Action as TanStack Query `mutationFn` | `wishlist.toggleWishlistItem` |
| Propose / accept / decline swap | Server Action | `swaps.*` via `runInTransaction` |
| Swap status polling (10s) | `GET /api/swaps/[id]` | `swaps.getSwapStatus` |
| Plant identify / AI helpers | `app/api/ai/**` | `lib/claude/*` |