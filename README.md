# Ultimate Code Council

A self-driving code quality pipeline for Claude Code. One command starts a full pipeline — from aligning on what to build, through planning, TDD, multi-reviewer code review, debugging, and shipping.

Built on top of [Matt Pocock's engineering skills](https://github.com/mattpocock/skills) with a master orchestrator and six-reviewer Code Council added on top.

---

## The pipeline

```
/council-start
      ↓
  SETUP        → /setup-matt-pocock-skills  (once per repo)
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

### Option A — Install from this repo (recommended)

```bash
npx skills@latest add Hamood1124/ultimate-code-council
```

### Option B — Manual install

Clone this repo and copy the skills to your Claude Code global skills directory:

```bash
git clone https://github.com/Hamood1124/ultimate-code-council.git
cp -r ultimate-code-council/code-council ~/.claude/skills/
cp -r ultimate-code-council/council-start ~/.claude/skills/
```

### Install Matt Pocock's skills (required dependency)

```bash
npx skills@latest add mattpocock/skills
```

---

## Skills in this repo

| Skill | Description |
|-------|-------------|
| `/council-start` | Master orchestrator — start here |
| `/code-council` | Six-reviewer code quality council |

### Skills from Matt Pocock (installed separately)

| Skill | Used for |
|-------|----------|
| `/setup-matt-pocock-skills` | One-time repo setup (CONTEXT.md, issue tracker) |
| `/grill-with-docs` | Design alignment before building |
| `/to-prd` | Generate PRD from conversation |
| `/to-issues` | Break PRD into GitHub issues |
| `/tdd` | Test-driven development |
| `/diagnose` | Disciplined debugging for Blockers |
| `/improve-codebase-architecture` | Weekly architecture review |
| `/zoom-out` | Get broader context on unfamiliar code |
| `/prototype` | Throwaway prototype before committing |
| `/triage` | Issue backlog management |

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
or
```
ship check
```

### Quick scan before pushing
```
quick scan
```

### Deep audit
```
deep review
```

---

## How it works

1. `/council-start` detects where you are in the pipeline (fresh start / has PRD / has code / needs review)
2. It ensures `CONTEXT.md` exists (your project's domain glossary) — creates it if not
3. It chains through the pipeline automatically, pausing only for real decisions
4. `/code-council` runs 6 specialist reviewers, executes your test suite and type checker, and produces a SHIP / SHIP WITH FIXES / DO NOT SHIP verdict
5. Blockers are routed to `/diagnose`. Fixes are applied directly in the repo.

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

## License

MIT
