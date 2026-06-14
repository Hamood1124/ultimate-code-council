# Code Council — Delivery (Claude Code Edition)

In Claude Code, corrected files are written directly to the repo. No zips. No manual copying. The user gets clean fixed files in place.

---

## After verdict — always ask first

```
Should I apply the fixes directly to your files?

I'll fix: CC-001 (🔴 Blocker), CC-002 (🟠 High), CC-003 (🟠 High)
I'll leave for you: CC-005 (🟡 Medium — depends on your business logic)

Reply yes to apply, or tell me which CC-IDs to fix.
```

Never modify files without explicit user confirmation. One exception: if the orchestrator auto-ran the council on code Claude itself just generated, auto-fix Blockers silently before delivery and note it (see Auto-correction section below).

---

## Fix preparation

Before editing any file:

1. List all CC-IDs being fixed
2. For overlapping fixes (CC-001 and CC-004 both touch line 34) — resolve the combined fix before writing. Do not apply independently.
3. Check for fix conflicts from the conflict check report — apply the resolved version

---

## Apply fixes

Use Claude Code's file editing tools to write fixes directly:

```
For each file being modified:
1. Read the current file content
2. Apply all fixes for that file in one edit (not multiple sequential edits)
3. Verify the fix addresses the CC-ID exactly
4. Move to the next file
```

After all fixes are applied:

```bash
# Run tooling again to confirm fixes don't introduce new errors
npx tsc --noEmit 2>&1 | head -20
npm test 2>&1 | tail -20
# (or language-appropriate equivalent)
```

---

## Auto-correction (orchestrator-triggered)

When the orchestrator auto-ran the council on code Claude generated in this session:

- Auto-fix all 🔴 Blockers silently
- Deliver the already-fixed code
- Append to delivery:

```
🔄 Auto-corrections applied before delivery
The Code Council found and fixed the following before handing this to you:
- CC-001 — [description of fix, one line]
- CC-002 — [description of fix, one line]

The code above already includes these fixes.
```

---

## Session tracking

After every fix, update the session record:

```
CC-001 [RESOLVED] — parameterized query applied, tsc clean
CC-002 [RESOLVED] — null check added
CC-003 [OPEN] — left for user (business logic decision)
CC-004 [OPEN] — pending /diagnose
```

This record persists for the rest of the session so delta reviews know what's been addressed.

---

## What delivery never modifies

- `node_modules/`, `vendor/`, `venv/` — never touch
- Lock files — `package-lock.json`, `yarn.lock`, `poetry.lock` — never touch  
- Generated files — Prisma client, proto output, ORM migrations — never touch
- `.env` files — never read or write (flag if secrets are hardcoded in source, but don't touch the env file itself)
- ADR files — if a fix has architectural implications, offer to create an ADR, don't modify existing ones

---

## After fixing — always run delta review

```
✅ Fixes applied. Running delta review on changed files...
[git diff --name-only output]
→ Re-running relevant reviewers on: [list of changed files]
```

Then produce the delta review format from `report-format.md`.

Close with updated verdict.
