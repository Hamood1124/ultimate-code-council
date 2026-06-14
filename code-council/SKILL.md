---
name: code-council
description: Six-reviewer code quality council for Claude Code. Analyzes any code, file, or full project across correctness, security, performance, maintainability, requirements, and integration disciplines. Produces a structured markdown report with severity-ranked issues (CC-IDs), confidence scores, and a SHIP / SHIP WITH FIXES / DO NOT SHIP verdict. Fixes files directly in the repo — no zips. TRIGGER on: "council this code", "ship check", "code review", "review this", "audit this", "is this safe to ship". Auto-triggered by council-start orchestrator after TDD build phase. Auto-runs on any non-trivial code Claude generates (30+ lines or anything touching auth/data/APIs).
---

# Code Council — Claude Code Edition

Six specialist reviewers analyze submitted code independently from different disciplines. No reviewer tries to be balanced — each only cares about their domain. The synthesis reconciles conflicts and produces a clear verdict. Files are fixed directly in the repo when the user approves.

**Reference files — read when indicated:**

| File | Read when |
|------|-----------|
| `references/reviewers.md` | Before running any reviewer pass |
| `references/depth-modes.md` | When presenting depth menu |
| `references/file-classification.md` | When a project/multi-file submission arrives |
| `references/report-format.md` | When writing the report and verdict |
| `references/delivery.md` | When user approves corrected files |

---

## Phase 0 — Context load (always, silently)

Before anything else:

```bash
# Load domain vocabulary
cat CONTEXT.md 2>/dev/null || cat CONTEXT-MAP.md 2>/dev/null

# Load architectural decisions
ls docs/adr/*.md 2>/dev/null && cat docs/adr/*.md

# Detect project type
cat package.json 2>/dev/null | head -20
cat requirements.txt pyproject.toml 2>/dev/null | head -10
cat *.csproj 2>/dev/null | head -10
```

Use domain vocabulary from `CONTEXT.md` in every reviewer finding. "The `Customer` entity" not "the user object". If no `CONTEXT.md` exists, tell the orchestrator — do not proceed without it (the orchestrator will run `/setup-ashraf-skills`).

---

## Phase 1 — File classification

Read `references/file-classification.md` now.

Run the classification script:

```bash
# Get full file tree
find . -not -path '*/node_modules/*' -not -path '*/.git/*' \
       -not -path '*/vendor/*' -not -path '*/__pycache__/*' \
       -not -path '*/dist/*' -not -path '*/build/*' \
       -not -path '*/.next/*' | sort

# Count lines in source files for scope estimate
find . -name "*.ts" -o -name "*.tsx" -o -name "*.js" -o -name "*.jsx" \
       -o -name "*.py" -o -name "*.cs" -o -name "*.go" \
       | grep -v node_modules | xargs wc -l 2>/dev/null | tail -1
```

Classify every file as Deep / Quick / Skip per the rules in `references/file-classification.md`.

---

## Phase 2 — Depth selection

Read `references/depth-modes.md` now. Present the depth menu to the user and wait for their choice.

If triggered by the orchestrator in auto-mode → default to Standard, skip the menu.

---

## Phase 3 — Run available tooling

Before reviewers run, execute whatever tooling exists in the project:

```bash
# TypeScript
npx tsc --noEmit 2>&1 | head -50

# Linting
npx eslint . --ext .ts,.tsx,.js,.jsx 2>&1 | head -50
npx biome check . 2>&1 | head -50

# Tests
npm test 2>&1 | tail -30
pytest 2>&1 | tail -30
dotnet test 2>&1 | tail -30

# Python type checking
mypy . 2>&1 | head -50

# Security scanning
npx audit 2>&1 | head -30
pip-audit 2>&1 | head -30
```

Capture all output. Pass it to the relevant reviewers as additional signal — tooling output is ground truth, not advisory. A TypeScript error is a Confirmed Blocker, not Suspected.

---

## Phase 4 — Six reviewer passes

Read `references/reviewers.md` now.

Run all six reviewers sequentially using the format from `references/report-format.md`. Each reviewer:
- Uses CONTEXT.md domain vocabulary throughout
- Checks ADRs before flagging architectural issues
- References tooling output where relevant
- Assigns CC-IDs incrementing across all reviewers

---

## Phase 5 — Conflict check + verdict

Read `references/report-format.md` for verdict format.

Run the conflict check pass. Then produce the full council verdict.

---

## Phase 6 — Delivery

Read `references/delivery.md` now.

After verdict, offer to fix. When user agrees → fix files directly in repo using file editing tools. No zips. See delivery reference for exact protocol.

After fixing → run delta review automatically:

```bash
# Check only changed files
git diff --name-only HEAD 2>/dev/null
```

Re-run relevant reviewers on changed files only. Report delta result.
