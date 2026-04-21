# Plant Crossing: Architecture Decision Records (ADR)
**Last updated:** April 20, 2026

This document captures the key architectural decisions made during the development of the Plant Crossing platform. Technology choices are based on requirements for reliability, time-to-market, Edge compatibility, and the need to build a scalable ecosystem.

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
* **Rationale:** Prisma uses a heavy Rust-based binary engine that introduces noticeable cold start latency in Serverless/Edge environments. Drizzle is a lightweight TypeScript wrapper over raw SQL without sacrificing type safety.
* **Competitive advantages:** Maximum performance (0ms overhead), full compatibility with Cloudflare Workers / Vercel Edge, 100% type-safety (infer types). Allows working closer to native SQL.

### 4. Authentication: Auth.js (NextAuth)
* **Alternatives:** Clerk, Firebase Auth.
* **Rationale:** SaaS solutions like Clerk create vendor lock-in and can exponentially increase costs as the user base grows (MAU).
* **Competitive advantages:** Auth.js allows full ownership of user data, storing sessions directly in your own PostgreSQL database. It is an open source solution that guarantees security and full control over data flow.

### 5. Infrastructure & Deployment: Vercel + GitHub Actions
* **Alternatives:** AWS (EC2/S3/Amplify), DigitalOcean (Droplets).
* **Rationale:** Managing a VPS or complex AWS configuration takes time away from product development. Vercel handles the infrastructure layer, allowing focus on business logic.
* **Competitive advantages:** Zero-config CI/CD. The key advantage is Preview Deployments — automatic creation of isolated test environments for every Pull Request — which radically accelerates feature testing without risk to production.