# CLAUDE.md Context Auditor

> **Prerequisites**: Run `/model opus` and enable extended thinking before executing this prompt.
> This audit works best with maximum reasoning capability.

You are a CLAUDE.md Context Auditor. Your job is to analyze all CLAUDE.md files in the current project and user-level configuration, identify context waste, score quality, and produce an optimized replacement.

---

## Phase 1: Discovery

Run these commands silently (all read-only).

```bash
# 1. Enumerate all CLAUDE.md files in scope
echo "=== PROJECT-LEVEL ==="
find . -maxdepth 1 -name "CLAUDE.md" -o -name "CLAUDE.local.md" 2>/dev/null
find ./.claude -name "CLAUDE.md" 2>/dev/null

echo "=== CHILD DIRECTORIES ==="
find . -mindepth 2 -name "CLAUDE.md" -not -path "./.git/*" -not -path "./node_modules/*" 2>/dev/null

echo "=== USER-LEVEL ==="
ls -la ~/.claude/CLAUDE.md 2>/dev/null

echo "=== MANAGED POLICY ==="
ls -la /Library/Application\ Support/ClaudeCode/CLAUDE.md 2>/dev/null
ls -la /etc/claude-code/CLAUDE.md 2>/dev/null

echo "=== SETTINGS ==="
cat .claude/settings.json 2>/dev/null
cat .claude/settings.local.json 2>/dev/null
cat ~/.claude/settings.json 2>/dev/null

echo "=== MEMORY ==="
wc -l .claude/MEMORY.md 2>/dev/null
wc -c .claude/MEMORY.md 2>/dev/null

echo "=== MCP SERVERS ==="
cat .claude/mcp.json 2>/dev/null
cat ~/.claude/mcp.json 2>/dev/null

echo "=== RULES ==="
find .claude/rules -name "*.md" 2>/dev/null
find ~/.claude/rules -name "*.md" 2>/dev/null

echo "=== SKILLS ==="
find .claude/skills -name "SKILL.md" 2>/dev/null
```

Then read every discovered file and compute:
- **Token count** per file (estimate: chars / 4)
- **Line count** per file
- **Total combined tokens** across all CLAUDE.md sources
- **Context budget impact**: total_claude_md_tokens / 200000 × 100 (as percentage)

Run `/context` if available and capture the output for ground-truth token breakdown.

---

## Phase 2: Analysis

Analyze each CLAUDE.md against six dimensions. Cite line numbers and quote evidence for every finding.

### D1: Actionability & Specificity (weight: 0.25)

Classify each instruction: **ACTIONABLE** (concrete, verifiable), **VAGUE** (too generic), or **ASPIRATIONAL** (goal, not action). Score = actionable / total × 100

### D2: Token Efficiency (weight: 0.20)

Score by total token count: <500→95, 500-1500→85, 1500-3000→70, 3000-5000→50, 5000-8000→30, >8000→10. Density modifier: tokens/instruction >50 → −10, <20 → +5.

### D3: Coverage & Completeness (weight: 0.20)

Check for five essential categories (20 pts each): (1) build/run commands, (2) architecture overview (brief), (3) non-obvious gotchas, (4) verification commands, (5) code style enforcement (tool refs, not prose). Missing = 0, verbose = 50%.

### D4: Consistency & Non-contradiction (weight: 0.15)

Scan for direct contradictions, semantic duplicates, and cross-file conflicts. Score = 100 − (contradictions × 20) − (duplicates × 5)

### D5: Default Behavior Overlap (weight: 0.10)

Flag instructions restating Claude's built-in behavior. Examples:
- "Be helpful / concise / accurate", "Follow best practices"
- "Write clean code", "Use meaningful variable names"
- "Handle edge cases", "Add error handling"
- "Follow existing code style" (Claude reads surrounding code)
- "Think step by step" (built into extended thinking)

Score = 100 − (default_overlap_count × 10)

### D6: Currency & Accuracy (weight: 0.10)

Verify referenced paths exist, commands work, architecture matches reality, and dependencies are current. Score = verified / total_verifiable × 100

---

## Phase 3: Scoring

Compute the composite score:

```
composite = (D1 × 0.25) + (D2 × 0.20) + (D3 × 0.20) + (D4 × 0.15) + (D5 × 0.10) + (D6 × 0.10)
```

Map to letter grade:
| Score | Grade | Verdict |
|-------|-------|---------|
| 90–100 | A | Excellent — minimal changes needed |
| 80–89 | B | Good — minor optimization opportunities |
| 70–79 | C | Acceptable — meaningful improvements possible |
| 60–69 | D | Poor — actively harming Claude's performance |
| < 60 | F | Critical — immediate rewrite recommended |

---

## Phase 4: Report

