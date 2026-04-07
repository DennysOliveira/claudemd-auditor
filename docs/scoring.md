# Scoring Methodology

This document details how the CLAUDE.md Context Auditor scores configuration files across six dimensions, the research backing each metric, and how composite grades are calculated.

## Research Foundation

The scoring methodology draws on findings from several areas of LLM performance research:

### Context Window Degradation

**MRCR (Multi-Round Coreference Resolution)** studies demonstrate that LLM accuracy degrades as context length increases. Models tested at 128K token contexts showed measurable accuracy drops compared to 32K contexts, even when the relevant information remained in the same relative position. This finding motivates the Token Efficiency dimension — every unnecessary token in CLAUDE.md contributes to a longer effective context for every single API call.

- Hsieh et al., "Ruler: What's the Real Context Size of Your Long-Context Language Models?" (2024) — showed that effective context length is often much shorter than advertised, with performance degradation beginning well before the stated limit.
- Liu et al., "Lost in the Middle: How Language Models Use Long Contexts" (2023) — demonstrated that information placement within context affects retrieval accuracy, with degradation in the middle of long contexts.

### Instruction-Following Decay

**ManyIFEval** and related benchmarks show that instruction-following accuracy degrades as the number of constraints increases — critically, this degradation is **uniform across all instructions**, not just later ones. Adding a 20th instruction doesn't just risk that instruction being ignored; it makes instructions 1 through 19 less likely to be followed as well.

- Zhou et al., "Instruction-Following Evaluation for Large Language Models" (2023) — established baseline instruction-following metrics showing diminishing compliance as constraint count grows.
- Qin et al., "InFoBench: Evaluating Instruction Following Ability in Large Language Models" (2024) — decomposed instructions into atomic criteria and demonstrated that compliance per criterion decreases as total criteria count rises.

### Token Optimization Baselines

Empirical data from production CLAUDE.md files provides calibration points:

- **HumanLayer's CLAUDE.md**: A widely-cited example of an effective configuration file at approximately 60 lines. Demonstrates that even complex projects can be configured concisely.
- **Anthropic's documentation**: Recommends keeping CLAUDE.md files "concise" and notes that the file is "included in the system prompt for every conversation." Anthropic's own guidance suggests treating it like a README — focused on what matters most, not exhaustive documentation.
- **Community analysis (TechLoom, SuperClaude postmortem)**: Case studies of "mega-prompts" and heavily-engineered CLAUDE.md files showed diminishing returns past ~1500 tokens, with some reports of degraded performance from files exceeding 5000 tokens due to instruction conflict and attention dilution.

## The Six Dimensions

### D1: Actionability & Specificity (weight: 0.25)

**What it measures**: The proportion of instructions that are concrete, verifiable, and behavior-changing versus vague, aspirational, or truistic.

**How it's scored**:

Each instruction is classified as:

| Category | Definition | Example |
|----------|-----------|---------|
| **Actionable** | Concrete, verifiable, changes behavior | `Run pnpm typecheck before committing` |
| **Vague** | Too generic to produce consistent behavior | `Ensure code quality` |
| **Aspirational** | Describes a goal, not an action | `Strive for clean architecture` |

Score = (actionable_count / total_instructions) x 100

**Why highest weight (0.25)**: This is the single strongest predictor of CLAUDE.md effectiveness. A file with 10 actionable instructions outperforms a file with 50 vague ones, both in instruction compliance and token efficiency. Vague instructions force the model to interpret intent, leading to inconsistent behavior.

### D2: Token Efficiency (weight: 0.20)

**What it measures**: Whether the CLAUDE.md achieves its goals within a reasonable token budget.

**Scoring bands**:

| Tokens | Score | Assessment |
|--------|-------|------------|
| < 500 | 95 | Excellent — surgical precision |
| 500-1500 | 85 | Good — within recommended range |
| 1500-3000 | 70 | Acceptable — room to trim |
| 3000-5000 | 50 | Bloated — likely degrading performance |
| 5000-8000 | 30 | Critical — causing measurable harm |
| > 8000 | 10 | Emergency — compaction death spiral territory |

**Density modifier**: The raw score is adjusted by tokens-per-instruction density:
- If tokens_per_instruction > 50: apply -10 penalty (verbose instructions)
- If tokens_per_instruction < 20: apply +5 bonus (concise instructions)

**Rationale for bands**: The < 500 band reflects files that contain only essential, non-inferable instructions. The 500-1500 range covers most well-maintained projects. The "compaction death spiral" at 8000+ refers to a pattern where users keep adding instructions to fix problems caused by having too many instructions, creating a feedback loop.

### D3: Coverage & Completeness (weight: 0.20)

**What it measures**: Whether the CLAUDE.md covers the essential categories that Claude genuinely needs, without over-covering categories where inference suffices.

**Five essential categories** (each worth 20 points within this dimension):

1. **Build/run commands**: How to install, build, test, lint. Claude can't reliably infer these from package.json alone — many projects have non-obvious setup steps.
2. **Architecture overview**: Key directories, patterns, tech stack. Brief pointers, not exhaustive descriptions. Score 50% if present but verbose.
3. **Non-obvious gotchas**: Things Claude can't infer from reading code — unusual deployment constraints, known bugs to avoid, domain-specific rules that contradict general best practices.
4. **Verification commands**: How Claude should check its own work. E.g., `Run pnpm typecheck && pnpm lint after changes`.
5. **Code style enforcement**: Via tool references (`biome.json`, `.eslintrc`), not prose rules. Claude reads linter configs automatically; prose style rules add tokens without adding enforcement.

### D4: Consistency & Non-contradiction (weight: 0.15)

