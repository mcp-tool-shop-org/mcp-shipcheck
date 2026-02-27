<p align="center">
  <a href="https://opensource.org/licenses/MIT"><img src="https://img.shields.io/badge/License-MIT-yellow.svg" alt="License: MIT"></a>
</p>

Product standards for MCP Tool Shop. Templates, contracts, and adoption guides that define what "done" means before anything ships.

## What's in here

```
templates/
  SHIP_GATE.md      Pre-release checklist (copy into each repo)
  SECURITY.md       Security policy baseline
  HANDBOOK.md       Operational field manual skeleton
  CHANGELOG.md      Keep a Changelog starter
  SCORECARD.md      Pre/post remediation scoring (internal)

contracts/
  error-contract.md Structured error standard (2-tier)

ADOPTION.md         How to apply shipcheck to a repo in <30 minutes
```

## Quick start

1. Read [ADOPTION.md](ADOPTION.md)
2. Copy `templates/SHIP_GATE.md` into your repo root
3. Check off applicable items, mark non-applicable with `SKIP:`
4. Ship when all hard gates pass

## Philosophy

- **Hard gates** (Security, Error Handling, Operator Docs, Shipping Hygiene) block release
- **Soft gates** (Identity) don't block release but define "whole"
- The gate says **what** must be true, not **how** to implement it
- Applicability tags prevent checkbox shame on repos where items don't apply

## Standards

| Standard | Covers |
|----------|--------|
| [Ship Gate](templates/SHIP_GATE.md) | 27 hard-gate + 4 soft-gate checks across all repo types |
| [Error Contract](contracts/error-contract.md) | Tier 1: error shape (all repos) / Tier 2: base type + exit codes (CLI/MCP/desktop) |
| [Security Baseline](templates/SECURITY.md) | Report email, response timeline, threat scope |
| [Handbook](templates/HANDBOOK.md) | Operational field manual for complex tools |
| [Scorecard](templates/SCORECARD.md) | Pre/post remediation scoring for tracking maturity |

## License

MIT

---

Built by [MCP Tool Shop](https://mcp-tool-shop.github.io/)
