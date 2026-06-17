---
name: adr-writer
description: "Writes a proper Architectural Decision Record (ADR) when a significant technical decision is made. Covers context, decision, consequences, and alternatives considered. Saves to docs/adr/. NEVER generates automatically — always asks first. TRIGGER on: 'write an ADR', 'record this decision', 'architectural decision', 'document why we chose', '/adr-writer'. Token-efficient: asks focused questions, writes one ADR at a time."
---

# ADR Writer

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

Writes a proper Architectural Decision Record when a significant technical decision is made. ADRs are permanent records — they explain not just what was decided but why, so future developers don't re-litigate closed decisions.

---

## Phase 0 — Load context (silently)

```bash
# Check existing ADRs for numbering and format
ls docs/adr/ 2>/dev/null
cat docs/adr/*.md 2>/dev/null | head -50

# Get next ADR number
ls docs/adr/ADR-*.md 2>/dev/null | wc -l
```

---

## Phase 1 — Capture the decision

Ask in one message — do not write the ADR until answered:

```
I'll write the ADR. A few things first:

1. What format do you want?
   - Markdown (.md) — saved to docs/adr/ in the project
   - Word document (.docx) — downloadable

2. What was decided? (one sentence — the actual decision)
3. What problem were you solving? (why was a decision needed)
4. What alternatives did you consider? (even briefly)
5. What are the downsides or trade-offs of what you chose?
6. Is this decision reversible or hard to undo?
```

Wait for answers. If the user already described the decision in the conversation above, extract from context and confirm instead of asking from scratch.

---

## Phase 2 — Generate ADR

Use this format (MADR — Markdown Architectural Decision Records):

```markdown
# ADR-[NNN] — [Short title of the decision]

**Date:** [today's date]  
**Status:** Accepted  
**Deciders:** [from session context or ask]

---

## Context

[What situation or problem made this decision necessary.
What constraints existed. What was at stake.]

## Decision

[The decision that was made. Clear and direct — one paragraph.]

## Alternatives Considered

### Option A — [What you chose]
**Pros:** [why it was chosen]  
**Cons:** [trade-offs accepted]

### Option B — [Alternative 1]
**Pros:** [what was attractive about it]  
**Cons:** [why it was not chosen]

### Option C — [Alternative 2 if applicable]
**Pros:** ...  
**Cons:** ...

## Consequences

**Positive:**
- [What this decision enables or improves]

**Negative / Trade-offs:**
- [What this decision costs or constrains]

**Risks:**
- [What could go wrong, and how it's mitigated]

## Reversibility

[Easy to reverse / Hard to reverse / Irreversible — and why]

---
*This ADR was written with /adr-writer from the Ultimate Code Council.*
```

---

## Phase 3 — Save

Deliver in the format confirmed in Phase 1:
- Markdown → save as `docs/adr/ADR-[NNN]-[slug].md` (create `docs/adr/` if it doesn't exist)
- Word → generate `.docx` file

> *"ADR-[NNN] saved. This decision is now part of the project's permanent record. The Security Council and Integration Skeptic will reference it in future reviews."*
