---
name: onboarding
description: "Generates a new developer onboarding guide for a project — how to set up locally, what the key files are, domain terms, what not to touch, who to ask. Reads CONTEXT.md, SECURITY.md, ADRs, and the codebase. TRIGGER on: 'write onboarding guide', 'new developer guide', 'getting started guide', 'onboarding doc', '/onboarding'. Delivers as a file. Never auto-generates — always asks first."
---

# Onboarding

Generates a getting-started guide for a new developer joining the project. Reads everything that exists before asking questions — so the output is specific to the actual project, not generic boilerplate.

---

## Phase 0 — Load context (silently)

```bash
cat CONTEXT.md 2>/dev/null
cat SECURITY.md 2>/dev/null
cat docs/adr/*.md 2>/dev/null
cat README.md 2>/dev/null

# Understand structure
find . -not -path '*/node_modules/*' -not -path '*/.git/*' \
       -not -path '*/dist/*' -type f | sort

# Detect stack
cat package.json 2>/dev/null | head -30
cat *.csproj 2>/dev/null | head -10
cat requirements.txt 2>/dev/null

# Check .env.example for setup steps
cat .env.example .env.sample 2>/dev/null
```

---

## Phase 1 — Ask only what's missing

```
I'll write the onboarding guide. A few things I can't get from the code:

1. Who's the person to ask when stuck? (name + Teams/Slack handle)
2. Are there any parts of the codebase that are fragile or "don't touch without asking"?
3. Any known gotchas or things that took you time to figure out that a new dev should know upfront?
4. Any access that needs to be requested? (Azure AD, D365 sandbox, GitHub teams, etc.)
```

---

## Phase 2 — Generate onboarding guide

```markdown
# Getting Started — [Project Name]

> Welcome to the team. This guide will get you from zero to running the project
> in as little time as possible.

---

## What this project does

[Plain English. One paragraph. What problem does it solve and who uses it.]

---

## Before you start — access you'll need

Request the following before trying to set up locally:

| Access | Who to ask | How long it takes |
|--------|------------|------------------|
| [e.g. Azure AD app registration] | [Ashraf / IT team] | [1-2 days] |
| [e.g. D365 sandbox credentials] | [Project lead] | [same day] |
| [e.g. GitHub repo access] | [Ashraf] | [minutes] |

---

## Local setup — step by step

### Prerequisites
- [Runtime + version — e.g. Node.js 20.x]
- [Tool — e.g. Docker Desktop]
- [Tool — e.g. VS Code with these extensions: ...]

### Setup

```bash
# 1. Clone the repo
git clone https://github.com/[org]/[repo].git
cd [repo]

# 2. Install dependencies
[npm install / pip install / dotnet restore]

# 3. Set up environment variables
cp .env.example .env
# Fill in the values — see the table below
```

### Environment variables you need to fill in

| Variable | What it is | Where to get it |
|----------|------------|----------------|
| `D365_CLIENT_ID` | Azure AD app client ID | Ask Ashraf |
| `D365_TENANT_ID` | Azure AD tenant ID | [value — not a secret] |
| `D365_CLIENT_SECRET` | App secret | Azure Key Vault: [vault/secret name] |

```bash
# 4. Start the app
[npm run dev / python main.py / dotnet run]

# 5. Run tests to confirm everything works
[npm test / pytest / dotnet test]
```

---

## Understanding the codebase

### Key domain terms

[From CONTEXT.md — the words this project uses and what they mean]

| Term | What it means in this project |
|------|-------------------------------|
| [LeaveRequest] | [A request submitted by an employee for time off] |
| [ApprovalChain] | [...] |

### Folder structure

```
src/
├── [folder]    — [what it contains and why]
├── [folder]    — [what it contains and why]
```

### The files that matter most

Start here if you want to understand how the system works:

1. `[file]` — [why it's the best starting point]
2. `[file]` — [what it does and why it's important]
3. `[file]` — [the core business logic]

### ⚠️ Things not to touch without checking first

- `[file or folder]` — [why it's sensitive / what breaks if you change it wrong]
- `[file or folder]` — [e.g. "this handles D365 auth — changes here affect all environments"]

---

## Key architectural decisions

[From ADRs — summarized as plain English, not full ADR format]

- **[Decision]:** We use [X] instead of [Y] because [reason]. Don't change this without discussing first.
- **[Decision]:** [...]

---

## Day-to-day development

### Making a change
```bash
git checkout -b feature/your-branch-name
# make changes
git add .
git commit -m "describe what you did"
git push origin feature/your-branch-name
# open a PR — run /diff-review before asking for a review
```

### Running the code review before a PR
```
/diff-review
```

### Gotchas and things that tripped people up
- [Gotcha 1 — e.g. "The D365 sandbox resets every Sunday night — don't run tests Monday morning without seeding data first"]
- [Gotcha 2]

---

## Who to ask

| Topic | Person | How to reach them |
|-------|--------|------------------|
| Architecture / design decisions | [Ashraf] | Teams: @ashraf |
| D365 / OData questions | [name] | Teams: @name |
| Deployment / DevOps | [name] | Teams: @name |
| Client-related questions | [name] | Teams: @name |

---

## Useful commands cheat sheet

| What you want to do | Command |
|--------------------|---------|
| Start the app | `[command]` |
| Run all tests | `[command]` |
| Run a specific test | `[command]` |
| Check TypeScript errors | `npx tsc --noEmit` |
| Code review before PR | `/diff-review` |
| Explain unfamiliar code | `/code-explain` |
```

---

## Phase 3 — Deliver

Deliver as `ONBOARDING.md` in the project root.

> *"Onboarding guide complete. Share this with new team members on their first day. Update the 'Gotchas' section as new ones are discovered — it becomes more valuable over time."*