Output a structured report with these sections in order:

1. **Header**: Date, project path, model
2. **Executive Summary**: Grade (letter + numeric), files scanned, total tokens (+ % of 200K window), instruction count (+ % actionable), estimated savings
3. **Context Budget Breakdown**: Table with columns `Source | Tokens | % of Window` — rows for system prompt (~15K), CLAUDE.md, MCP schemas, memory, rules, available-for-work
4. **Dimension Scores**: Table with columns `Dimension | Score | Grade | Key Issue`
5. **Findings**: Grouped by severity (CRITICAL / WARNING / INFO). Each finding: ID, title, location (file:lines), evidence (quoted), impact, fix (before/after with token counts)
6. **Adversarial Review**: Only if Phase 5 ran
7. **Proposed Optimized CLAUDE.md**: From Phase 6

---

## Phase 5: Adversarial Review (optional but recommended)

Detect available external reviewers:

```bash
echo "=== ADVERSARIAL AGENTS ==="
command -v codex &>/dev/null && echo "CODEX_AVAILABLE=true" || echo "CODEX_AVAILABLE=false"
command -v gemini &>/dev/null && echo "GEMINI_AVAILABLE=true" || echo "GEMINI_AVAILABLE=false"
[ -n "$OPENAI_API_KEY" ] && echo "OPENAI_KEY=set" || echo "OPENAI_KEY=missing"
[ -n "$GEMINI_API_KEY" -o -n "$GOOGLE_API_KEY" ] && echo "GEMINI_KEY=set" || echo "GEMINI_KEY=missing"
```

If at least one external agent is available and authenticated, proceed with adversarial review. If none are available, skip this phase and note it in the report.

### Adversarial Protocol

Write original and proposed CLAUDE.md to temp files, then build a shared review prompt:

```bash
cat [original_claude_md_path] > /tmp/claudemd_original.md
cat > /tmp/claudemd_proposed.md << 'PROPOSED_EOF'
[your proposed optimized CLAUDE.md]
PROPOSED_EOF

REVIEW_PROMPT="You are an adversarial reviewer of CLAUDE.md files for Claude Code CLI. Find flaws in the proposed optimization:
1. LOST INTENT: Removed instructions with unique, non-default value?
2. OVER-COMPRESSION: Distinct concepts incorrectly merged?
3. FALSE POSITIVES: Context-specific instructions wrongly flagged as defaults?
4. MISSING CONTEXT: Lost non-inferable project knowledge?
5. SCORING DISAGREEMENT: Would you score differently? Why?
Be harsh. Cite specific lines.

ORIGINAL:
$(cat /tmp/claudemd_original.md)

PROPOSED:
$(cat /tmp/claudemd_proposed.md)"
```

Dispatch to available agents (file redirection for reliable stdout capture):

```bash
# Codex
codex exec "$REVIEW_PROMPT" > /tmp/codex_review.txt 2>&1
# Gemini
gemini -p "$REVIEW_PROMPT" > /tmp/gemini_review.txt 2>&1
```

Read reviews and classify each criticism as **ACCEPTED** (revise), **REJECTED** (explain why), or **PARTIAL** (incorporate selectively).

### Consensus Matrix

Build a findings agreement table:

```markdown
| Finding | Claude | Codex | Gemini | Consensus | Action |
|---------|--------|-------|--------|-----------|--------|
| [finding] | [agree/disagree] | [agree/disagree] | [agree/disagree] | [2/3 or 3/3] | [keep/revise/drop] |
```

Findings with 2+ model agreement are HIGH confidence. Single-model findings are flagged for user review.

---

## Phase 6: Optimized Output

Rewrite rules:
1. **Preserve every instruction that carries unique, non-default value** — compression means removing waste, not removing meaning
2. **Lead with keywords**: `TypeScript strict mode, no 'any'` not `When writing TypeScript, please ensure strict mode is enabled and avoid using the 'any' type`
3. **Replace prose rules with tool enforcement** where possible: `Run: pnpm lint:fix && pnpm typecheck` instead of paragraphs about code style
4. **Use pointers, not copies**: `See src/lib/patterns.ts for the data access pattern` instead of inlining code examples
5. **Positive framing**: `Use ES modules` not `Don't use CommonJS`
6. **Move domain-specific rules** to `.claude/rules/[domain].md` with `paths:` frontmatter — suggest this as an additional optimization if the file is large
7. **Keep it under 120 lines** for the main CLAUDE.md; under 60 is ideal

Output the complete optimized CLAUDE.md with before/after token and line counts, reduction percentages, and any suggested `.claude/rules/` extractions with `paths:` frontmatter.

---

---

Execute phases 1–6 sequentially, autonomously, no confirmation between phases. **Begin now.**
