# CLAUDE.md Context Audit Report

**Date**: 2025-07-15
**Project**: /home/dev/my-nextjs-app
**Model**: claude-opus-4-20250514

## Executive Summary

- **Grade**: D (62/100)
- **Files scanned**: 1 (CLAUDE.md)
- **Total tokens**: ~3,850 (~1.9% of 200K context window)
- **Instructions found**: 64 (of which 47% are actionable)
- **Estimated token savings**: ~2,900 tokens (75% reduction)

## Context Budget Breakdown

| Source | Tokens | % of Window |
|--------|--------|-------------|
| System prompt (est.) | ~15,000 | ~7.5% |
| CLAUDE.md (combined) | ~3,850 | ~1.9% |
| MCP tool schemas | ~2,000 | ~1.0% |
| Memory (MEMORY.md) | 0 | 0% |
| Rules (.claude/rules/) | 0 | 0% |
| **Available for work** | **~179,150** | **~89.6%** |

## Dimension Scores

| Dimension | Score | Grade | Key Issue |
|-----------|-------|-------|-----------|
| Actionability & Specificity | 47 | F | 34 of 64 instructions are vague or aspirational |
| Token Efficiency | 40 | F | 3,850 tokens with density of 60 tokens/instruction |
| Coverage & Completeness | 72 | C | Good coverage but verbose — commands present, architecture over-described |
| Consistency & Non-contradiction | 75 | C | 1 direct contradiction, 4 semantic duplicates |
| Default Behavior Overlap | 30 | F | 7 instructions restate Claude's default behavior |
| Currency & Accuracy | 85 | B | 1 potentially stale dashboard URL, rest verifiable |

## Findings

### CRITICAL

**[C-001]**: Default behavior restatement block
- **Severity**: CRITICAL
- **Location**: `CLAUDE.md:5-7`
- **Evidence**: > "Be helpful, accurate, and concise in all your responses. Always strive to provide the best possible assistance... Write clean, readable, and maintainable code at all times. Follow best practices for software development. Think step by step when solving complex problems."
- **Proposed**: delete
- **Impact**: Removes ~46 wasted tokens restating Claude's built-in behaviors; "think step by step" is redundant with extended thinking.

**[C-002]**: Verbose TypeScript section could be a one-liner
- **Severity**: CRITICAL
- **Location**: `CLAUDE.md:34-42`
- **Evidence**: > "When writing TypeScript code, please ensure that you always use strict mode. TypeScript's strict mode helps catch potential errors at compile time..."
- **Proposed**: `TypeScript strict mode, no 'any' — use 'unknown' or generics instead`
- **Impact**: Reduces 156 tokens to 14 while preserving the only actionable instruction.

**[C-003]**: Project structure section duplicates what `ls` reveals
- **Severity**: CRITICAL
- **Location**: `CLAUDE.md:14-27`
- **Evidence**: > Full directory tree block listing `src/app`, `src/components`, `src/lib`, etc.
- **Proposed**: delete
- **Impact**: Removes ~118 tokens describing standard Next.js App Router layout that Claude reads directly from the filesystem.

### WARNING

**[W-001]**: Contradiction between test comprehensiveness and PR size limits
- **Severity**: WARNING
- **Location**: `CLAUDE.md:109` vs `CLAUDE.md:117`
- **Evidence**: > "Write comprehensive test suites for all new features" vs "each PR should change fewer than 50 lines of code when possible"
- **Proposed**: `Keep PRs focused on a single concern. Comprehensive tests are expected and don't count toward PR size.`
- **Impact**: Resolves contradictory instructions that force Claude to choose which rule to violate.

**[W-002]**: Semantic duplicate — code style instructions repeated 3 times
- **Severity**: WARNING
- **Location**: `CLAUDE.md:5-7`, `CLAUDE.md:50-52`, `CLAUDE.md:56-57`
- **Evidence**: > "Write clean, readable, and maintainable code" (line 5), "Write clean, readable code with meaningful variable and function names" (line 50), "Always follow the existing code style in the project" (line 56)
- **Proposed**: delete
- **Impact**: Removes three variations of "write good code" that are both duplicated and default behavior.

**[W-003]**: Component file structure template is over-specified
- **Severity**: WARNING
- **Location**: `CLAUDE.md:75-92`
- **Evidence**: > Full TypeScript code block showing component file structure with import ordering
- **Proposed**: delete
- **Impact**: Removes ~92 tokens describing standard React conventions Claude already follows and infers from existing components.

### INFO

**[I-001]**: API status code list is standard HTTP knowledge
- **Severity**: INFO
- **Location**: `CLAUDE.md:99-106`
- **Evidence**: > List of HTTP status codes (200, 201, 400, 401, 403, 404, 500)
- **Proposed**: `Return proper HTTP status codes. Wrap handlers in try/catch with structured error responses.`
- **Impact**: Replaces ~50 tokens of standard HTTP semantics with a concise actionable instruction.

**[I-002]**: Environment variable list could be a pointer
- **Severity**: INFO
- **Location**: `CLAUDE.md:131-138`
- **Evidence**: > Full list of 7 environment variables with descriptions
- **Proposed**: `See .env.example for required variables. Key services: PostgreSQL, Redis, Stripe, S3.`
- **Impact**: Eliminates ~60 tokens of duplication with `.env.example` and removes a second place to maintain env var changes.

## Adversarial Review

*Skipped — no external agents (Codex CLI, Gemini CLI) detected.*

## Proposed Optimized CLAUDE.md

**Original**: ~3,850 tokens / 155 lines
**Optimized**: ~950 tokens / 38 lines
**Reduction**: 75% tokens / 75% lines

---

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

---

### Extracted Rules (suggested .claude/rules/ files)

No extraction needed — the optimized file is under 40 lines. For larger projects, consider:

```yaml
# .claude/rules/prisma.md
---
paths: ["prisma/**", "src/lib/db/**"]
---
Use `select` to limit fields. Use transactions for multi-record writes.
Prefer separate queries over deep `include` nesting.
```

Review the 7 findings above. You can apply the optimized version, cherry-pick specific finding IDs, or keep your current file.
