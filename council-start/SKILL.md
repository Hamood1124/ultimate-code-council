---
name: council-start
description: "Master orchestrator for the Ultimate Code Council pipeline. Single entry point that detects repo state and drives the full pipeline automatically — setup → align → plan → build → review → fix → ship. ALWAYS TRIGGER on: 'let's build X', 'start the council', 'new feature', 'I want to build', 'start a new project', 'help me build', 'council start', '/cc'. Also trigger when user starts describing something they want to create and no prior pipeline session is active. The user should never have to call individual skills manually — this orchestrator calls them in the right order automatically."
---

# Council Start — Master Orchestrator

You are the entry point for the Ultimate Code Council pipeline. Your job is to detect where the user is in the development lifecycle and drive the pipeline forward automatically. The user should only ever need to make real decisions — not manage which skill to call next.

---

## Pipeline overview

```
SETUP      → ALIGN      → PLAN       → BUILD      → REVIEW     → FIX        → SHIP
/setup*    → /grill     → /to-prd    → /tdd       → /council   → /diagnose  → ✓
            /zoom-out*   /to-issues
```

`*` = runs automatically only if needed, invisible to user unless something is missing.

---

## Phase 0 — Repo health check (always runs first, silently)

Before doing anything else, silently check:

```bash
# Check if git is initialized
ls .git 2>/dev/null || echo "NOT_A_GIT_REPO"

# Check if there's at least one commit
git rev-parse HEAD 2>/dev/null || echo "NO_COMMITS_YET"
```

If `NOT_A_GIT_REPO`:
> *"This project isn't tracked by git yet. I can run `git init` to set that up — it won't affect your code or push anything anywhere. Want me to do that?"*
- User says yes → run `git init`, then check for commits
- User says no → continue, skip all git-dependent steps

If `NO_COMMITS_YET`:
> *"Git is initialized but there are no commits yet. I'll need at least one commit for git commands to work. Want me to make an initial commit now?"*
- User says yes → run `git add . && git commit -m "Initial commit"`, continue normally
- User says no → continue, skip all git-dependent steps

```bash
# Check for CONTEXT.md
ls CONTEXT.md CONTEXT-MAP.md 2>/dev/null

# Check for SECURITY.md
ls SECURITY.md 2>/dev/null

# Check for ADRs
ls docs/adr/ 2>/dev/null

# Check for agent skills config
grep -l "Agent skills" CLAUDE.md AGENTS.md 2>/dev/null

# Check for existing PRDs / issues
ls .scratch/ docs/prd/ 2>/dev/null

# Detect Microsoft stack (for auto-triggering msft-security)
cat package.json 2>/dev/null | grep -i "spfx\|@microsoft\|@pnp\|sharepoint\|graph" | head -5
find . -name "package-solution.json" -o -name "*.csproj" 2>/dev/null | head -3
```

**Decision tree based on findings:**

| State | Action |
|-------|--------|
| No `CONTEXT.md` AND no agent skills config | → Auto-run `/setup-ashraf-skills` before anything else. Tell user: *"No project context found — setting up your repo first. This only happens once."* |
| `CONTEXT.md` exists but no `SECURITY.md` | → Auto-run `/setup-ashraf-skills` security baseline section only. Tell user: *"No security baseline found — creating SECURITY.md. Takes 2 minutes."* |
| `CONTEXT.md` exists but no agent skills config | → Auto-run `/setup-ashraf-skills` in quick mode (issue tracker + labels only, reuse existing CONTEXT.md) |
| Everything exists | → Skip setup, proceed to Phase 1 |

After setup completes (or if already set up), read:
1. `CONTEXT.md` — load the domain glossary into context
2. `SECURITY.md` — load security baseline, data classification, accepted risks, auth provider
3. `docs/adr/*.md` — load architectural decisions
4. Any existing PRDs or issues — detect pipeline position

**Never tell the user you're doing this check.** Just do it and move on.

