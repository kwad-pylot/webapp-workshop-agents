---
name: tech-stack-selector
description: |
  Evaluate and recommend technology choices for web applications.
  Considers requirements, team skills, and project constraints.
---

# Tech Stack Selector Skill

## Purpose
Analyze project requirements and recommend appropriate technologies.

## Evaluation Framework

### 1. Gather Requirements
- Application type (SPA, SSR, SSG, hybrid)
- Scale expectations (users, data volume)
- Team experience
- Timeline constraints
- Budget considerations
- Integration requirements

### 2. Evaluate Categories

#### Frontend Framework
| Option | Best For | Considerations |
|--------|----------|----------------|
| **Next.js** | Full-stack React apps, SEO needed | Vercel ecosystem, React knowledge required |
| **React + Vite** | SPAs, maximum flexibility | Need to set up routing, etc. |
| **Vue + Nuxt** | Gentle learning curve | Smaller ecosystem than React |
| **Svelte + SvelteKit** | Performance critical, simpler syntax | Smaller community |
| **Remix** | Full-stack, progressive enhancement | Newer, still maturing |

#### Backend Framework
| Option | Best For | Considerations |
|--------|----------|----------------|
| **Next.js API Routes** | Unified frontend/backend | Tied to Next.js |
| **Express.js** | Maximum flexibility, existing Node skills | Manual setup needed |
| **Fastify** | Performance, TypeScript | Less middleware ecosystem |
| **NestJS** | Enterprise apps, Angular-like structure | Opinionated, learning curve |
| **tRPC** | Type-safe APIs with TypeScript frontend | Tied to TypeScript |
| **Hono** | Edge deployments, lightweight | Newer |

#### Database
| Option | Best For | Considerations |
|--------|----------|----------------|
| **PostgreSQL** | Complex queries, reliability, flexibility | Hosted: Supabase, Neon, Railway |
| **MySQL** | Wide hosting support, familiarity | Less modern features than Postgres |
| **MongoDB** | Flexible schema, document data | Not ideal for relations |
| **SQLite** | Embedded, edge, prototypes | Single-writer limitation |
| **PlanetScale** | Serverless MySQL, branching | MySQL-compatible |
| **Supabase** | Postgres + realtime + auth | Full platform |

#### ORM/Query Builder
| Option | Best For | Considerations |
|--------|----------|----------------|
| **Prisma** | Type safety, migrations, ease of use | Generated client, some performance overhead |
| **Drizzle** | Performance, SQL-like syntax | Newer, less ecosystem |
| **Kysely** | Type-safe SQL, lightweight | More manual |
| **TypeORM** | Active Record pattern | Can be complex |

#### Authentication
| Option | Best For | Considerations |
|--------|----------|----------------|
| **NextAuth.js** | Next.js apps, OAuth providers | Tight Next.js integration |
| **Clerk** | Quick setup, managed service | Paid service |
| **Auth0** | Enterprise, many features | Can be expensive |
| **Supabase Auth** | Using Supabase stack | Tied to Supabase |
| **Custom JWT** | Full control | More to implement |
| **Lucia** | Lightweight, flexible | More setup required |

#### Styling
| Option | Best For | Considerations |
|--------|----------|----------------|
| **Tailwind CSS** | Rapid development, utility-first | Learning curve, verbose HTML |
| **CSS Modules** | Component scoping, standard CSS | More files |
| **styled-components** | Dynamic styles, CSS-in-JS | Runtime overhead |
| **Vanilla Extract** | Type-safe CSS, zero runtime | Build setup |

#### State Management
| Option | Best For | Considerations |
|--------|----------|----------------|
| **React Query / TanStack Query** | Server state | Best for API data |
| **Zustand** | Simple global state | Minimal boilerplate |
| **Jotai** | Atomic state | Fine-grained updates |
| **Redux Toolkit** | Complex state, time-travel debugging | More boilerplate |

#### Hosting/Deployment
| Option | Best For | Considerations |
|--------|----------|----------------|
| **Vercel** | Next.js, frontend apps | Free tier limits |
| **Netlify** | Static sites, serverless | Similar to Vercel |
| **Railway** | Full-stack, databases | Easy deployment |
| **Render** | Full-stack, containers | Good free tier |
| **Fly.io** | Edge deployment, containers | Global distribution |
| **AWS/GCP/Azure** | Full control, enterprise | Complex setup |

## Recommendation Template

```markdown
## Tech Stack Recommendation

### Project: [Name]
### Requirements Summary
- Type: [SPA/SSR/Full-stack]
- Scale: [Expected users/data]
- Team: [Experience level]
- Timeline: [Deadline if any]

### Recommended Stack

| Layer | Technology | Rationale |
|-------|------------|-----------|
| Frontend | Next.js 14 | SSR needed, React experience |
| Styling | Tailwind CSS | Rapid development |
| Backend | Next.js API Routes | Unified codebase |
| Database | PostgreSQL (Supabase) | Relational data, realtime needed |
| ORM | Prisma | Type safety, easy migrations |
| Auth | Supabase Auth | Integrated with database |
| Hosting | Vercel + Supabase | Optimized for Next.js |

### Alternatives Considered

| Layer | Alternative | Why Not |
|-------|-------------|---------|
| Database | MongoDB | Relational data model preferred |
| Auth | Custom JWT | Time constraints |

### Trade-offs
- **Pros:** Rapid development, type safety, integrated ecosystem
- **Cons:** Vendor lock-in with Vercel/Supabase

### Getting Started
1. `npx create-next-app@latest`
2. Add Tailwind: `npm install -D tailwindcss`
3. Add Prisma: `npm install prisma @prisma/client`
4. Configure Supabase project
```

## Quick Recommendations

### MVP / Prototype
- Next.js + Tailwind + SQLite/Prisma
- Vercel deployment
- Simple, fast to build

### Production SaaS
- Next.js + Tailwind + PostgreSQL/Prisma
- Supabase or Railway for database
- Vercel for frontend

### Enterprise
- Next.js or NestJS
- PostgreSQL with proper hosting
- Auth0 or similar enterprise auth
- AWS/GCP infrastructure
