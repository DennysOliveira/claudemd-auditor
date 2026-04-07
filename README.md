[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](https://github.com/DennysOliveira/claudemd-auditor/pulls)

# claudemd-auditor

**A context auditor for Claude Code CLAUDE.md files.**

Your CLAUDE.md is re-injected into every API call. Research shows that instruction-following accuracy degrades uniformly as instruction count increases — a bloated CLAUDE.md doesn't just waste tokens, it makes Claude worse at following *all* your instructions, including the ones you care about most.

This tool audits your CLAUDE.md files, scores them across six dimensions, and produces an optimized rewrite that preserves intent while cutting the waste.

Zero install. Copy, paste, run.

---

## Quick Start

**1.** Copy the contents of [`audit-prompt.md`](audit-prompt.md)

**2.** In your terminal, set model to Opus with extended thinking:
```
/model opus
```

**3.** Paste the prompt into Claude Code at your project root. The audit runs autonomously — no interaction needed.

That's it. You'll get a scored report and an optimized CLAUDE.md you can copy-paste.

---

## What It Does

The auditor runs six phases autonomously:

| Phase | What happens |
|-------|-------------|
| **Discovery** | Finds all CLAUDE.md files (project, user-level, managed policy), settings, memory, MCP configs, rules |
| **Analysis** | Evaluates every instruction against six scoring dimensions |
| **Scoring** | Computes weighted composite score and letter grade |
| **Report** | Produces structured findings with evidence, impact, and fixes |
| **Adversarial Review** | Cross-validates with Codex CLI and Gemini CLI if available |
| **Optimized Output** | Generates a complete rewritten CLAUDE.md ready to use |

### Scoring Dimensions

| Dimension | Weight | What it catches |
|-----------|--------|----------------|
| Actionability & Specificity | 25% | Vague instructions ("ensure quality") that don't change behavior |
| Token Efficiency | 20% | Bloated files burning context window on every call |
| Coverage & Completeness | 20% | Missing essentials (build commands, gotchas, verification steps) |
| Consistency | 15% | Contradictions and semantic duplicates across files |
| Default Behavior Overlap | 10% | Instructions restating what Claude already does ("write clean code") |
| Currency & Accuracy | 10% | Stale file paths, dead commands, outdated references |

### Grade Scale

| Score | Grade | Meaning |
|-------|-------|---------|
| 90-100 | A | Excellent — minimal changes needed |
| 80-89 | B | Good — minor optimizations available |
| 70-79 | C | Acceptable — meaningful improvements possible |
| 60-69 | D | Poor — actively harming performance |
| < 60 | F | Critical — immediate rewrite recommended |

For the full scoring methodology, including research citations and calibration data, see [docs/scoring.md](docs/scoring.md).

---

## Multi-Agent Adversarial Review

The auditor auto-detects [Codex CLI](https://github.com/openai/codex) and [Gemini CLI](https://github.com/google-gemini/gemini-cli) on your system. If available, it dispatches the original and optimized files to these agents as adversarial critics.

Each external agent reviews for:
- **Lost Intent** — did the optimization cut something important?
- **Over-Compression** — were distinct concepts incorrectly merged?
- **False Positives** — were context-specific instructions wrongly flagged as defaults?
- **Missing Context** — does the rewrite lose non-inferable project knowledge?
- **Scoring Disagreement** — would they grade differently?

Findings with 2+ model agreement are marked HIGH confidence. Single-model findings are flagged for your review.

**This is entirely optional.** The auditor works perfectly without external agents — they just add an extra validation layer. See [docs/adversarial.md](docs/adversarial.md) for details.

---

## Examples

The [`examples/`](examples/) directory contains a complete worked example:

| File | Description | Tokens | Lines |
|------|-------------|--------|-------|
| [`bloated-sample.md`](examples/bloated-sample.md) | Realistic bloated CLAUDE.md | ~3,850 | ~155 |
| [`optimized-sample.md`](examples/optimized-sample.md) | After audit optimization | ~950 | ~38 |
| [`sample-report.md`](examples/sample-report.md) | Full audit report | — | — |

**Result: 75% token reduction** while preserving all genuinely valuable instructions.

---

## FAQ

**Will this delete or modify my CLAUDE.md?**
No. The audit is completely read-only. It reads your files, runs analysis, and outputs a report with a *proposed* rewrite. You decide what to apply.

**What model should I use?**
Opus with extended thinking enabled. The audit requires deep analysis across multiple files and dimensions — Opus produces significantly better results than Sonnet for this task.

**Does it work without Codex/Gemini?**
Yes. External agents are optional adversarial critics. Without them, you get the full audit and optimized output — you just skip the cross-validation step.

**How long does it take?**
Depends on how many CLAUDE.md files you have and whether adversarial review runs. Typically 2-5 minutes for the core audit, plus 1-2 minutes per external agent.

**Can I run it on someone else's project?**
Yes. Clone their repo and run the audit from the project root. It only reads files — nothing is modified.

**What if I disagree with a finding?**
The report includes evidence and reasoning for every finding. Review the evidence, and if the instruction carries value in your specific context, keep it. The auditor optimizes for general best practices — you know your project best.

---

## Research Backing

The scoring methodology is grounded in published research and community findings:

- **Context degradation**: MRCR studies show LLM accuracy drops as context length increases, even for information at consistent positions (Hsieh et al., 2024; Liu et al., 2023)
- **Instruction-following decay**: ManyIFEval benchmarks demonstrate that compliance *per instruction* decreases as total instruction count rises — adding instructions makes *all* instructions less reliable (Zhou et al., 2023; Qin et al., 2024)
- **Token efficiency baselines**: Production CLAUDE.md files from HumanLayer (~60 lines) and Anthropic's own recommendations establish that effective configuration is achievable in well under 1500 tokens
- **Compaction death spirals**: Community case studies document a pattern where users add instructions to fix problems caused by too many instructions, creating a feedback loop of degradation

Full citations and methodology details: [docs/scoring.md](docs/scoring.md)

---

## Contributing

Contributions are welcome! Areas where help is especially valuable:

- **New anti-patterns**: Found a common CLAUDE.md mistake not covered by the six dimensions? Open an issue with examples.
- **Research citations**: Know of published work on LLM instruction-following, context window optimization, or prompt engineering? We'd love to strengthen the evidence base.
- **Example files**: Real-world before/after CLAUDE.md pairs (anonymized) help validate the scoring methodology.
- **Agent integrations**: Support for additional adversarial review agents beyond Codex and Gemini.

### How to contribute

1. Fork the repository
2. Create a feature branch (`feat/your-feature`)
3. Make your changes
4. Submit a pull request with a clear description of the change and its motivation

Please keep `audit-prompt.md` under 3000 tokens — the auditor practices what it preaches.

---

## License

[MIT](LICENSE) - DennysOliveira