---

## Phase 1 — Detect pipeline position

Based on what you found, determine where the user is:

### Position A — Fresh start (no PRD, no issues, no recent code)
User wants to build something new.
→ Go to **Stage 1: Align**

### Position B — Has context, needs a plan
User has described what they want (in this conversation or prior) but no PRD exists yet.
→ Skip grilling if context is rich enough, go straight to **Stage 2: Plan**

### Position C — Has PRD, needs issues
A PRD exists but hasn't been broken into issues yet.
→ Go to **Stage 3: Issues**

### Position D — Has issues, needs to build
Issues exist and user wants to work on one.
→ Go to **Stage 4: Build**

### Position E — Has code, needs review
Code has been written (in this session or user submitted files).
→ Go to **Stage 5: Review**

### Position F — Has review results, has blockers
Council ran and found Blockers/Highs.
→ Go to **Stage 6: Fix**

Tell the user where you've detected they are and what you're going to do next. One line. Then proceed.

---

## Stage 1 — Align (`/grill-with-docs`)

Run the `/grill-with-docs` skill. Interview the user relentlessly about what they want to build. Walk every branch of the design tree. One question at a time. Wait for answers.

**Auto-advance:** When you've reached a shared understanding (user has answered all branches, no open questions remain) → tell the user: *"We have a clear picture. Creating the PRD now."* → auto-advance to Stage 2.

**User can skip:** If user says "skip grilling", "I know what I want", "just start" → go straight to Stage 2 using whatever context exists.

---

## Stage 2 — Plan (`/to-prd` → `/to-issues`)

### 2a. PRD
Run `/to-prd`. Synthesize the conversation into a Product Requirements Document. Cover:
- Problem statement
- Solution
- User stories (extensive)
- Implementation decisions
- Testing decisions
- Out of scope

Post to issue tracker. Show the user the PRD and ask: *"Does this capture everything? Reply yes to continue or tell me what to change."*

### 2b. Issues
Once PRD is approved, run `/to-issues`. Break into vertical-slice issues. Present the breakdown, confirm with user, then publish to issue tracker.

**Auto-advance:** After issues are published → tell the user: *"Issues are ready. Which one do you want to start with?"* → wait for selection → advance to Stage 3.

---

## Stage 3 — Build (`/tdd`)

Run `/tdd` on the selected issue.

Before writing any code:
1. Read `CONTEXT.md` — use domain vocabulary in all test names and interfaces
2. Check ADRs in the area being touched — don't re-litigate closed decisions
3. Confirm interface changes with user
4. Confirm which behaviors to test

Then: red → green → refactor. One test at a time. Vertical slices only.

**Auto-advance:** When all tests pass and the issue's acceptance criteria are met → tell the user: *"Feature complete. Running the Code Council before we ship."* → auto-advance to Stage 4.

**User can trigger manually:** "review this", "run the council", "ship check" → jump straight to Stage 4.

---

## Stage 4 — Review (`/code-council` + security layer)

### 4a — Code Council
Run the full Code Council. See `code-council/SKILL.md` for the complete protocol.

Quick summary of what happens:
1. Present depth menu (Quick / Standard / Deep) — default Standard
2. Run context capture (reads CONTEXT.md + SECURITY.md + ADRs, classifies files)
3. Run all 6 reviewers using domain vocabulary from CONTEXT.md
4. Run conflict check
5. Produce verdict: SHIP IT / SHIP WITH FIXES / DO NOT SHIP

### 4b — Security layer (auto-triggered when relevant)

After the Code Council runs, check whether changed files touch security-sensitive areas:

```bash
# Check if changed files touch auth, APIs, data, or external integrations
git diff --name-only HEAD 2>/dev/null | grep -iE \
  "auth|login|session|token|oauth|jwt|password|credential|\
   api|route|endpoint|controller|handler|\
   database|db|query|repository|model|service|\
   secret|key|config|env"
```

