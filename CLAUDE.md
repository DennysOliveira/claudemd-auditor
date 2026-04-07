# claudemd-auditor

Context auditor for Claude Code CLAUDE.md files. Detects bloat, redundancy, contradictions, and default-behavior overlap. Produces scored reports with evidence-backed rewrites.

## Project structure

```
├── audit-prompt.md          # Core artifact — the audit prompt users paste into Claude Code
├── CLAUDE.md                # This file
├── README.md                # Public-facing docs with usage, examples, scoring methodology
├── examples/
│   ├── bloated-sample.md    # Example bloated CLAUDE.md for testing
│   ├── optimized-sample.md  # Expected output after audit
│   └── sample-report.md     # Example audit report
├── docs/
│   ├── scoring.md           # Detailed scoring methodology and research backing
│   └── adversarial.md       # How multi-agent review works (Codex/Gemini integration)
└── LICENSE                  # MIT
```

## Key constraints

- `audit-prompt.md` must stay under 3000 tokens — it practices what it preaches
- All research claims must have source attribution in `docs/scoring.md`
- README.md targets non-technical users who just want to copy-paste and run
- Examples must be realistic, not strawmen — use patterns from real GitHub issues

## Commands

Validate token count: `wc -c audit-prompt.md | awk '{print int($1/4), "estimated tokens"}'`
