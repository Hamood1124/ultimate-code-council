---
name: handover-doc
description: "Generates a professional project handover document for when a project moves from development to client, or between team members. Reads CONTEXT.md, SECURITY.md, ADRs, PRD, and session CC/SC issue history to auto-populate content. NEVER generates automatically — always asks the user first. TRIGGER on: 'write handover', 'create handover doc', 'project handover', 'handover document', '/handover-doc'. Token-efficient: asks clarifying questions before writing, generates in sections, delivers as .md or .docx file."
---

# Handover Doc

Generates a complete project handover document. Reads everything already in the project before asking a single question — minimizes back-and-forth and saves tokens.

---

## Phase 0 — Load context (silently, before asking anything)

```bash
cat CONTEXT.md 2>/dev/null
cat SECURITY.md 2>/dev/null
cat docs/adr/*.md 2>/dev/null
cat docs/prd/*.md .scratch/*.md 2>/dev/null
# Get list of resolved issues from session if available
git log --oneline -20 2>/dev/null
```

---

## Phase 1 — Ask only what's missing (max 5 questions)

Based on what was loaded, ask only what can't be inferred. One message, all questions at once:

```
I've read the project context. Before writing the handover, I need a few things:

1. Who is this handover for? (client / new team member / internal archive)
2. What's the project status? (complete / ongoing / paused)
3. Are there any known open issues or risks to flag?
4. Who are the key contacts? (client contact, project lead, support)
5. Where do credentials/secrets live? (don't share the values — just the location)

Answer what you can, skip what's not applicable.
```

Wait for answers. Do not generate anything until answered.

---

## Phase 2 — Generate section by section

Tell the user: *"Generating handover doc in sections — let me know if you want to stop or adjust at any point."*

Generate and confirm each section before moving to the next:

### Section 1 — Project Overview
- Project name, client, dates, team members
- One-paragraph summary of what was built and why
- Technology stack (from package.json / csproj / detected stack)

### Section 2 — Architecture & Technical Decisions
- High-level architecture diagram (described in markdown)
- Key technical decisions made (from ADRs)
- Why certain approaches were chosen over alternatives

### Section 3 — Codebase Guide
- Folder structure with purpose of each key directory
- Entry points and how the app starts
- Key files and what they do
- What NOT to touch without understanding first

### Section 4 — Environment & Deployment
- Prerequisites (Node version, .NET version, etc.)
- Environment variables needed (names only, not values — point to SECURITY.md)
- How to run locally
- How to deploy to staging / production
- Any manual steps required

### Section 5 — Known Issues & Open Items
- Any CC/SC issues that were accepted but not fixed (from session history)
- Known limitations or technical debt
- Recommended next steps

### Section 6 — Contacts & Access
- Key contacts with roles
- Where to find credentials (Azure Key Vault / .env location / etc.)
- Repo access, deployment access, third-party service accounts

### Section 7 — Maintenance Guide
- How to update dependencies
- How to run tests
- Common issues and how to fix them
- Who to contact for support

---

## Phase 3 — Deliver

Ask: *"Deliver as markdown file or Word document (.docx)?"*

- Markdown → deliver as `HANDOVER.md` in project root
- Word → generate `.docx` file

Always end with:
> *"Handover doc complete. Review Section 4 (deployment) and Section 6 (contacts) carefully before sharing — these often need manual updates."*