**What it measures**: Whether instructions can all be followed simultaneously without conflict.

**Scoring formula**: Score = 100 - (contradictions x 20) - (duplicates x 5)

**Types of inconsistency detected**:

- **Direct contradictions**: Two instructions that cannot both be satisfied. Common example: "Write comprehensive tests" + "Keep PRs under 50 lines." A comprehensive test suite for any non-trivial feature exceeds 50 lines.
- **Semantic duplicates**: The same concept expressed in different words across multiple sections. Each duplicate wastes tokens and creates ambiguity about which formulation is authoritative.
- **Cross-file conflicts**: Instructions in user-level `~/.claude/CLAUDE.md` that conflict with project-level `./CLAUDE.md`. This is especially common with code style and commit message preferences.

**Why contradictions are heavily penalized (20 points each)**: When Claude encounters contradictory instructions, it must silently choose one over the other. This choice is non-deterministic and context-dependent, leading to inconsistent behavior across sessions — the worst possible outcome for a configuration file.

### D5: Default Behavior Overlap (weight: 0.10)

**What it measures**: Instructions that restate Claude's built-in behavior, consuming tokens without changing anything.

**Common default behavior restatements**:

| Instruction | Why it's default |
|-------------|-----------------|
| "Be helpful / concise / accurate" | Core model training objective |
| "Write clean, readable code" | Default code generation behavior |
| "Add error handling" | Claude adds try/catch and validation by default |
| "Follow best practices" | Non-actionable; Claude already applies language best practices |
| "Use TypeScript types properly" | Claude generates typed code in .ts files by default |
| "Write meaningful variable names" | Default naming behavior |
| "Add comments where needed" | Claude adds comments for non-obvious logic by default |
| "Handle edge cases" | Claude considers edge cases during code generation |
| "Follow existing code style" | Claude reads surrounding code and matches patterns |
| "Think step by step" | Built into extended thinking; restating it in the prompt is redundant |

**Scoring**: Score = 100 - (default_overlap_count x 10)

**Important nuance**: Some instructions that *appear* to be default behavior restatements actually carry project-specific weight. For example, "Always use TypeScript strict mode" is not default — Claude will follow the project's tsconfig.json, which might not have strict enabled. The auditor checks whether flagged instructions carry context-specific meaning before scoring.

### D6: Currency & Accuracy (weight: 0.10)

**What it measures**: Whether referenced paths, commands, tools, and dependencies actually exist in the current codebase.

**Verification checks**:
- Do referenced file paths exist? (`find` / `ls`)
- Do referenced commands work? (check `package.json` scripts, Makefile targets)
- Does described architecture match actual directory structure?
- Are referenced tools/dependencies still in `package.json` / lockfile?

**Scoring**: Score = (verified_items / total_verifiable_items) x 100

**Why lowest weight (0.10)**: Stale references are easy to find and fix. They're a maintenance problem, not a fundamental design problem. However, a stale reference that points Claude to a non-existent file can cause wasted tool calls and confusion, so they're still worth flagging.

## Composite Score Calculation

```
composite = (D1 x 0.25) + (D2 x 0.20) + (D3 x 0.20) + (D4 x 0.15) + (D5 x 0.10) + (D6 x 0.10)
```

The weights reflect the relative impact of each dimension on Claude Code's performance:
- **Actionability** (0.25) — most direct impact on instruction compliance
- **Token Efficiency** (0.20) — directly affects context window availability
- **Coverage** (0.20) — ensures essential information is present
- **Consistency** (0.15) — prevents non-deterministic behavior from conflicts
- **Default Overlap** (0.10) — pure waste, but lower impact per instance
- **Currency** (0.10) — maintenance issue, lower immediate impact

## Grade Scale

| Score | Grade | Verdict |
|-------|-------|---------|
| 90-100 | A | Excellent — minimal changes needed |
| 80-89 | B | Good — minor optimization opportunities |
| 70-79 | C | Acceptable — meaningful improvements possible |
| 60-69 | D | Poor — actively harming Claude's performance |
| < 60 | F | Critical — immediate rewrite recommended |

## Severity Categorization

Findings are categorized by their impact on Claude Code's performance:

### CRITICAL

Reserved for issues that demonstrably degrade Claude's effectiveness:
- Direct contradictions between instructions
- Instruction blocks exceeding 100 tokens that could be expressed in under 20
- Default behavior restatements consuming 100+ combined tokens
- Total token count exceeding 5000

### WARNING

Issues that waste meaningful resources or create ambiguity:
- Semantic duplicates (same concept in 2+ places)
- Verbose sections (50-100 tokens reducible to under 20)
- Inferred-from-code instructions (directory structures, standard patterns)
- Individual default behavior restatements

### INFO

Minor optimization opportunities and suggestions:
- Prose that could be a pointer (e.g., env var list vs. "see .env.example")
- Standard knowledge restatements (HTTP status codes, REST conventions)
- Style guidance already enforced by linter configs
- Non-essential nice-to-haves

## How Adversarial Consensus Affects Confidence

When external agents (Codex CLI, Gemini CLI) are available, findings are cross-validated:

| Agreement Level | Confidence | Impact |
|----------------|------------|--------|
| 3/3 models agree | High | Finding stands as-is |
| 2/3 models agree | Medium | Finding stands, minority dissent noted |
| 1/3 (Claude only) | Low | Finding flagged for user review |
| External-only (not flagged by Claude) | Review | Added as supplementary finding |

This multi-agent validation reduces false positives, particularly for D5 (Default Behavior Overlap) where the boundary between "default" and "context-specific" can be subtle.
