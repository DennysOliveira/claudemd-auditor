# Multi-Agent Adversarial Review

The CLAUDE.md Context Auditor can optionally leverage external AI agents as adversarial critics to validate its findings. This document explains how the multi-agent review works, its protocol, and graceful degradation when external agents aren't available.

## Why Adversarial Review?

A single model auditing a CLAUDE.md file has blind spots. The auditor might:
- Flag a context-specific instruction as "default behavior" when it actually overrides a project-specific default
- Over-compress distinct concepts that share surface similarity
- Remove instructions that carry subtle but important intent

By dispatching the original and proposed optimization to independent models, we get competing perspectives that catch these errors.

## Agent Detection

The auditor checks for two external agents during Phase 5:

### Codex CLI (OpenAI)

```bash
command -v codex &>/dev/null && echo "CODEX_AVAILABLE=true"
[ -n "$OPENAI_API_KEY" ] && echo "OPENAI_KEY=set"
```

Codex CLI must be installed and `OPENAI_API_KEY` must be set. The auditor uses `codex exec` to run non-interactive prompts.

### Gemini CLI (Google)

```bash
command -v gemini &>/dev/null && echo "GEMINI_AVAILABLE=true"
[ -n "$GEMINI_API_KEY" -o -n "$GOOGLE_API_KEY" ] && echo "GEMINI_KEY=set"
```

Gemini CLI must be installed with either `GEMINI_API_KEY` or `GOOGLE_API_KEY` set. The auditor uses `gemini -p` for prompt-based invocation.

## The Review Protocol

### Step 1: Prepare Review Materials

The auditor writes two temporary files:
- `/tmp/claudemd_original.md` — the original CLAUDE.md content
- `/tmp/claudemd_proposed.md` — the auditor's proposed optimized version

### Step 2: Dispatch Review Prompts

Each available agent receives an identical adversarial prompt instructing it to:

1. Compare the original and proposed versions
2. Assume the optimizer was too aggressive
3. Find specific flaws in the optimization
4. Cite line numbers and explain risks

The prompt is designed to elicit harsh criticism — the adversarial framing intentionally biases reviewers toward finding problems, which counterbalances the auditor's bias toward cutting content.

### Step 3: Capture Output via File Redirection

Output is captured to temporary files:
- `/tmp/codex_review.txt`
- `/tmp/gemini_review.txt`

**Why file redirection?** Claude Code's subprocess execution can suppress stdout in certain contexts. By redirecting to files and reading them back, we ensure reliable output capture regardless of shell behavior.

### Step 4: Incorporate Feedback

The auditor reads each review and classifies every criticism as:
- **ACCEPTED**: The criticism is valid — revise the proposed CLAUDE.md
- **REJECTED**: The criticism is invalid — explain why with evidence
- **PARTIAL**: Part of the critique is valid — incorporate selectively

## The Five Adversarial Critique Dimensions

External reviewers are asked to evaluate across five specific dimensions:

### 1. Lost Intent

Did the optimization remove instructions that carried unique, non-default value?

**What to look for**: Instructions that appear generic but carry project-specific weight. For example, "Use parameterized queries" might seem redundant in a Prisma project (Prisma handles this automatically), but if the project also has raw SQL in migration scripts, the instruction serves a purpose.

### 2. Over-Compression

Were distinct concepts merged in a way that loses nuance?

**What to look for**: Two instructions compressed into one where the original distinction mattered. E.g., "Use Vitest for unit tests and Playwright for e2e" compressed to "Use Vitest" loses the e2e testing requirement.

### 3. False Positives

Were instructions flagged as "default behavior overlap" that actually DO change Claude's behavior in this specific codebase context?

**What to look for**: This is the most common source of adversarial disagreement. Claude's "default behavior" is context-dependent — what's default in a greenfield project might not be default when existing code establishes a different pattern.

### 4. Missing Context

Does the optimized version lose project-specific context that Claude can't infer from reading code alone?

**What to look for**: Deployment constraints, team conventions, regulatory requirements, performance budgets — things that aren't expressed in code but shape how code should be written.

### 5. Scoring Disagreement

Would the reviewer score any dimension differently? Why?

**What to look for**: Calibration differences. One model might consider 2000 tokens "acceptable" while another considers it "bloated." These disagreements help calibrate the scoring bands.

## Consensus Matrix

After collecting reviews, the auditor builds a consensus matrix:

```markdown
| Finding | Claude | Codex | Gemini | Consensus | Action |
|---------|--------|-------|--------|-----------|--------|
| Remove "Be helpful" | Agree | Agree | Agree | 3/3 HIGH | Keep finding |
| Remove env var list | Agree | Agree | Disagree | 2/3 MED | Keep, note dissent |
| Merge test commands | Agree | Disagree | Disagree | 1/3 LOW | Flag for user review |
```

### Confidence Levels

| Agreement | Confidence | What it means |
|-----------|------------|---------------|
| 3/3 | HIGH | Strong consensus — finding is reliable |
| 2/3 | MEDIUM | Majority agreement — finding stands with noted dissent |
| 1/3 | LOW | Single-model finding — flagged for user review, may be a false positive |
| External-only | REVIEW | Found by external agent but not by Claude — added as supplementary finding |

### How Confidence Affects the Report

- **HIGH confidence** findings are included in the main findings section without qualification
- **MEDIUM confidence** findings are included with a note about the dissenting opinion
- **LOW confidence** findings are moved to an "Under Review" subsection with the recommendation that the user verify manually
- **REVIEW** findings from external agents are listed separately as "Additional Considerations"

## Graceful Degradation

The adversarial review is **optional**. The auditor functions fully without any external agents:

| Agents Available | Behavior |
|-----------------|----------|
| Both Codex + Gemini | Full adversarial review with 3-model consensus |
| One agent only | Partial review with 2-model consensus |
| No agents | Adversarial review section notes "Skipped — no external agents detected" |

When no agents are available:
- All findings are reported at their face value
- The report notes that adversarial validation was not performed
- Users are encouraged to manually review findings flagged as potentially aggressive

## Known Issues and Workarounds

### Claude Code stdout Suppression

In some Claude Code versions, subprocess stdout may be suppressed or truncated. The auditor works around this by:
1. Redirecting all external agent output to files (`> /tmp/codex_review.txt 2>&1`)
2. Reading the files back with `cat`
3. Checking file existence before reading (`cat /tmp/codex_review.txt 2>/dev/null || echo "No review available"`)

### Agent Timeout

External agents may take 30-60 seconds to respond. The auditor does not set explicit timeouts — it relies on the shell's default behavior. If an agent hangs, the user can interrupt with Ctrl+C and the audit will continue without that agent's review.

### Prompt Length Limits

For very large CLAUDE.md files (8000+ tokens), the combined review prompt (original + proposed + instructions) may approach some agents' input limits. The auditor mitigates this by sending the full files rather than truncating — if an agent can't handle the input, it will error gracefully and the review proceeds without it.

## Security Considerations

- Temporary files (`/tmp/claudemd_*.txt`) contain your CLAUDE.md content. The auditor does not clean these up automatically — they persist until system temp cleanup or manual deletion.
- Review prompts are sent to external AI providers (OpenAI for Codex, Google for Gemini). Your CLAUDE.md content will be processed by these services according to their respective data policies.
- No data is sent to any service unless the corresponding CLI tool is both installed and authenticated. The auditor never prompts for API keys.
