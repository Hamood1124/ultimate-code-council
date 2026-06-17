---
name: user-guide
description: "Generates an end-user guide for non-technical users of the system. Reads the PRD and requirements to understand what the system does, then writes plain-English instructions with steps, tips, and FAQs. NEVER generates automatically — always asks first. TRIGGER on: 'write user guide', 'end user documentation', 'user manual', 'how to use guide', '/user-guide'. Token-efficient: scopes to specific features if requested, delivers as file."
---

# User Guide

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

Generates plain-English documentation for the actual users of the system — not developers. No technical jargon, no code snippets. Written for people who just need to know how to use the thing.

---

## Phase 0 — Load context (silently)

```bash
# Load requirements to understand what the system does
cat docs/prd/*.md .scratch/*.md 2>/dev/null
cat CONTEXT.md 2>/dev/null

# Understand the UI surface
find . -not -path '*/node_modules/*' \
  \( -name "*.page.*" -o -name "*.view.*" -o -name "*.screen.*" \
     -o -name "*Page*" -o -name "*Form*" -o -name "*Dialog*" \) \
  --include="*.tsx" --include="*.jsx" 2>/dev/null | head -20
```

---

## Phase 1 — Format + scope (ask before writing anything)

Ask in one message — do not generate any content until this is answered:

```
Before I write anything, two quick questions:

1. What format do you want the output in?
   - Markdown (.md) — stays in chat or saved as a file
   - Word document (.docx) — downloadable, client-friendly

2. Do you want the full guide or specific features only?
   Also: who are the users? (e.g. HR managers, employees, finance team)
   Any known pain points or things users find confusing?
```

Wait for answers. Do not write a single line of the guide until format is confirmed.

---

## Phase 2 — Generate

Write in plain English. Rules:
- No technical terms unless explained immediately after
- Use "you" throughout — talk directly to the user
- Every action is a numbered step
- Screenshots described as: *[Screenshot: shows the X button in the top right corner]*
- Include tips (`💡 Tip:`) and warnings (`⚠️ Note:`) where relevant

#### Structure:
```
# [System Name] — User Guide

## Getting Started
How to log in, first-time setup, where to find things

## [Feature 1 — from PRD]
What it does (one sentence)
Step-by-step how to use it
Common questions about this feature

## [Feature 2]
...

## Troubleshooting
Most common issues and how to fix them

## FAQ
Questions and plain-English answers

## Getting Help
Who to contact, how to raise a support request
```

---

## Phase 3 — Deliver

Deliver in the format the user confirmed in Phase 1:
- Markdown → deliver as `USER-GUIDE.md` in the project root or in chat
- Word → generate `.docx` file

> *"User guide complete. Recommend having a non-technical team member read through it before sharing — they'll catch any jargon that slipped through."*
