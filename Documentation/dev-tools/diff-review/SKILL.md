---
name: diff-review
description: "Reviews only what changed between two branches or commits — not the whole codebase. Runs the full six-reviewer council and security layer scoped exclusively to changed files. Massively token-efficient for day-to-day PR reviews. TRIGGER on: 'review this PR', 'review my changes', 'diff review', 'what changed', 'review before merge', '/diff-review'. Requires git."
---

# Diff Review

Reviews only changed files — not the whole codebase. Same six-reviewer quality and security checks as `/code-council` but scoped to the diff. The most token-efficient way to review code day-to-day.

---

## Phase 0 — Get the diff (silently)

```bash
# Check git status
git rev-parse HEAD 2>/dev/null || echo "NO_GIT"

# Get changed files vs main/master
git diff --name-only main 2>/dev/null || git diff --name-only master 2>/dev/null

# Get the actual diff
git diff main 2>/dev/null || git diff master 2>/dev/null

# If no branch comparison, fall back to staged/unstaged changes
git diff --cached 2>/dev/null
git diff 2>/dev/null

# Get commit messages for context
git log main..HEAD --oneline 2>/dev/null || git log -5 --oneline 2>/dev/null
```

If no git → tell user:
> *"/diff-review requires git. Run `git init` and make at least one commit first, or use `/code-council` to review files directly."*

---

## Phase 1 — Scope confirmation

Present what was found:

```
I found [N] changed files:
- src/services/LeaveService.ts (+45, -12 lines)
- src/components/LeaveForm.tsx (+89, -3 lines)
- api/routes/leave.ts (+34, -0 lines)

Review all of these, or specific files only?
Also: Quick / Standard / Deep? (default Standard)
```

Wait for confirmation.

---

## Phase 2 — Run scoped council

Run the same six reviewers as `/code-council` but **only on changed files and only on changed lines** where possible.

```bash
# Load context
cat CONTEXT.md 2>/dev/null
cat SECURITY.md 2>/dev/null
```

For each changed file:
- Read the full file for context
- Focus reviewer attention on the changed lines specifically
- Flag issues introduced by the changes — not pre-existing issues in unchanged code

**Important:** If a change in File A introduces a problem in unchanged File B (e.g. a function signature change breaks a caller), flag it — even though File B isn't in the diff.

Run security layer automatically if changed files touch auth/API/data — same auto-trigger rules as `/code-council`.

---

## Phase 3 — Scoped report

Same format as `/code-council` but prefixed with diff context:

```markdown
## 🔀 Diff Review — [branch] → main

**Changed files:** N
**Lines added:** +XXX  |  **Lines removed:** -XXX
**Depth:** Standard

[Normal reviewer output with CC-IDs...]

## 🏛️ Diff Review Verdict

**Verdict:** SHIP IT / SHIP WITH FIXES / DO NOT SHIP

> *Note: This review covers only changed files. Pre-existing issues
> in unchanged code are not included. Run /code-council for a full review.*
```

---

## Phase 4 — Fix offer

Same as `/code-council` — offer to fix Blockers and Highs directly in repo, delta review after.
