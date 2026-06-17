---
name: handover-doc
description: "Generates a professional project handover document for when a project moves from development to client, or between team members. Reads CONTEXT.md, SECURITY.md, ADRs, PRD, and session CC/SC issue history to auto-populate content. NEVER generates automatically — always asks the user first. TRIGGER on: 'write handover', 'create handover doc', 'project handover', 'handover document', '/handover-doc'. Token-efficient: asks clarifying questions before writing, generates in sections, delivers as .md or .docx file."
---

# Handover Doc

Generates a complete project handover document. Reads everything already in the project before asking a single question — minimizes back-and-forth and saves tokens.

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

## Phase 1 — Format + missing context (ask before writing anything)

Based on what was loaded, ask only what can't be inferred. One message, all questions at once — do not generate any content until answered:

```
Before I write anything:

1. What format do you want the output in?
   - Markdown (.md) — stays in project root, zero dependencies
   - Word document (.docx) — client-friendly, downloadable

2. Who is this handover for? (client / new team member / internal archive)
3. What's the project status? (complete / ongoing / paused)
4. Any known open issues or risks to flag?
5. Who are the key contacts? (client contact, project lead, support)
6. Where do credentials/secrets live? (don't share the values — just the location)
```

Wait for answers. Do not write a single line of the document until format is confirmed.

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

Deliver in the format confirmed in Phase 1:
- Markdown → deliver as `HANDOVER.md` in project root
- Word → generate `.docx` file using the docx library

Always end with:
> *"Handover doc complete. Review Section 4 (deployment) and Section 6 (contacts) carefully before sharing — these often need manual updates."*
