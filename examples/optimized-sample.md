# Project

Next.js 14 (App Router) + TypeScript strict + Prisma + Tailwind CSS. pnpm. Deploys to Vercel via GitHub integration.

## Commands

```bash
pnpm test              # Vitest unit tests
pnpm test:e2e          # Playwright e2e
pnpm test:coverage     # Coverage report (target: 80%+ on new code)
pnpm lint && pnpm typecheck  # Run before every commit
```

## Code Standards

- TypeScript strict mode, no `any` — use `unknown` or generics instead
- ES modules only, no CommonJS `require()`
- Functional components + hooks only, no class components
- Validate API request bodies with Zod schemas
- Prisma for all DB access — use `select` to limit fields, transactions for multi-record writes
- Tailwind utilities only — custom CSS via CSS modules when unavoidable
- Follow design tokens in `tailwind.config.ts`, no arbitrary pixel values

## Architecture Decisions

- Component-specific types co-locate with component; shared types go in `src/types/`
- API routes in `src/app/api/` — RESTful naming, proper HTTP status codes
- Use Next.js Image component, `React.lazy()` for code splitting
- Rate limiting on public API endpoints

## Environment

See `.env.example` for required variables. Key services: PostgreSQL, Redis, Stripe, S3.

## Git

Conventional commits (`feat:`, `fix:`, `refactor:`, `chore:`). Branch from `main`. Keep PRs small and focused.

## Monitoring

Datadog Core Web Vitals dashboard: `https://app.datadoghq.com/dashboard/abc-def-123`
