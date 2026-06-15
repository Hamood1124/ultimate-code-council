---
name: user-guide
description: "Generates an end-user guide for non-technical users of the system. Reads the PRD and requirements to understand what the system does, then writes plain-English instructions with steps, tips, and FAQs. NEVER generates automatically — always asks first. TRIGGER on: 'write user guide', 'end user documentation', 'user manual', 'how to use guide', '/user-guide'. Token-efficient: scopes to specific features if requested, delivers as file."
---

# User Guide

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

## Phase 1 — Scope the request

Ask in one message:

```
I'll write the user guide. A few quick questions:

1. What's the name of the system/app? (what do users call it)
2. Who are the users? (e.g. HR managers, employees, finance team)
3. Do you want the full guide or specific features only?
4. Any known pain points or things users find confusing?
```

Wait for answers before writing anything.

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

Deliver as `USER-GUIDE.md` or `.docx`.

> *"User guide complete. Recommend having a non-technical team member read through it before sharing — they'll catch any jargon that slipped through."*
