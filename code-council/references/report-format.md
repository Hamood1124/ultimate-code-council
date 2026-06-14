# Code Council — Report Format

## Issue ID system

Every finding gets a unique ID: `CC-001`, `CC-002`, etc. IDs increment across all reviewers globally — not per-reviewer. Start at `CC-001` on first run. On re-review/delta, continue from the last issued ID.

Track status:
- `CC-001 [OPEN]` — not yet fixed
- `CC-001 [RESOLVED]` — fixed and confirmed in delta review
- `CC-001 [REGRESSION]` — was resolved, has re-appeared

---

## Reviewer block format

```markdown
## [emoji] [Reviewer Name]
**Issues found:** N  |  **Highest severity:** 🔴 Blocker

### CC-001 — [Short descriptive title] [🔴 Blocker] [Confirmed]
**Location:** `src/auth/login.ts` line 34
**Domain reference:** Relates to the `Customer` authentication flow (CONTEXT.md)
**What's wrong:** [Clear explanation using CONTEXT.md vocabulary where possible]
**Why it matters:** [Impact if not fixed — be specific, not generic]
**Tooling signal:** `tsc` reports: `error TS2345: Argument of type...` ← include if tooling caught it
**Suggested fix:** [Concrete direction — what needs to change, not the full corrected code]

### CC-002 — ...
```

If a reviewer finds nothing at the current depth threshold:
```markdown
## ✅ [Reviewer Name]
No issues found at [Quick/Standard/Deep] depth.
```

Do not pad with generic observations. If there's nothing to flag, say so and move on.

---

## Severity definitions

| Level | Emoji | Definition |
|-------|-------|------------|
| Blocker | 🔴 | Will crash, break, or get you exploited. Do not ship. Tooling errors are always Blockers. |
| High | 🟠 | Won't crash now but causes real problems at scale, edge cases, or security-adjacent |
| Medium | 🟡 | Degraded quality, maintainability debt, suboptimal but functional |
| Low | 🟢 | Style, naming, nitpicks. Noted but non-blocking |

---

## Confidence definitions

| Level | Definition |
|-------|------------|
| `Confirmed` | Bug/issue is directly visible in the code shown. Tooling output confirming it = always Confirmed. |
| `Suspected` | Pattern strongly suggests a problem but runtime or caller context would confirm |
| `Advisory` | Worth knowing, depends on use case, hardening recommendation |

---

## Conflict check format

```markdown
## ⚡ Conflict Check

### Conflict 1 — CC-003 fix introduces CC-007 concern
**Reviewer A (Security) says:** Replace string concatenation with parameterized query
**Reviewer B (Performance) flags:** The parameterized query pattern proposed uses 
  a new DB connection per call — this will create connection pool exhaustion at scale
**Resolution:** Use parameterized query AND pass the existing connection object. 
  Security takes priority; Performance concern is resolved by reusing the connection.
```

If no conflicts: `✅ No conflicts between proposed fixes.`

---

## Full verdict format

```markdown
---

## 🏛️ Council Verdict

**Verdict:** [SHIP IT / SHIP WITH FIXES / DO NOT SHIP]

| Severity | Count | CC-IDs |
|----------|-------|--------|
| 🔴 Blocker | N | CC-001, CC-004 |
| 🟠 High | N | CC-002, CC-003 |
| 🟡 Medium | N | CC-005 |
| 🟢 Low | N | CC-006 |

**Verdict logic:**
- 🔴 Any Blocker present → **DO NOT SHIP**
- 🟠 Any High present (no Blockers) → **SHIP WITH FIXES**  
- Only 🟡 Medium / 🟢 Low → **SHIP IT** (with notes)

---

### Where the council agrees
[Points 2+ reviewers flagged independently — highest-confidence signals. 
Use CONTEXT.md vocabulary.]

### The one thing to fix first
CC-XXX — [description]. Fix this before anything else.

### What this review did not cover
- Runtime behavior under real traffic
- Third-party API response contracts (assumed based on docs)
- Database schema (no migration files in scope)
- Environment variables (`.env` detected but not read)
- [anything else genuinely not covered]

---
```

---

## Verdict → next step mapping

| Verdict | Orchestrator action |
|---------|-------------------|
| SHIP IT | Pipeline complete — offer `/improve-codebase-architecture` |
| SHIP WITH FIXES | Offer to fix Highs in-repo → delta review → loop |
| DO NOT SHIP | Auto-route each Blocker to `/diagnose` → fix → delta review → loop |

---

## Delta review format

Used when re-reviewing after fixes are applied:

```markdown
## 🔁 Delta Review — Checking CC-001, CC-003, CC-005

### CC-001 — ✅ Resolved
Parameterized query confirmed in place. No new issues introduced.

### CC-003 — ✅ Resolved  
Null check added. Note: introduced CC-008 (🟢 Low) — variable name `safeUser` 
is inconsistent with CONTEXT.md term `authenticatedCustomer`. Non-blocking.

### CC-005 — ⚠️ Partially resolved
The immediate XSS is fixed but the sanitizer is not applied to the 
`description` field on line 89. Updated severity: 🟠 High. Still open.

### 🆕 CC-008 — New issue [🟢 Low] [Advisory]
[description]

**Updated verdict: SHIP WITH FIXES** — CC-005 still open, CC-008 is non-blocking.
```
