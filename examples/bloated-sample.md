# Project Guidelines for Claude

## General Behavior

You are an AI coding assistant. Be helpful, accurate, and concise in all your responses. Always strive to provide the best possible assistance to the developer. Write clean, readable, and maintainable code at all times. Follow best practices for software development. Think step by step when solving complex problems.

## About This Project

This is a Next.js 14 application using the App Router with TypeScript, Prisma ORM for database access, and Tailwind CSS for styling. The project uses pnpm as the package manager. We deploy to Vercel.

### Project Structure

The project is organized as follows:

```
src/
  app/           - Next.js App Router pages and layouts
  components/    - Reusable React components
  lib/           - Utility functions and shared code
  hooks/         - Custom React hooks
  types/         - TypeScript type definitions
  styles/        - Global styles and Tailwind config
prisma/
  schema.prisma  - Database schema
  migrations/    - Database migrations
public/          - Static assets
```

## TypeScript Guidelines

When writing TypeScript code, please ensure that you always use strict mode. TypeScript's strict mode helps catch potential errors at compile time and improves code quality. Avoid using the `any` type wherever possible, as it defeats the purpose of using TypeScript in the first place. Instead, use proper type definitions, generics, or `unknown` when the type is truly uncertain. Always define interfaces for component props and API response types. Make sure to use proper type narrowing with type guards when working with union types.

When creating new types, place them in the `src/types/` directory. For component-specific types that are only used in one place, you can co-locate them with the component file. Always export types that might be reused across the codebase.

## Code Style and Quality

Write clean, readable code with meaningful variable and function names. Use descriptive names that clearly convey the purpose of the variable or function. Avoid single-letter variable names except in short loops or lambda functions.

Always follow the existing code style in the project. Look at surrounding code to understand the patterns and conventions being used, and match them in your contributions.

Add comments where needed to explain complex logic, but don't over-comment obvious code. Comments should explain "why" not "what".

Make sure to handle edge cases and error conditions appropriately. Think about what could go wrong and add proper error handling.

Use ES modules (import/export) for all module imports. Never use CommonJS require() syntax in the source code.

## React Component Guidelines

When creating React components, follow these best practices:

1. Use functional components with hooks — never use class components
2. Keep components small and focused on a single responsibility
3. Extract reusable logic into custom hooks in the `src/hooks/` directory
4. Use `React.memo()` for components that receive stable props and render frequently
5. Always add proper TypeScript types for component props
6. Handle loading and error states in all components that fetch data

### Component File Structure

Every component file should follow this structure:

```typescript
// Imports (external, then internal, then styles)
import React from 'react';
import { useRouter } from 'next/navigation';

import { Button } from '@/components/ui/Button';
import { cn } from '@/lib/utils';

// Types
interface MyComponentProps {
  title: string;
  onAction: () => void;
}

// Component
export function MyComponent({ title, onAction }: MyComponentProps) {
  // hooks first, then derived state, then handlers, then render
  return <div>{title}</div>;
}
```

## API Routes

All API routes should be placed in `src/app/api/`. Follow RESTful conventions for naming and HTTP methods. Always validate request bodies using Zod schemas before processing.

### Error Handling in API Routes

Always return proper HTTP status codes:
- 200 for successful GET/PUT requests
- 201 for successful POST requests that create resources
- 400 for validation errors
- 401 for authentication errors
- 403 for authorization errors
- 404 for not found
- 500 for internal server errors

Always wrap API route handlers in try/catch blocks and return structured error responses.

## Database Access

Use Prisma for all database operations. Never write raw SQL queries unless absolutely necessary for performance-critical operations. Always use Prisma transactions for operations that modify multiple related records.

### Database Best Practices

- Always use `select` to limit returned fields for performance
- Use `include` sparingly — prefer separate queries for complex relations
- Always handle the case where a database record is not found
- Use Prisma's built-in pagination for list endpoints

## Testing

Write comprehensive test suites for all new features. We use Vitest for unit tests and Playwright for end-to-end tests. Every PR should include appropriate test coverage.

### Test Commands

```bash
pnpm test          # Run unit tests
pnpm test:e2e      # Run e2e tests
pnpm test:coverage # Run tests with coverage report
```

Aim for at least 80% code coverage on new code. All critical business logic must have unit tests.

### Testing Best Practices

Keep changes minimal and focused — each PR should change fewer than 50 lines of code when possible. This makes reviews easier and reduces the risk of introducing bugs.

## Styling

Use Tailwind CSS utility classes for all styling. Avoid writing custom CSS unless absolutely necessary. When you need custom styles, use CSS modules.

Follow the design system tokens defined in `tailwind.config.ts`. Use the spacing scale consistently and never use arbitrary pixel values.

## Git Workflow

- Create feature branches from `main`
- Use conventional commits: `feat:`, `fix:`, `refactor:`, `chore:`
- Keep PRs small and focused
- Always run `pnpm lint && pnpm typecheck` before committing

## Environment Variables

Environment variables are defined in `.env.local` for development. See `.env.example` for the required variables. Never commit actual environment variable values.

Required variables:
- `DATABASE_URL` — PostgreSQL connection string
- `NEXTAUTH_SECRET` — NextAuth.js secret key
- `NEXTAUTH_URL` — Application URL
- `STRIPE_SECRET_KEY` — Stripe API key
- `STRIPE_WEBHOOK_SECRET` — Stripe webhook signing secret
- `S3_BUCKET_NAME` — AWS S3 bucket for file uploads
- `REDIS_URL` — Redis connection for caching

## Deployment

We deploy to Vercel using the GitHub integration. The `main` branch auto-deploys to production. Preview deployments are created for all PRs.

### Deployment Checklist

Before deploying, make sure to:
1. Run all tests locally
2. Check that TypeScript compiles without errors
3. Verify environment variables are set in Vercel dashboard
4. Review the Vercel build logs after deployment

## Performance

- Use Next.js Image component for all images
- Implement proper caching strategies with Redis
- Use React.lazy() and dynamic imports for code splitting
- Monitor Core Web Vitals using our Datadog dashboard at `https://app.datadoghq.com/dashboard/abc-def-123`

## Security

- Always sanitize user input
- Use parameterized queries (Prisma handles this)
- Implement rate limiting on public API endpoints
- Follow OWASP Top 10 guidelines
- Never log sensitive data like passwords or API keys
