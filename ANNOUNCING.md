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
SHIP IT ✅          — nothing blocking, you're good
SHIP WITH FIXES ⚠️  — Highs found, fix before merging
DO NOT SHIP 🔴      — Blockers found, needs diagnosis first
```

But honestly the review is just one part of it.

---

## Phase 2 — Security layer (just added)

On top of the code review, we now have a dedicated security layer that runs automatically:

### `/secrets-scan` — runs before every single ship
Scans every file for hardcoded API keys, tokens, passwords, connection strings, and private keys. Checks your `.gitignore` covers sensitive files. Checks git history for secrets that were committed and deleted (they're still there). Works on any language or stack.

```
SS-001 — API key hardcoded in source [🔴 Blocker] [Confirmed]
File: src/services/ApiService.ts line 12
Found: apiKey: "sk-ab****************************"
Fix: Move to environment variable. See SECURITY.md for approved approach.
```

### `/security-council` — deep 5-agent security review
Five specialist agents each cover a different attack surface:

| Agent | What they hunt for |
|-------|-------------------|
| Auth & Session | JWT flaws, OAuth misconfig, session fixation, weak passwords |
| Secrets (contextual) | Secrets in bundles, logs, URLs, error responses |
| Data Access | SQL/NoSQL injection, IDOR, mass assignment, path traversal |
| API Surface | Unprotected endpoints, broken auth, missing rate limits, CORS |
| Dependency | Known CVEs in your npm/pip/NuGet packages |

Issue IDs: `SC-001`, `SC-002`... Reads your project's `SECURITY.md` to avoid false positives on known-accepted patterns.

### `/msft-security` — Microsoft stack bonus layer
Specifically for SPFx, D365, Graph API, and Azure projects. Catches things generic tools miss:
- Graph API scope over-permission (`Files.ReadWrite.All` when `Files.Read` suffices)
- SPFx web part properties exposing secrets to the page
- D365 OData filter injection
- Azure Key Vault vs hardcoded secrets
- Azure AD app registration hygiene

Issue IDs: `MS-001`, `MS-002`...

### How security triggers automatically
You don't have to remember to run these. The orchestrator handles it:
- Before every SHIP IT → `/secrets-scan` always runs
- Changed files touch auth/tokens/sessions → `/security-council` (Auth agent) auto-runs
- Changed files touch API routes/controllers → `/security-council` (API agent) auto-runs
- Changed files touch DB/queries/models → `/security-council` (Data agent) auto-runs
- Microsoft stack detected + security files changed → `/msft-security` also runs

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
   Reviews it before handing it back (code + security)
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
Looks for `CONTEXT.md` (your project glossary) and `SECURITY.md` (security baseline). If neither exists, it runs `/setup-ashraf-skills` automatically — takes 2 minutes, happens once.

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
Writes one test, makes it pass, refactors, repeat. Uses your project's domain vocabulary from `CONTEXT.md`.

**Step 5 — Runs the full council**
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

Code Council runs. Then because files touch OData (data access), Security Council auto-triggers. Then because Microsoft stack is detected, MSFT Security also runs:

```
## 🔍 Security Auditor [Code Council]
CC-001 — OData query built with string interpolation [🔴 Blocker] [Confirmed]

## 🛡️ Data Access Agent [Security Council]
SC-001 — Employee ID not validated before OData filter [🔴 Blocker] [Confirmed]

## 🏢 D365 Check [MSFT Security]
MS-001 — OData $select not used — returning full Employee entity [🟠 High]

## 🔑 Secrets Scan
✅ No secrets found. .gitignore coverage: clean.
```

**Step 6 — Asks if you want it fixed**
```
Should I apply the fixes directly to your files?
I'll fix: CC-001, SC-001 (🔴 Blockers), MS-001 (🟠 High)
Reply yes to apply.
```

Fixes applied in place. Delta review confirms everything clean.

```
Updated verdict: SHIP IT ✅
```

---

## How to install it

**You need:** Claude Code installed. If you don't have it yet, ask Ashraf.

**One command — gets everything:**
```bash
npx skills@latest add Hamood1124/ultimate-code-council -g
```

Everything is bundled — code council, security layer, orchestrator, all supporting skills. Nothing else to install.

**Set up your project (once per repo):**

Open Claude Code in your project and run:
```
/setup-ashraf-skills
```

This creates `CONTEXT.md` (domain glossary) and `SECURITY.md` (security baseline). Takes 2 minutes. Every skill reads both from that point on.

**That's it.** Open any project and type `/council-start` to begin.

---

## Quick reference — commands you'll use

| What you want | What to type |
|---------------|-------------|
| Start a new feature | `/council-start let's build X` |
| Review code you wrote | `ship check` or `review this` |
| Quick scan before pushing | `quick scan` |
| Full deep audit | `deep review` |
| Scan for secrets/credentials | `/secrets-scan` |
| Full security audit | `/security-council` |
| Microsoft-stack security check | `/msft-security` |
| Debug a specific bug | `/diagnose` |
| Weekly architecture review | `/improve-codebase-architecture` |
| Get broader context on unfamiliar code | `/zoom-out` |

---

## Questions?

Ping Ashraf on Teams or drop a message in the dev channel. Happy to pair on the first session so you can see the full pipeline in action.

GitHub repo: https://github.com/Hamood1124/ultimate-code-council
