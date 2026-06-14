# Ultimate Code Council

A self-driving code quality pipeline for Claude Code. One command starts a full pipeline — from aligning on what to build, through planning, TDD, multi-reviewer code review, debugging, and shipping.

Everything is in this repo. No external dependencies. No third-party installs.

---

## The pipeline

```
/council-start
      ↓
  SETUP        → /setup-ashraf-skills       (once per repo)
  ALIGN        → /grill-with-docs           (design tree interview)
  PLAN         → /to-prd + /to-issues       (PRD → vertical slice issues)
  BUILD        → /tdd                       (red-green-refactor)
  REVIEW       → /code-council              (6 reviewers, SHIP verdict)
  FIX          → /diagnose                  (disciplined debugging)
  SHIP         → ✓
  MAINTAIN     → /improve-codebase-architecture (weekly)
```

The user only intervenes for real decisions. Everything else chains automatically.

---

## Installation

### One command — gets everything

```bash
npx skills@latest add Hamood1124/ultimate-code-council -g
```

The `-g` flag installs globally so it works in every project, not just one.

That's it. All skills — the Code Council, the orchestrator, and all supporting engineering skills — are bundled in this repo. Nothing else to install.

---

## Skills in this repo

### Your skills (built in-house)

| Skill | Description |
|-------|-------------|
| `/council-start` | Master orchestrator — start here |
| `/code-council` | Six-reviewer code quality council |

### Engineering skills (bundled — no separate install needed)

| Skill | Used for |
|-------|----------|
| `/setup-ashraf-skills` | One-time repo setup (CONTEXT.md, issue tracker) |
| `/grill-with-docs` | Design alignment before building |
| `/to-prd` | Generate PRD from conversation |
| `/to-issues` | Break PRD into GitHub issues |
| `/tdd` | Test-driven development |
| `/diagnose` | Disciplined debugging for Blockers |
| `/improve-codebase-architecture` | Weekly architecture review |
| `/zoom-out` | Get broader context on unfamiliar code |
| `/prototype` | Throwaway prototype before committing |
| `/triage` | Issue backlog management |
| `/grill-me` | Relentless design interview |
| `/handoff` | Pass context between agent sessions |

---

## Usage

### Start a new feature
```
/council-start let's build a user authentication system
```

### Review existing code
```
/code-council
```
or just say:
```
ship check
review this
is this safe to ship
```

### Quick scan before pushing
```
quick scan
```

### Deep audit
```
deep review
```

### Debug a specific bug
```
/diagnose
```

### Weekly architecture review
```
/improve-codebase-architecture
```

---

## How it works

1. `/council-start` detects where you are in the pipeline — fresh start, has a PRD, has code ready to review, or needs debugging
2. It ensures `CONTEXT.md` exists (your project's domain glossary) — creates it if not, using `/setup-ashraf-skills`
3. It chains through the pipeline automatically, pausing only for real decisions
4. `/code-council` runs 6 specialist reviewers, executes your test suite and type checker, and produces a SHIP / SHIP WITH FIXES / DO NOT SHIP verdict
5. Blockers are routed to `/diagnose`. Fixes are applied directly in the repo — no zips, no manual copying.

---

## The six reviewers

| Reviewer | Domain |
|----------|--------|
| Correctness Judge | Does it work? Logic, nulls, async, edge cases |
| Security Auditor | Injection, XSS, secrets, auth, OWASP Top 10 |
| Performance Engineer | N+1, memory leaks, blocking calls, scale |
| Maintainability Critic | Naming, structure, duplication, complexity |
| Requirements Verifier | Does it do what was asked? |
| Integration Skeptic | Side effects, contracts, environment assumptions |

---

## Per-repo setup (run once per project)

When you open a new project in Claude Code for the first time:

```
/setup-ashraf-skills
```

This creates:
- `CONTEXT.md` — your project's domain glossary (all skills read this)
- `docs/adr/` — architectural decision records
- `docs/agents/` — issue tracker config and triage labels

The orchestrator (`/council-start`) runs this automatically if it hasn't been done yet — you don't have to remember.

---

## License

MIT
