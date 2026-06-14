# 👋 Hey ACME team — read this first

This is the Ultimate Code Council. I built it to solve a problem we all hit constantly: you write code, you're not sure if it's solid, you ask Claude, it says "looks good!" and then something breaks in prod.

This fixes that. Here's everything you need to know.

---

## What it actually does

Instead of asking Claude to review your code and hoping for the best, this runs your code through **6 independent specialist reviewers** — each one focused on a completely different thing:

| Reviewer | What they check |
|----------|----------------|
| Correctness Judge | Will this actually work? Nulls, logic errors, broken async, edge cases |
| Security Auditor | SQL injection, exposed secrets, missing auth checks, XSS |
| Performance Engineer | N+1 queries, memory leaks, blocking calls, things that die at scale |
| Maintainability Critic | Can someone else read this in 6 months? |
| Requirements Verifier | Does it do what you actually asked for? |
| Integration Skeptic | Will this break when it talks to the rest of the system? |

Each reviewer gives you severity-ranked issues (`🔴 Blocker / 🟠 High / 🟡 Medium / 🟢 Low`) with unique IDs (`CC-001`, `CC-002`...) and a final verdict:

```
SHIP IT ✅       — nothing blocking, you're good
SHIP WITH FIXES ⚠️  — Highs found, fix before merging
DO NOT SHIP 🔴   — Blockers found, needs diagnosis first
```

But honestly the review is just one part of it.

---

## The full pipeline

The real power is that it's a **self-driving pipeline**. You type one thing and it handles the rest:

```
You: "let's build a leave request approval workflow"
        ↓
   Interviews you until the plan is clear
        ↓
   Writes a PRD and breaks it into tasks
        ↓
   Builds it using TDD (test first, then code)
        ↓
   Reviews it before handing it back
        ↓
   Fixes any blockers it finds
        ↓
   "SHIP IT ✅"
```

You only need to:
- Answer questions during the planning phase
- Pick which task to work on
- Decide depth: Quick / Standard / Deep review

Everything else is automatic.

---

## Real example — SPFx leave request component

Here's what an actual session looks like. You open Claude Code in your SPFx project and type:

```
/council-start let's build the leave request submission form
```

**What happens next:**

**Step 1 — It checks your project first (silently)**
Looks for `CONTEXT.md` (your project glossary), reads existing ADRs, detects it's an SPFx TypeScript project. If `CONTEXT.md` doesn't exist yet, it creates one by asking you a few questions about your project's domain terms.

**Step 2 — It interviews you**
```
Claude: What types of leave does this form need to support?
You: Annual, sick, emergency, and unpaid

Claude: Should the requester see their remaining balance per type before submitting?
You: Yes, pull it from D365 F&O via OData

Claude: Who are the approvers — direct manager only, or multi-level?
You: Direct manager first, then department head if more than 5 days
```
...keeps going until every branch of the design is resolved.

**Step 3 — Writes the PRD and tasks**
Creates a proper requirements doc with user stories and breaks it into independently workable tasks. Posts them to your issue tracker.

**Step 4 — Builds it with TDD**
Writes one test, makes it pass, refactors, repeat. Uses your project's domain vocabulary from `CONTEXT.md` — it calls things "LeaveRequest" not "formData" because it read your glossary.

**Step 5 — Runs the council before handing it back**
```
📋 Council Context
──────────────────
Language(s):    TypeScript
Framework(s):   SPFx 1.18, React, PnP.js v3
Depth mode:     Standard
Deep scan:      12 files
Quick scan:     3 files (lock files, dist)
Skipped:        node_modules/

Tooling run:
  ✅ tsc --noEmit:  clean
  ✅ eslint:        2 warnings
  ✅ jest:          14 passed
```

Then the reviewers run. Example findings:

```
## 🔍 Security Auditor
Issues found: 1  |  Highest severity: 🔴 Blocker

### CC-001 — OData query built with string interpolation [🔴 Blocker] [Confirmed]
Location: src/services/LeaveService.ts line 34
What's wrong: The employee ID is concatenated directly into the OData
  filter string. If the ID comes from user input, this is an injection risk.
Why it matters: Can be exploited to access other employees' leave records.
Suggested fix: Use encodeURIComponent() or build the filter with a
  parameterized helper instead of string concatenation.

## ✅ Performance Engineer
No issues found at Standard depth.

## ✅ Correctness Judge  
No issues found at Standard depth.
```

**Step 6 — Asks if you want it fixed**
```
Should I apply the fixes directly to your files?
I'll fix: CC-001 (🔴 Blocker)
Reply yes to apply.
```

You say yes. It fixes the file in place, runs `tsc --noEmit` again to confirm it's clean, runs a delta review on just that file, and comes back:

```
🔁 Delta Review
CC-001 — ✅ Resolved. No new issues introduced.

Updated verdict: SHIP IT ✅
```

Done. The whole thing took less time than a manual PR review.

---

## How to install it

**You need:** Claude Code installed. If you don't have it yet, ask Naveen.

**Step 1 — Install Matt Pocock's engineering skills (dependency)**
```bash
npx skills@latest add mattpocock/skills -g
```

**Step 2 — Install the Code Council**
```bash
npx skills@latest add Hamood1124/ultimate-code-council -g
```

The `-g` flag installs globally so it works in every project, not just one.

**Step 3 — Set up your project (once per repo)**

Open Claude Code in your project and run:
```
/setup-matt-pocock-skills
```

This creates a `CONTEXT.md` file — your project's domain glossary. Takes 2 minutes. Every skill reads this so Claude always uses your project's actual terminology.

**That's it.** Open any project and type `/council-start` to begin.

---

## Quick reference — commands you'll use

| What you want | What to type |
|---------------|-------------|
| Start a new feature | `/council-start let's build X` |
| Review code you wrote | `ship check` or `review this` |
| Quick scan before pushing | `quick scan` |
| Full deep audit | `deep review` |
| Debug a specific bug | `/diagnose` |
| Weekly architecture review | `/improve-codebase-architecture` |
| Get broader context on unfamiliar code | `/zoom-out` |

---

## Questions?

Ping Ashraf on Teams or drop a message in the dev channel. Happy to pair on the first session so you can see the full pipeline in action.

GitHub repo: https://github.com/Hamood1124/ultimate-code-council
