---
name: tech-doc
description: "Generates technical documentation for a codebase — API reference, component docs, data flow, environment setup, dependency guide. Reads actual code, not just comments. NEVER generates automatically — always asks the user first. TRIGGER on: 'write technical docs', 'create tech doc', 'document this codebase', 'API docs', 'technical documentation', '/tech-doc'. Token-efficient: scopes to what the user needs, generates in sections, delivers as file."
---

# Tech Doc

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

## Phase 1 — Scope the request

Ask in one message:

```
What kind of technical documentation do you need?

A) API reference — endpoints, parameters, request/response formats
B) Component docs — UI components, props, usage examples
C) Data flow — how data moves through the system
D) Setup guide — how to install, configure, and run
E) Full technical docs — all of the above
F) Something specific — tell me what

Also: who's the audience? (other developers / DevOps / architects)
```

Wait for answer. Scope accordingly — if they say A, only generate A.

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

Deliver as `docs/TECHNICAL.md` or `.docx` based on user preference.

> *"Tech doc complete. API reference and component docs may need updating as the codebase evolves — consider running `/tech-doc` again after major feature additions."*
