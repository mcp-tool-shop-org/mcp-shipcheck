# Adopting Shipcheck

How to apply the Ship Gate to any MCP Tool Shop repo in under 30 minutes.

---

## Before you start

Know your repo type. The Ship Gate uses applicability tags:

| Tag | You are this if... |
|-----|--------------------|
| `[all]` | Every repo |
| `[npm]` | Published to npm |
| `[pypi]` | Published to PyPI |
| `[vsix]` | VS Code extension |
| `[desktop]` | Desktop app (WinUI, Electron, MAUI) |
| `[container]` | Ships a Docker image |
| `[mcp]` | MCP server |
| `[cli]` | CLI tool |
| `[complex]` | Has background daemons, state files, or operational modes |

Most repos hit 2-3 tags. A typical MCP server is `[all]` + `[npm]` + `[mcp]` + `[cli]`.

---

## Step-by-step (~30 minutes)

### 1. Copy the gate (2 min)

Copy `templates/SHIP_GATE.md` into your repo root.

Scan through and mark items that don't apply to your repo type with `SKIP:`:

```
- [ ] `[pypi]` SKIP: not a Python project
```

### 2. Security baseline (5 min)

Copy `templates/SECURITY.md` into your repo root and fill in:
- Supported versions table
- Scope section (what data is touched, what isn't)

Add a threat model paragraph to your README:

```markdown
## Trust model

**Data touched:** local log files, process metrics
**No network egress** unless explicitly configured
**No secrets handling** — does not read, store, or transmit credentials
**No telemetry** collected or sent
```

Check off: A1, A2, A3, A4.

### 3. Error shape (10 min)

**For simple scripts:** Make sure errors emit the shape (code, message, hint) as a dict or JSON object. That's it. Check off B1.

**For CLI/MCP/desktop:** Implement a product error class following the [error contract](contracts/error-contract.md). Reference implementations exist for TypeScript, Python, and .NET. Check off B1 through B5 (whichever apply).

### 4. Operator docs (5 min)

- [ ] README current? If not, update install + usage sections.
- [ ] CHANGELOG.md exists? If not, copy `templates/CHANGELOG.md`.
- [ ] LICENSE exists? If not, add MIT.
- [ ] For CLI: verify `--help` output matches actual commands.
- [ ] For complex tools: copy `templates/HANDBOOK.md` and fill in the sections.

Check off: C1 through C7 (whichever apply).

### 5. Shipping hygiene (5 min)

- [ ] Add a `verify` script to package.json / Makefile / pyproject.toml:
  ```json
  "scripts": {
    "verify": "npm run test && npm run build && npm pack --dry-run"
  }
  ```
- [ ] Run `verify` and fix anything it catches.
- [ ] Confirm dependency scanning exists in CI (any mechanism).

Check off: D1 through D9 (whichever apply).

### 6. Review the gate (3 min)

Open your `SHIP_GATE.md`. Every line should be either:
- `[x]` checked with a date
- `SKIP:` with a justification
- `[ ]` unchecked (you're not done yet)

If all hard gates (A-D) pass: you're clear to tag and publish.

---

## What "done" looks like

```
SHIP_GATE.md    — all hard gates checked or skipped with justification
SECURITY.md     — filled in with real scope
CHANGELOG.md    — at least one entry
LICENSE         — exists
README.md       — includes trust model paragraph
errors.ts/.py   — follows the error shape (Tier 1 minimum)
```

## What "whole" looks like (soft gates too)

All of the above, plus:
- Logo in README
- 8-language translations
- Landing page (org repos)
- GitHub metadata set

---

## FAQ

**Do I need a HANDBOOK.md?**
Only if your tool has operational complexity (background daemons, state files, attention levels, recovery procedures). Simple CLIs and libraries don't need one.

**What if my repo doesn't ship anywhere?**
Skip section D entirely with a note. The security and error handling sections still apply to anything that runs code.

**Can I add custom gate items?**
Yes. Add them to a "## F. Project-Specific" section at the bottom of your SHIP_GATE.md. Don't modify sections A-E — those are the org standard.
