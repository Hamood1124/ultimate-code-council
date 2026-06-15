---
name: refactor
description: "Safe refactoring assistant. Reads code, proposes specific refactoring moves, asks which ones you want, applies them, then auto-runs the council to confirm nothing broke. TRIGGER on: 'refactor this', 'clean up this code', 'improve this code', 'this is messy', 'simplify this', '/refactor'. Never refactors without explicit approval per move. Always runs council after to confirm no regressions."
---

# Refactor

Safe, proposal-first refactoring. Never changes code without showing you exactly what it plans to do and getting a yes first. Runs the council after every refactor to confirm nothing broke.

---

## Phase 0 — Load context (silently)

```bash
cat CONTEXT.md 2>/dev/null

# Run tests before touching anything — establish baseline
npx jest --passWithNoTests 2>/dev/null | tail -5
pytest -q 2>/dev/null | tail -5
dotnet test 2>/dev/null | tail -5

# Check for TypeScript errors as baseline
npx tsc --noEmit 2>/dev/null | head -20
```

**If tests fail before refactoring starts** → stop and tell user:
> *"Tests are already failing before any refactoring. Fix the failing tests first so we have a clean baseline to work from."*

---

## Phase 1 — Scope the request

```
What do you want to refactor?

A) Specific file(s) — tell me which
B) Specific function or class — paste it or tell me where
C) The whole codebase — I'll identify the highest-impact areas
D) Something specific — e.g. "remove all the duplication", "simplify the conditionals"
```

---

## Phase 2 — Identify and propose moves

Read the target code. Identify refactoring opportunities. Present as a numbered proposal — never apply anything yet:

```
Here are the refactoring moves I'd suggest:

1. [EXTRACT FUNCTION] — Lines 45-67 in LeaveService.ts
   The approval logic is duplicated in 3 places. Extract to `validateApprovalChain()`.
   Impact: Medium | Risk: Low | Tests needed: existing ones cover this

2. [SIMPLIFY CONDITIONAL] — Lines 89-103 in LeaveService.ts
   5-level nested if/else. Reduce to early returns.
   Impact: Low | Risk: Very Low | Tests: no change needed

3. [REMOVE DUPLICATION] — LeaveForm.tsx and LeaveHistory.tsx
   Same date formatting logic in both. Extract to `utils/formatLeaveDate.ts`.
   Impact: Low | Risk: Low | Tests: add one utility test

4. [RENAME] — variable `data` on line 34 of LeaveService.ts
   Rename to `leaveRequestResponse` per CONTEXT.md domain vocabulary.
   Impact: Very Low | Risk: Very Low | Tests: no change needed

Which would you like me to apply? Reply with numbers (e.g. "1, 3") or "all".
```

Wait for explicit selection. Never apply all by default.

---

## Phase 3 — Apply selected moves

For each approved move:
1. Show exactly what will change (before/after for key lines)
2. Apply the change
3. Run tests immediately after each move

```bash
# After each individual refactor move
npx jest --passWithNoTests 2>/dev/null | tail -5
npx tsc --noEmit 2>/dev/null | head -10
```

If tests break after a move:
> *"Tests broke after [move name]. Reverting this change and flagging for review."*
Then revert and continue with the next approved move.

---

## Phase 4 — Council verification

After all approved moves are applied and tests are passing:

> *"Refactoring complete. Running the Maintainability Critic and Correctness Judge to confirm quality improved and nothing broke."*

Run a scoped council (Correctness Judge + Maintainability Critic only — skip Security, Performance, Requirements for pure refactors) on changed files.

```markdown
## Refactor Summary

Moves applied: [N of N selected]
Tests: ✅ All passing
TypeScript: ✅ Clean

### Quality delta
Before: [issues from Maintainability Critic pre-refactor]
After: [issues post-refactor]

### Net improvement
[Plain English summary of what's cleaner now]
```

> *"Refactor complete and verified. Commit these changes before your next feature — mixing refactoring with feature work makes PRs harder to review."*
