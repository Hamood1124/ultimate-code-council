# Ultimate Code Council

A self-driving code quality, security, and documentation pipeline for Claude Code. One command starts a full pipeline — from aligning on what to build, through planning, TDD, multi-reviewer code review, security scanning, debugging, documentation, and shipping.

Everything is in this repo. No external dependencies. No third-party installs.

---

## Installation

```bash
npx skills@latest add Hamood1124/ultimate-code-council -g --full-depth
```

One command. Gets everything. Global install — works in every project.

---

## Per-project setup (run once per project)

```
/setup-ashraf-skills
```

Creates `CONTEXT.md` (domain glossary) and `SECURITY.md` (security baseline). Takes 2 minutes. All skills read both.

---

## The pipeline

```
/council-start
      ↓
  SETUP        → /setup-ashraf-skills       (once per repo)
  ALIGN        → /grill-with-docs           (design interview)
  PLAN         → /to-prd + /to-issues       (PRD → tasks)
  ESTIMATE     → /estimate                  (effort & timeline)
  BUILD        → /tdd                       (test-first development)
  REVIEW       → /code-council              (6 reviewers)
               → /secrets-scan              (auto — every ship)
               → /security-council          (auto — auth/API/data changes)
               → /msft-security             (auto — Microsoft stack)
  FIX          → /diagnose                  (blocker debugging)
  SHIP         → ✓
  DOCUMENT     → (user picks from doc menu)
  MAINTAIN     → /improve-codebase-architecture
```

---

## All skills

### Core
| Skill | Description |
|-------|-------------|
| `/council-start` | Master orchestrator — start here |
| `/code-council` | 6-reviewer code quality council |

### Security
| Skill | Description |
|-------|-------------|
| `/secrets-scan` | Credential and secret scanner |
| `/security-council` | 5-agent deep security review |
| `/msft-security` | Microsoft stack security layer |

### Documentation
| Skill | Description |
|-------|-------------|
| `/handover-doc` | Project handover document |
| `/tech-doc` | Technical documentation |
| `/user-guide` | End user guide |
| `/changelog` | Auto-generate changelog |
| `/adr-writer` | Write architectural decision records |
| `/meeting-notes` | Format meeting minutes / MOM |
| `/sow` | Statement of Work / proposal |

### Dev Tools
| Skill | Description |
|-------|-------------|
| `/estimate` | Effort and timeline estimation |
| `/code-explain` | Explain unfamiliar code from zero |
| `/diff-review` | Review only changed files (PR review) |
| `/test-writer` | Write tests for existing untested code |
| `/refactor` | Safe proposal-first refactoring |
| `/env-setup` | Validate local environment setup |

### Client
| Skill | Description |
|-------|-------------|
| `/onboarding` | New developer onboarding guide |
| `/client-report` | Weekly client status update |

### Engineering (bundled)
| Skill | Description |
|-------|-------------|
| `/setup-ashraf-skills` | One-time repo setup |
| `/grill-with-docs` | Design alignment interview |
| `/to-prd` | Generate PRD |
| `/to-issues` | Break PRD into issues |
| `/tdd` | Test-driven development |
| `/diagnose` | Disciplined debugging |
| `/improve-codebase-architecture` | Architecture review |
| `/zoom-out` | Architectural context |
| `/prototype` | Throwaway prototype |
| `/triage` | Issue backlog management |
| `/handoff` | Pass context between sessions |

---

## Issue ID system

| Prefix | Skill | Example |
|--------|-------|---------|
| `CC-` | Code Council | `CC-001` |
| `SC-` | Security Council | `SC-001` |
| `SS-` | Secrets Scan | `SS-001` |
| `MS-` | MSFT Security | `MS-001` |

---

## Quick reference

| What you want | Command |
|---------------|---------|
| Start a new feature | `/council-start let's build X` |
| Review code | `ship check` or `review this` |
| Quick scan before pushing | `quick scan` |
| Full deep audit | `deep review` |
| Review only what changed | `/diff-review` |
| Scan for secrets | `/secrets-scan` |
| Full security audit | `/security-council` |
| Explain unfamiliar code | `/code-explain` |
| Write tests for existing code | `/test-writer` |
| Safe refactoring | `/refactor` |
| Check environment setup | `/env-setup` |
| Estimate effort | `/estimate` |
| Write a SOW | `/sow` |
| Write handover doc | `/handover-doc` |
| Weekly client update | `/client-report` |
| New dev onboarding | `/onboarding` |
| Debug a blocker | `/diagnose` |
| Weekly architecture check | `/improve-codebase-architecture` |

---

## Repo structure

```
ultimate-code-council/
├── council-start/          ← master orchestrator
├── code-council/           ← 6-reviewer quality council
├── security/
│   ├── security-council/
│   ├── msft-security/
│   └── secrets-scan/
├── docs/
│   ├── handover-doc/
│   ├── tech-doc/
│   ├── user-guide/
│   ├── changelog/
│   ├── adr-writer/
│   ├── meeting-notes/
│   └── sow/
├── dev-tools/
│   ├── estimate/
│   ├── code-explain/
│   ├── diff-review/
│   ├── test-writer/
│   ├── refactor/
│   └── env-setup/
├── client/
│   ├── onboarding/
│   └── client-report/
├── templates/
│   └── acme-sow-template.md
└── Third-Party-Tools/
```

---

## License

MIT
