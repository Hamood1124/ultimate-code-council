---
name: tech-doc
description: "Generates technical documentation for a codebase — API reference, component docs, data flow, environment setup, dependency guide. Reads actual code, not just comments. NEVER generates automatically — always asks the user first. TRIGGER on: 'write technical docs', 'create tech doc', 'document this codebase', 'API docs', 'technical documentation', '/tech-doc'. Token-efficient: scopes to what the user needs, generates in sections, delivers as file."
---

# Tech Doc

---

## Phase 0 — Tooling check (silently, before anything else)

```bash
# Check if Node.js is available
node --version 2>/dev/null || echo "NODE_NOT_FOUND"

# Check if docx library is installed globally
node -e "require('docx')" 2>/dev/null || echo "DOCX_NOT_INSTALLED"
```

If `NODE_NOT_FOUND`:
> *"Node.js is required to generate Word documents. Install it from nodejs.org, then try again. In the meantime, I can deliver this as Markdown."*
→ Default to Markdown, continue.

If `DOCX_NOT_INSTALLED`:
> *"Installing the docx library — this only happens once."*
```bash
npm install -g docx
```
Then confirm installation succeeded before continuing. If install fails, default to Markdown and note it.

> Note: PDF output is not supported in Claude Code. For PDFs, use the web version at claude.ai instead.

Generates technical documentation by reading the actual codebase. Covers API reference, components, data flow, and setup guides. Scoped to exactly what the user needs — never generates everything if only part is needed.

---

## Phase 0 — Load context (silently)

```bash
cat CONTEXT.md 2>/dev/null
cat SECURITY.md 2>/dev/null
cat docs/adr/*.md 2>/dev/null

# Detect stack and entry points
cat package.json 2>/dev/null | head -30
find . -not -path '*/node_modules/*' -not -path '*/.git/*' \
  \( -name "index.ts" -o -name "index.js" -o -name "Program.cs" \
     -o -name "app.py" -o -name "main.go" \) | head -10

# Get public API surface
find . -not -path '*/node_modules/*' -not -path '*/.git/*' \
  \( -name "*.controller.*" -o -name "*.route.*" -o -name "*.api.*" \
     -o -name "*.service.*" \) | head -20
```

---

## Phase 1 — Format + scope (ask before writing anything)

Ask in one message — do not generate any content until this is answered:

```
Before I write anything, two quick questions:

1. What format do you want the output in?
   - Markdown (.md) — stays in chat or saved as a file
   - Word document (.docx) — downloadable, shareable

2. What kind of technical documentation do you need?
   A) API reference — endpoints, parameters, request/response formats
   B) Component docs — UI components, props, usage examples
   C) Data flow — how data moves through the system
   D) Setup guide — how to install, configure, and run
   E) Full technical docs — all of the above
   F) Something specific — tell me what

   Also: who's the audience? (other developers / DevOps / architects)
```

Wait for answers. Do not write a single line until format and scope are confirmed.

---

## Phase 2 — Generate scoped sections

#### API Reference (if requested)
For each endpoint/route found:
```
### POST /api/leave-requests
**Auth required:** Yes (Bearer token)
**Description:** Submit a new leave request
**Request body:**
  - leaveType: string (Annual | Sick | Emergency | Unpaid)
  - startDate: ISO date string
  - endDate: ISO date string
  - reason: string (optional)
**Response 201:** LeaveRequest object
**Response 400:** Validation error
**Response 401:** Unauthorized
```

#### Component Docs (if requested)
For each significant component:
```
### LeaveRequestForm
**Props:**
  - employeeId: string (required)
  - onSubmit: (request: LeaveRequest) => void
  - onCancel: () => void
**Description:** Renders the leave request submission form.
  Fetches leave balances on mount via D365 OData.
**Usage example:** [inline code example]
```

#### Data Flow (if requested)
Described in markdown — where data enters, how it transforms, where it goes. References CONTEXT.md domain terms throughout.

#### Setup Guide (if requested)
- Prerequisites with exact versions
- Step-by-step install
- Environment variables (names only, point to SECURITY.md for values)
- How to run tests
- How to run locally

---

## Phase 3 — Deliver

Deliver in the format confirmed in Phase 1:
- Markdown → deliver as `docs/TECHNICAL.md`
- Word → generate `.docx` file

> *"Tech doc complete. API reference and component docs may need updating as the codebase evolves — consider running `/tech-doc` again after major feature additions."*
