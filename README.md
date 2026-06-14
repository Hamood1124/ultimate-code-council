# Ultimate Code Council

A self-driving code quality pipeline for Claude Code. One command starts a full pipeline — from aligning on what to build, through planning, TDD, multi-reviewer code review, security scanning, debugging, and shipping.

Everything is in this repo. No external dependencies. No third-party installs.

---

## The pipeline

```
/council-start
      ↓
  SETUP        → /setup-ashraf-skills       (once per repo, creates CONTEXT.md + SECURITY.md)
  ALIGN        → /grill-with-docs           (design tree interview)
  PLAN         → /to-prd + /to-issues       (PRD → vertical slice issues)
  BUILD        → /tdd                       (red-green-refactor)
  REVIEW       → /code-council              (6 reviewers, SHIP verdict)
               → /secrets-scan              (auto — before every SHIP IT)
               → /security-council          (auto — when auth/API/data files changed)
               → /msft-security             (auto — when Microsoft stack detected)
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

That's it. All skills are bundled in this repo. Nothing else to install.

---

## Skills in this repo

### Core pipeline

| Skill | Description |
|-------|-------------|
| `/council-start` | Master orchestrator — start here |
| `/code-council` | Six-reviewer code quality council |

### Security (Phase 2)

| Skill | Description |
|-------|-------------|
| `/secrets-scan` | Fast credential and secret scanner — runs before every ship |
| `/security-council` | Deep 5-agent security review (auth, secrets, data, API, deps) |
| `/msft-security` | Microsoft-stack security layer (SPFx, Graph API, D365, Azure) |

### Engineering skills (bundled)

| Skill | Used for |
|-------|----------|
| `/setup-ashraf-skills` | One-time repo setup (CONTEXT.md, SECURITY.md, issue tracker) |
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

### Security-specific commands
```
/secrets-scan              ← scan for hardcoded secrets, keys, credentials
/security-council          ← full deep security audit (auth, API, data, deps)
/msft-security             ← Microsoft-stack specific security review
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

## Issue ID system

Each skill uses its own issue ID series — no collisions:

| Prefix | Skill | Example |
|--------|-------|---------|
| `CC-` | Code Council | `CC-001` — null check missing |
| `SC-` | Security Council | `SC-001` — JWT issuer not validated |
| `SS-` | Secrets Scan | `SS-001` — API key hardcoded |
| `MS-` | MSFT Security | `MS-001` — Graph API over-permissioned |

---

## How it works

1. `/council-start` detects where you are in the pipeline — fresh start, has a PRD, has code ready to review, or needs debugging
2. It ensures `CONTEXT.md` (domain glossary) and `SECURITY.md` (security baseline) exist — creates both if not
3. It chains through the pipeline automatically, pausing only for real decisions
4. `/code-council` runs 6 specialist reviewers, executes your test suite and type checker, and produces a SHIP / SHIP WITH FIXES / DO NOT SHIP verdict
5. Security layer runs automatically based on what files changed — no manual triggering needed
6. Blockers are routed to `/diagnose`. Fixes are applied directly in the repo — no zips, no manual copying

---

## The six code reviewers

| Reviewer | Domain |
|----------|--------|
| Correctness Judge | Does it work? Logic, nulls, async, edge cases |
| Security Auditor | Injection, XSS, secrets, auth, OWASP Top 10 |
| Performance Engineer | N+1, memory leaks, blocking calls, scale |
| Maintainability Critic | Naming, structure, duplication, complexity |
| Requirements Verifier | Does it do what was asked? |
| Integration Skeptic | Side effects, contracts, environment assumptions |

## The five security agents

| Agent | Domain |
|-------|--------|
| Auth & Session | JWT, OAuth, session fixation, privilege escalation |
| Secrets (contextual) | Secrets in bundles, logs, URLs, Docker layers |
| Data Access | Injection, mass assignment, IDOR, over-fetching |
| API Surface | Unprotected endpoints, CORS, rate limiting, CSRF |
| Dependency | CVE scanning across npm, pip, NuGet, Go, Rust |

---

## Per-repo setup (run once per project)

When you open a new project in Claude Code for the first time:

```
/setup-ashraf-skills
```

This creates:
- `CONTEXT.md` — your project's domain glossary (all skills read this)
- `SECURITY.md` — your project's security baseline (all security skills read this)
- `docs/adr/` — architectural decision records
- `docs/agents/` — issue tracker config and triage labels

The orchestrator (`/council-start`) runs this automatically if it hasn't been done yet — you don't have to remember.

---

## License

MIT
