# AutoDCF

An AI skill that builds institutional-grade DCF valuation models in Excel through deterministic steps with per-step adversarial audits.

## What It Does

Give it a company ticker — it produces a complete DCF valuation model:

- **4-tab workbook** (Cover, Outputs, Inputs, Model) following Financial Modeling Guidelines
- **11 operational schedules** stacked in Model tab (Revenue through DCF Multiple)
- **Dual UFCF** with algebraic reconciliation (EBITDA method + Net Income method)
- **8 sensitivity tables** with per-cell re-discounting
- **Every number traceable** to audited filings via CSV data pipeline with source quotes

## Architecture (v4)

```
Orchestrator (zero-knowledge, stateless)
  │
  ├── Phase 1: Data Acquisition (7 sub-steps, parallel)
  │   └── dcf-researcher agents
  │       → CSV files with 13-column provenance + raw artifacts
  │
  ├── Phase 2: Model Build (18 steps, strictly sequential)
  │   └── For EACH step:
  │       ├── dcf-builder (writes Python module, runs build)
  │       ├── dcf-auditor (numbered checklist audit)
  │       └── dcf-adversarial (free-form hunt)
  │       └── 100/100 gate before next step
  │
  └── Phase 3: Integration Audit
      └── Cross-step references, BS balance, dual UFCF tie-out
```

### Key Design Principles

- **Zero hardcoded data** — everything from CSV with source_doc, source_page, source_quote
- **Zero-knowledge orchestrator** — passes only file paths, never reads data
- **Structural anti-batching** — each agent gets ONE step's spec, cannot skip ahead
- **Dual audit every step** — checklist + adversarial, fresh agent each time
- **Guidelines over template** — follows published Financial Modeling Guidelines standard, not just the template implementation
- **Deterministic data pipeline** — Python scripts copy CSV to hidden Data sheets (not agents)

## Font Color Convention

| Cell Type | Color | Hex |
|-----------|-------|-----|
| Hardcoded inputs | Blue | `FF3271D2` |
| Same-sheet formulas | Black | `FF000000` |
| Cross-sheet links | Green | `FF006100` |

## Model Structure (11 Schedules)

| # | Schedule | Model Rows |
|---|----------|-----------|
| 1 | Revenue Schedule | 3–44 |
| 2 | Cost Schedule | 47–90 |
| 3 | Income Statement | 93–124 |
| 4 | Working Capital Schedule | 127–159 |
| 5 | Depreciation Schedule | 162–205 |
| 6 | Asset Schedule | 208–234 |
| 7 | Income Tax Schedule (Levered) | 237–276 |
| 8 | Income Tax Schedule (Unlevered) | 279–318 |
| 9 | Unlevered Free Cash Flow Schedule | 321–358 |
| 10 | DCF: Perpetuity Method | 361–405 |
| 11 | DCF: Multiple Method | 408–452 |

## Repo Structure

```
auto-dcf/
├── SKILL.md                    # Orchestrator spec (Agent Skills standard)
└── references/
    ├── formatting_rules.md     # Extraction-verified formatting specs
    ├── known_mistakes.md       # 25 catalogued bugs from 8+ audit rounds
    ├── step_index.md           # Step-to-file mapping
    └── steps/                  # Per-step build specs with audit checklists
        ├── step_00_research.md
        ├── step_00a–00g_*.md   # Data acquisition sub-steps
        ├── step_01_skeleton.md # Workbook skeleton (v4 complete)
        ├── step_02–18_*.md     # Build steps (v4 in progress)
        └── ...
```

## Installation

```bash
# Clone to personal skills directory
git clone https://github.com/Siriko404/auto-dcf.git ~/.claude/skills/auto-dcf
```

## Usage

```
/auto-dcf <ticker>
```

## Version History

| Version | Score | Key Change |
|---------|-------|------------|
| v3 | 93/100 | Per-step specs, modular Python, CSV pipeline |
| v4 | TBD | Extraction-verified formatting, guidelines-first conventions, 25 known mistake guards |

## Platform

Built on the [Agent Skills](https://agentskills.io) open standard for Claude Code.

## License

Private. Not for redistribution.
