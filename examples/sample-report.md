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

**C-001**: Default behavior restatement block wastes ~180 tokens
- **Location**: CLAUDE.md:5-7
- **Evidence**: `"Be helpful, accurate, and concise in all your responses. Always strive to provide the best possible assistance... Write clean, readable, and maintainable code at all times. Follow best practices for software development. Think step by step when solving complex problems."`
- **Impact**: These are Claude's built-in behaviors. Restating them consumes tokens and adds noise to the instruction set, diluting actually unique instructions. "Think step by step" is redundant with extended thinking.
- **Fix**:
  - Before (46 tokens): The entire "General Behavior" section
  - After (0 tokens): Delete entirely
  - Savings: 46 tokens

**C-002**: Verbose TypeScript section could be a one-liner
- **Location**: CLAUDE.md:34-42
- **Evidence**: `"When writing TypeScript code, please ensure that you always use strict mode. TypeScript's strict mode helps catch potential errors at compile time..."`
- **Impact**: 150+ tokens explaining what TypeScript strict mode is. Claude knows what strict mode is. The only actionable instruction is "use strict mode, no `any`."
- **Fix**:
  - Before (156 tokens): Two full paragraphs about TypeScript usage
  - After (14 tokens): `TypeScript strict mode, no 'any' — use 'unknown' or generics instead`
  - Savings: 142 tokens

**C-003**: Project structure section duplicates what `ls` reveals
- **Location**: CLAUDE.md:14-27
- **Evidence**: The full directory tree block listing `src/app`, `src/components`, `src/lib`, etc.
- **Impact**: Claude reads the actual directory structure. Describing standard Next.js App Router layout wastes ~120 tokens. Only non-obvious structural decisions add value.
- **Fix**:
  - Before (118 tokens): Full directory tree with descriptions
  - After (0 tokens): Delete entirely — structure is standard Next.js and self-evident
  - Savings: 118 tokens

### WARNING

**W-001**: Contradiction between test comprehensiveness and PR size limits
- **Location**: CLAUDE.md:109 vs CLAUDE.md:117
- **Evidence**: `"Write comprehensive test suites for all new features"` vs `"each PR should change fewer than 50 lines of code when possible"`
- **Impact**: These instructions conflict — comprehensive test suites for features routinely exceed 50 lines. Claude must choose which to follow, leading to inconsistent behavior.
- **Fix**: Remove the 50-line PR limit or reframe: `Keep PRs focused on a single concern. Comprehensive tests are expected and don't count toward PR size.`

**W-002**: Semantic duplicate — code style instructions repeated 3 times
- **Location**: CLAUDE.md:5-7, CLAUDE.md:50-52, CLAUDE.md:56-57
- **Evidence**:
  - `"Write clean, readable, and maintainable code"` (line 5)
  - `"Write clean, readable code with meaningful variable and function names"` (line 50)
  - `"Always follow the existing code style in the project"` (line 56)
- **Impact**: Three variations of "write good code / follow style." Each one dilutes the instruction set. Claude already matches surrounding code style.
- **Fix**: Delete all three. They are both duplicated and default behavior.

**W-003**: Component file structure template is over-specified
- **Location**: CLAUDE.md:75-92
- **Evidence**: Full TypeScript code block showing component file structure
- **Impact**: ~90 tokens to describe standard React conventions that Claude already follows. The import ordering and component structure shown are Claude's defaults.
- **Fix**:
  - Before (92 tokens): Full code example
  - After (0 tokens): Delete — Claude infers patterns from existing components
  - Savings: 92 tokens

### INFO

**I-001**: API status code list is standard HTTP knowledge
- **Location**: CLAUDE.md:99-106
- **Evidence**: List of HTTP status codes (200, 201, 400, 401, 403, 404, 500)
- **Impact**: ~50 tokens describing standard HTTP semantics Claude already knows. Only keep if the project has non-standard status code usage.
- **Fix**: Replace with `Return proper HTTP status codes. Wrap handlers in try/catch with structured error responses.`

**I-002**: Environment variable list could be a pointer
- **Location**: CLAUDE.md:131-138
- **Evidence**: Full list of 7 environment variables with descriptions
- **Impact**: ~60 tokens duplicating what exists in `.env.example`. Changes to env vars must be updated in two places.
- **Fix**: Replace with `See .env.example for required variables. Key services: PostgreSQL, Redis, Stripe, S3.`

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
