---
name: meeting-notes
description: "Formats meeting minutes / Minutes of Meeting (MOM) from a description, bullet points, or rough notes. Produces a clean structured document with attendees, agenda, decisions, and action items with owners and deadlines. TRIGGER on: 'write meeting notes', 'format MOM', 'meeting minutes', 'write up the meeting', '/meeting-notes'. Token-efficient: user provides raw notes, skill formats them — minimal back-and-forth."
---

# Meeting Notes

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

Turns rough meeting notes, bullet points, or a verbal description into a clean, structured Minutes of Meeting document. Fast — minimal questions, maximum output from whatever the user provides.

---

## Phase 0 — Check for existing notes

If the user has already pasted notes or described the meeting in their message → skip questions and go straight to Phase 2.

If no notes provided → go to Phase 1.

---

## Phase 1 — Gather input (only if nothing provided)

Ask in one message — do not generate anything until format is confirmed:

```
Before I format the notes, quick questions:

1. What format do you want the output in?
   - Markdown (.md) — in chat or saved as a file
   - Word document (.docx) — downloadable, easy to share
   - Both .docx and .md

2. Paste your meeting notes below — rough bullets, voice memo transcript,
   whatever you have. Also tell me:
   - Meeting title / subject
   - Date and time
   - Who attended (names and roles)
   - Any action items you remember (even if they're in the notes too)

The rougher the better — I'll clean it up.
```

---

## Phase 2 — Generate MOM

```markdown
# Minutes of Meeting

**Subject:** [title]  
**Date:** [date]  
**Time:** [time] — [end time if known]  
**Location / Platform:** [in-person / Teams / Zoom / etc.]  
**Prepared by:** [from context or leave blank]

---

## Attendees

| Name | Role / Company |
|------|---------------|
| [name] | [role] |

**Apologies / Not present:** [if any]

---

## Agenda

1. [agenda item 1]
2. [agenda item 2]
...

---

## Discussion & Decisions

### [Agenda Item 1]
**Discussion:** [summary of what was discussed — neutral, factual]  
**Decision:** [what was decided, if anything]  

### [Agenda Item 2]
...

---

## Action Items

| # | Action | Owner | Due Date |
|---|--------|-------|----------|
| 1 | [what needs to be done] | [name] | [date] |
| 2 | | | |

---

## Next Meeting

**Date:** [if agreed]  
**Agenda items carried forward:** [if any]

---
*Please review and flag any corrections within 24 hours.*
```

**Rules for generating:**
- Decisions must be clearly marked — don't bury them in discussion
- Action items must have an owner — if the notes say "someone should..." flag it as `[Owner TBD]`
- Keep discussion summaries factual and neutral — no opinions
- If something is unclear from the notes, mark it `[Clarification needed]` rather than guessing

---

## Phase 3 — Deliver

Deliver in the format confirmed in Phase 1:
- Markdown → deliver in chat or as `MOM-[YYYY-MM-DD]-[subject-slug].md`
- Word → generate `.docx` file with suggested filename: `MOM-[YYYY-MM-DD]-[subject-slug].docx`
- Both → generate both `.docx` and `.md`

> *"MOM complete. Check the Action Items table — owners and due dates are the most important part to verify before sharing."*
