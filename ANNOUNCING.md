# 👋 Hey ACME team — the Ultimate Code Council

I've been building something for the past few weeks and it's ready to share. This is the most complete version yet — three phases of development covering code quality, security, documentation, and the full project lifecycle from client inquiry to handover.

Read this top to bottom once. After that the quick reference at the bottom is all you'll need day-to-day.

---

## What problem does this solve?

We've all been there:

- You ask Claude to review code and it says "looks good!" — then something breaks in prod
- You finish a project and spend two days writing documentation from scratch
- A new dev joins and spends a week just figuring out how to run the project locally
- You write a SOW, under-estimate the effort, and end up eating the difference
- A secret key gets committed to GitHub accidentally
- You ship code that works but has an SQL injection no one noticed

The Ultimate Code Council is a skill system for Claude Code that addresses all of these. It's not one tool — it's a full pipeline that covers the entire development lifecycle, with every skill talking to every other skill.

---

## How it's organized — three phases

### Phase 1 — Code Quality (the foundation)
The core six-reviewer code council that runs on any code, any language.

### Phase 2 — Security
Three dedicated security skills that scan for secrets, audit your auth and API surface, and go deep on Microsoft-stack specific vulnerabilities.

### Phase 3 — Documentation & Dev Tools
Seven documentation skills, six developer utility tools, and two client-facing report generators.

---

## The master orchestrator — `/council-start`

Everything starts here. One command, and the pipeline figures out where you are and what to do next.

```
/council-start let's build a leave request approval workflow
```

It detects your position in the lifecycle automatically:

| What it finds | What it does |
|---------------|-------------|
| No `CONTEXT.md` | Runs setup first — creates domain glossary and security baseline |
| Fresh start | Interviews you about what you want to build |
| PRD exists, no issues | Breaks the PRD into tasks |
| Tasks exist, code needed | Builds using TDD |
| Code written | Runs the full review pipeline |
| Blockers found | Routes to `/diagnose` |
| Clean verdict | Shows doc menu, waits for your choice |

You only intervene for real decisions. Everything else is automatic.

---

## Phase 1 — Code Quality

### `/code-council` — the six reviewers

Six specialist reviewers analyze your code independently. Each one only cares about their domain — they don't try to be balanced.

| Reviewer | What they look for |
|----------|-------------------|
| **Correctness Judge** | Logic errors, null references, broken async, off-by-one, edge cases that crash at runtime |
| **Security Auditor** | SQL injection, XSS, exposed secrets, missing auth checks, OWASP Top 10 |
| **Performance Engineer** | N+1 queries, memory leaks, blocking calls, things that die at scale |
| **Maintainability Critic** | Naming, duplication, complexity, "will someone understand this in 6 months" |
| **Requirements Verifier** | Does it actually do what was asked? Checks against the original PRD/issue |
| **Integration Skeptic** | Side effects, API contract assumptions, environment dependencies, things that work alone but break in the system |

Every finding gets a unique ID (`CC-001`, `CC-002`...), a severity level, and a confidence rating:

```
🔴 Blocker    — will crash, break, or get you hacked. DO NOT SHIP.
🟠 High       — real problem, fix before merging
🟡 Medium     — quality debt, fix when you can
🟢 Low        — nitpick, your call
```

**Three depth modes — you choose:**

```
⚡ Quick    — Blockers and Highs only. "Is this safe to push right now?"
🔍 Standard — Full pass, flags everything High and above. Default.
🧠 Deep     — Everything including Lows. Pre-launch, legacy audits, client deliverables.
```

**The verdict:**

```
SHIP IT ✅          — nothing blocking
SHIP WITH FIXES ⚠️  — Highs found, fix before merging
DO NOT SHIP 🔴      — Blockers present
```

**Fixes go directly into your files** — no zip, no manual copying. Claude edits the file in place, runs your test suite to confirm it's clean, then does a delta review on only the changed files.

**The tooling runs too** — before reviewers even start, it runs whatever your project has:

```bash
tsc --noEmit      # TypeScript errors
eslint            # Lint errors
jest / pytest     # Tests
npm audit         # Dependency CVEs
```

Tooling errors are always `Confirmed` findings — ground truth, not advisory.

---