**Auto-trigger rules:**

| Condition | Action |
|-----------|--------|
| Changed files touch auth / session / tokens | → Auto-run `/security-council` (Auth + Secrets agents) |
| Changed files touch API routes / controllers | → Auto-run `/security-council` (API Surface agent) |
| Changed files touch DB / queries / models | → Auto-run `/security-council` (Data Access agent) |
| Any of the above + Microsoft stack detected | → Also auto-run `/msft-security` |
| Before every SHIP IT verdict | → Always auto-run `/secrets-scan` |
| User explicitly asks for security review | → Run full `/security-council` (all 5 agents) |

Tell the user when security layer is triggered:
> *"Changed files touch [auth/API/data] — running Security Council before clearing for ship."*

### 4c — Verdict logic (combined)

| Code Council | Security Council | Final verdict |
|-------------|-----------------|---------------|
| SHIP IT | SECURE | ✅ SHIP IT |
| SHIP IT | HARDEN BEFORE SHIP | ⚠️ SHIP WITH SECURITY FIXES |
| SHIP IT | DO NOT SHIP | 🔴 DO NOT SHIP |
| SHIP WITH FIXES | Any | ⚠️ SHIP WITH FIXES (resolve all) |
| DO NOT SHIP | Any | 🔴 DO NOT SHIP (blockers first) |

---

## Stage 5 — Fix (`/diagnose`)

For each 🔴 Blocker found by the council:

Tell the user:
```
🔴 CC-001 requires diagnosis before this can ship.
Running /diagnose on CC-001 — [short description of the blocker].
```

Run the `/diagnose` skill protocol on the specific issue:
- Build a feedback loop (failing test, curl, CLI, etc.)
- Reproduce
- Hypothesise (3–5 ranked)
- Instrument
- Fix + regression test
- Cleanup

**Auto-advance:** After each Blocker is fixed → run a delta review (council re-checks only the changed code) → if clean, advance to next Blocker → when all Blockers resolved → re-run full council → loop back to Stage 4.

---

## Stage 6 — Ship

When council verdict is SHIP IT (either first run or after fixes):

```
✅ Pipeline complete.

Summary:
- Feature: [name]
- Issues closed: [list]
- Council: SHIP IT
- Issues fixed: [CC-IDs if any]
- Tests: passing

Ready to merge.
```

Then ask — **one message, user picks what they need, nothing runs without a yes:**

```
What would you like to do next?

DOCUMENTATION
📄 /handover-doc      — project handover for client or team
📘 /tech-doc          — technical documentation for the codebase
👤 /user-guide        — end user guide (non-technical)
📝 /changelog         — changelog entry from this session
🏛️  /adr-writer       — write an ADR for decisions made today
🗒️  /meeting-notes    — format meeting minutes / MOM
📊 /client-report     — weekly client status update

MAINTENANCE
🏗️  /improve-codebase-architecture  — check for architectural debt
🔍 /diff-review                     — review changes before merging

Reply with which one(s) you want, or skip to finish.
```

Wait for explicit reply. Nothing runs automatically. If user says skip or nothing → done.

---

## Cross-cutting rules

**CONTEXT.md is the source of truth.** Every stage reads it. If a stage introduces a new domain term, update CONTEXT.md immediately — don't batch.

**ADRs are closed decisions.** Don't re-open them unless the friction is real and significant. If the council flags something that contradicts an ADR, surface it clearly but don't override silently.

**Never make the user call a skill.** If the next step is obvious, do it. Only pause for:
- Real decisions (picking an issue, approving PRD, choosing depth)
- User grilling answers
- Confirmation before posting to issue tracker

**If user interrupts the pipeline** ("actually, let's change X") → incorporate the change, update CONTEXT.md if needed, and resume from the current stage.

**Session memory:** Track pipeline position, all CC-IDs, open issues, and domain terms resolved in this session. On re-entry to the same session, resume from where you left off.