### `/diff-review` — PR review, token-efficient

Same six reviewers, but scoped only to files that changed. The most common tool you'll use day-to-day.

```
/diff-review
```

Reads `git diff` vs main, shows you what changed, runs the council on only those files. If a change in File A breaks something in unchanged File B, it still catches it.

Verdict at the end tells you if it's safe to merge.

---

## Phase 2 — Security

Three dedicated security skills that run on top of the code council. Language-agnostic — work on any stack.

### `/secrets-scan` — runs before every ship

Scans every file for hardcoded credentials using pattern matching:

- API keys (OpenAI `sk-`, GitHub `ghp_`, Slack `xox*`, AWS `AKIA*`, Google `AIza*`)
- Passwords and connection strings
- Private keys and certificates
- Azure/Microsoft credentials (client secrets, storage account keys, SAS tokens)
- Secrets in comments (the most dangerous kind — deleted code still in history)

Also checks:
- `.gitignore` covers `.env`, `*.pem`, `*.key`, certificate files
- Git history for secrets that were committed and deleted (they're still recoverable)

Reads `SECURITY.md` to avoid flagging known-accepted patterns as false positives.

Issue IDs: `SS-001`, `SS-002`...

Can wire itself as a git pre-commit hook — runs automatically before every `git commit`.

---

### `/security-council` — five specialist agents

Goes deeper than the Security Auditor in the main council. Five agents, each covering a different attack surface:

| Agent | What they hunt |
|-------|---------------|
| **Auth & Session** | JWT flaws (algorithm confusion, missing expiry, weak secrets), OAuth misconfig (missing state param, implicit flow), session fixation, privilege escalation, weak password hashing |
| **Secrets (contextual)** | Secrets in client-side bundles, secrets passed in URLs, secrets in logs and error responses, secrets in Docker ENV layers |
| **Data Access** | SQL/NoSQL injection, ORM mass assignment, IDOR (accessing other users' records), path traversal, missing input validation |
| **API Surface** | Unauthenticated endpoints, broken authorization (authn ≠ authz), missing rate limiting, permissive CORS, CSRF gaps, GraphQL introspection in prod |
| **Dependency** | CVE scanning across npm, pip, NuGet, Go, Rust, Ruby — maps CVSS scores to council severity |

Each finding tagged with OWASP Top 10 category. Severity adjusted based on data classification from `SECURITY.md` — a Blocker in a public-internet app may be a High in an internal tool.

Issue IDs: `SC-001`, `SC-002`...

After a clean review, offers to update `SECURITY.md` with the review date, scope, and any accepted risks.

---

### `/msft-security` — Microsoft stack bonus layer

Only activates when Microsoft stack is detected (SPFx, D365, Graph API, Azure, Power Platform). Catches things generic tools miss entirely:

**SharePoint / SPFx:**
- Sensitive values in web part properties (visible to SharePoint admins, can appear in page source)
- Over-permissioned package-solution.json scopes (`Directory.ReadWrite.All`, `Sites.FullControl.All`)
- SharePoint search queries that may return content the user shouldn't access

**Graph API:**
- Scope over-permission — maps every API call to minimum required scope
- Access tokens in `localStorage` (XSS-accessible) or passed in URLs (appear in logs)
- `/beta/` endpoint usage in production code

**Azure AD:**
- Client secrets hardcoded anywhere (always a Blocker in SPFx — secrets in browser bundles are public)
- Wildcard redirect URIs
- Missing tenant ID validation in multi-tenant apps

**D365 F&O / Business Central:**
- OData filter injection via string concatenation
- Missing `$select` returning full entities when partial data needed
- Cross-company data access without explicit company filter
- Direct SQL via `Statement`/`ResultSet` instead of X++ select

**Azure:**
- Storage account keys in code or config
- Azure Functions with `authLevel: "anonymous"` on protected functions
- Managed identity available but client secret used instead

Issue IDs: `MS-001`, `MS-002`...

---

### How security triggers automatically

You don't have to remember to run these:

| What changed | What triggers |
|-------------|--------------|
| Any ship → always | `/secrets-scan` |
| Auth/token/session files | `/security-council` (Auth agent) |
| API routes/controllers | `/security-council` (API agent) |
| DB/queries/models | `/security-council` (Data agent) |
| Above + Microsoft stack | `/msft-security` also runs |

---

## Phase 3 — Documentation

Seven documentation skills. **None generate automatically** — after every SHIP IT, a menu appears and you pick exactly what you need. Nothing runs without an explicit yes.

### `/handover-doc` — project handover

Full handover document for when a project moves to a client or a new team member. Reads `CONTEXT.md`, `SECURITY.md`, ADRs, PRD, and issue history before asking a single question. Covers: what was built, architecture decisions, how to deploy, known issues, contacts, where credentials live, maintenance guide.

Delivered as `HANDOVER.md` or `.docx`.

---

### `/tech-doc` — technical documentation

API reference, component docs, data flow diagrams, environment setup guide. Reads actual code — not just comments. Scoped to exactly what you need: ask for API docs only and it generates only that.

---

### `/user-guide` — end user guide

Plain-English documentation for the actual users. No code, no technical terms without explanation. Uses "you" throughout. Every action is a numbered step. Built from the PRD so it covers what was actually asked for.

---

### `/changelog` — auto-generate changelog

Reads resolved CC/SC/SS/MS IDs, git commit messages, and PRD features to generate a clean `CHANGELOG.md` entry in Keep a Changelog format. Suggests version bump based on what changed (patch / minor / major). Writes directly to `CHANGELOG.md`.

---

### `/adr-writer` — architectural decision records

Writes a proper ADR when a significant technical decision is made. Covers context, decision, alternatives considered, consequences, and reversibility. Saves to `docs/adr/ADR-NNN-[slug].md`. Referenced by Security Council and Integration Skeptic in future reviews.

---

### `/meeting-notes` — MOM / meeting minutes

Paste rough notes, bullets, or a voice memo transcript — it formats them into a clean Minutes of Meeting document. Action items get owners and due dates. Decisions are clearly separated from discussion. Delivered as markdown or `.docx`.

---

### `/sow` — Statement of Work

Pre-project skill — runs before `/council-start`, not after. Structured interview covering client, project scope, in-scope, out-of-scope, timeline, assumptions, dependencies, and commercial terms. Reads ACME templates from `templates/` for standard language and T&Cs.

Delivered as `.docx` for client-facing use.

> Review Section 3.2 (Out of Scope) and Section 7 (Assumptions) before sending — these protect ACME if scope creep happens.

---

## Phase 3 — Dev Tools

Six developer utility tools that make day-to-day work faster.

### `/estimate` — effort estimation

Estimates development effort per task from a PRD or issue list. Calibrates to your team (senior/mid/junior), accounts for integration complexity, adds standard buffers (code review, UAT, unknowns). Flags risks that could inflate the estimate. Useful before writing a SOW to make sure your pricing is right.

---

### `/code-explain` — explain unfamiliar code

Opens an unfamiliar codebase and explains it from zero. What does it do, how is it structured, what are the key files, how does data flow through it. Adjusts depth based on whether you're a developer new to the codebase or a non-technical person who needs the general idea.

---

### `/test-writer` — write tests for existing code

Writes tests for code that has none. Different from `/tdd` (which writes tests before code) — this works backward from existing code. Runs the Correctness Judge first to understand what the code actually does before writing any tests. Reports coverage before and after.

---

### `/refactor` — safe proposal-first refactoring

Reads code, proposes specific refactoring moves (extract function, simplify conditional, remove duplication, rename to match domain vocabulary), asks which ones you want, applies them one at a time, runs tests after each move, and reverts automatically if tests break. Runs the council after to confirm quality improved.

---

### `/env-setup` — environment validator

Checks your local environment against what the project needs. Reports exact version mismatches, missing `.env` variables (with where to get the values from `SECURITY.md`), missing dependencies, Docker not running. Gives you the exact commands to fix each issue. Re-runs after you apply fixes to confirm everything is clean.

---

### `/diff-review` — PR review

Covered in Phase 1 above. Listed here because it's also a standalone dev tool — use it any time before opening a PR, not just at the end of a full build session.

---

## Phase 3 — Client Skills

### `/onboarding` — new developer onboarding guide

Generates a getting-started guide for a new developer joining the project. Covers: what the project does, access they need to request, step-by-step local setup, environment variables (names + where to get values), folder structure and key files, what NOT to touch, known gotchas, key contacts, and a cheat sheet of common commands.

Saved as `ONBOARDING.md` in the project root.

---

### `/client-report` — weekly client status update

Converts technical work (resolved CC-IDs, closed issues, features shipped) into non-technical language a business stakeholder can read. No CC-IDs, no code, no jargon. Covers completed work, in-progress, next week's plan, risks, and any decisions needed from the client. 

Delivered as `.docx` or email-ready markdown.

---

## Real example — SPFx leave request, full pipeline

This is what an actual session looks like from start to finish.

```
/council-start let's build the leave request submission form
```

**Setup (silent, automatic):**
Detects SPFx TypeScript project. Reads `CONTEXT.md` (domain glossary — knows to call it "LeaveRequest" not "formData"). Reads `SECURITY.md` (knows auth is Azure AD, data classification is client-facing). Reads ADRs.

**Alignment:**
```
Claude: What types of leave does this form support?
You: Annual, sick, emergency, unpaid

Claude: Should the requester see their balance before submitting?
You: Yes, pull from D365 F&O via OData

Claude: Who approves — direct manager only or multi-level?
You: Manager first, then department head if more than 5 days
```

**Plan:**
PRD created, broken into 4 vertical-slice issues, posted to issue tracker.

**Build:**
TDD — one test, one implementation, one refactor, repeat. Domain terms from `CONTEXT.md` used throughout.

**Review (all auto-triggered):**
```
📋 Council Context
Language: TypeScript | Framework: SPFx 1.18, React, PnP.js v3
Deep scan: 12 files | Quick scan: 3 | Skipped: node_modules
tsc: clean | eslint: 2 warnings | jest: 14 passed

🔴 CC-001 [Correctness] — Null reference on LeaveRequest.balance when type is Unpaid
🔴 SC-001 [Security/Data] — OData filter built with string interpolation — injection risk
🟠 MS-001 [MSFT] — $select not used — returning full Employee entity
🔑 SS: No secrets found. .gitignore: clean.

Verdict: DO NOT SHIP
```

**Fix:**
```
Should I fix CC-001, SC-001 (Blockers) and MS-001 (High)?
You: yes
```

Files fixed in place. Tests re-run. Delta review confirms clean.

```
Updated verdict: SHIP IT ✅
```

**Post-ship menu:**
```
What do you need?
📄 /handover-doc  📘 /tech-doc  👤 /user-guide
📝 /changelog  🏛️ /adr-writer  🗒️ /meeting-notes  📊 /client-report

You: changelog and client-report
```

Two documents generated. Done.

---

## How to install

There are two ways to use the Ultimate Code Council depending on how you access Claude.

---

### Option A — Claude Code (VS Code / Terminal) — Full power

This gives you the complete 32-skill pipeline with file editing, git integration, tooling runner, and everything else.

**Step 1 — Install Claude Code**

If you don't have it yet, ask Ashraf. You need it installed either as a VS Code extension or as a terminal tool.

**Step 2 — Install all skills (one command)**

```bash
npx skills@latest add Hamood1124/ultimate-code-council -g --full-depth
```

The `-g` flag installs globally so it works in every project. The `--full-depth` flag is required — without it the installer only finds 2 skills instead of all 32.

**Step 3 — Set up your project (once per project)**

Open Claude Code inside your project folder and run:
```
/setup-ashraf-skills
```

This creates `CONTEXT.md` (domain glossary) and `SECURITY.md` (security baseline). Takes 2 minutes. Only needs to happen once per project.

**Step 4 — Start using it**
```
/council-start let's build X
```

**Update when new skills are released:**
```bash
npx skills@latest add Hamood1124/ultimate-code-council -g --full-depth
```

---

### Option B — Claude.ai Web (claude.ai) — Lightweight version

If you use Claude on the web browser without Claude Code installed, you can still use many of the skills manually. You won't have file system access or git integration, but the document and analysis skills work perfectly.

**Step 1 — Go to the GitHub repo**
```
https://github.com/Hamood1124/ultimate-code-council
```

**Step 2 — Find the skill you want**

Navigate to the skill folder (e.g. `Documentation/docs/meeting-notes/`) and click `SKILL.md`.

**Step 3 — Download the raw file**

Click the **Raw** button at the top right of the file, then save it to your computer (`Ctrl+S` / `Cmd+S`).

**Step 4 — Upload to Claude.ai**

Go to [claude.ai](https://claude.ai) → **Settings** → **Skills** → Upload the `.md` file you downloaded.

**Step 5 — Use it**

Start a new chat and the skill will be available. Trigger it by name, e.g. `/meeting-notes` or just describe what you want.

---

### Which skills work where?

Not all skills can run on the web — some need file system access, git, or a terminal. Here's the full breakdown:

| Skill | Claude Code ✅ | Web (claude.ai) |
|-------|--------------|----------------|
| `/council-start` | ✅ Full pipeline | ❌ Needs filesystem + git |
| `/code-council` | ✅ Full — runs tooling, fixes files | ⚠️ Paste code manually, no auto-fixes |
| `/diff-review` | ✅ Full — reads git diff | ❌ Needs git |
| `/secrets-scan` | ✅ Full — scans all files + git history | ⚠️ Paste code manually, no history scan |
| `/security-council` | ✅ Full — scans filesystem | ⚠️ Paste code manually |
| `/msft-security` | ✅ Full — scans filesystem | ⚠️ Paste code manually |
| `/tdd` | ✅ Full — writes + runs tests | ❌ Needs filesystem |
| `/refactor` | ✅ Full — edits files, runs tests | ❌ Needs filesystem + tests |
| `/env-setup` | ✅ Full — checks your environment | ❌ Needs terminal |
| `/test-writer` | ✅ Full — writes test files | ❌ Needs filesystem |
| `/diagnose` | ✅ Full — runs feedback loop | ❌ Needs terminal |
| `/estimate` | ✅ Full | ✅ Works great on web |
| `/code-explain` | ✅ Full | ✅ Works great — paste code |
| `/meeting-notes` | ✅ Full | ✅ Works great on web |
| `/sow` | ✅ Full | ✅ Works great on web |
| `/handover-doc` | ✅ Full — reads project files | ✅ Works — paste context manually |
| `/tech-doc` | ✅ Full — reads actual code | ✅ Works — paste code manually |
| `/user-guide` | ✅ Full | ✅ Works great on web |
| `/changelog` | ✅ Full — reads git + issue IDs | ⚠️ Works — paste changes manually |
| `/adr-writer` | ✅ Full | ✅ Works great on web |
| `/client-report` | ✅ Full — reads git history | ✅ Works — describe work manually |
| `/onboarding` | ✅ Full — reads project | ✅ Works — paste context manually |

**Web recommendation:** For web users, the most useful skills to download are `/meeting-notes`, `/sow`, `/estimate`, `/user-guide`, `/adr-writer`, `/client-report`, and `/code-explain`. These work fully without needing any local setup.

---

## The complete command reference

| What you want | Command |
|---------------|---------|
| Start new feature | `/council-start let's build X` |
| Review code | `ship check` or `review this` |
| Quick scan | `quick scan` |
| Deep audit | `deep review` |
| PR review (changed files only) | `/diff-review` |
| Scan for secrets | `/secrets-scan` |
| Full security audit | `/security-council` |
| Microsoft security check | `/msft-security` |
| Explain unfamiliar code | `/code-explain` |
| Write tests for existing code | `/test-writer` |
| Safe refactoring | `/refactor` |
| Check environment | `/env-setup` |
| Estimate effort | `/estimate` |
| Debug a blocker | `/diagnose` |
| Write SOW | `/sow` |
| Write handover doc | `/handover-doc` |
| Write tech doc | `/tech-doc` |
| Write user guide | `/user-guide` |
| Generate changelog | `/changelog` |
| Write an ADR | `/adr-writer` |
| Format meeting notes | `/meeting-notes` |
| Weekly client update | `/client-report` |
| New dev onboarding guide | `/onboarding` |
| Weekly architecture check | `/improve-codebase-architecture` |
| Explain architectural context | `/zoom-out` |
| Throwaway prototype | `/prototype` |

---

## Questions?

Ping Ashraf on Teams or drop a message in the dev channel.

GitHub: https://github.com/Hamood1124/ultimate-code-council