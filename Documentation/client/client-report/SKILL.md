---
name: client-report
description: "Converts technical work (closed issues, CC-IDs fixed, features shipped) into a non-technical client status update. Weekly progress report written for business stakeholders, not developers. TRIGGER on: 'write client report', 'weekly update', 'client status update', 'progress report', '/client-report'. Token-efficient: reads session/git history, minimal questions, fast output. Delivers as .docx or email-ready markdown."
---

# Client Report

Turns technical work into a non-technical client update. Reads what was actually done and writes it in language a business stakeholder understands — no CC-IDs, no code, no jargon.

---

## Phase 0 — Load context (silently)

```bash
# What was done this period
git log --oneline --since="7 days ago" 2>/dev/null
git diff --name-only HEAD~10 HEAD 2>/dev/null | head -20

# Load PRD to understand features in business terms
cat docs/prd/*.md .scratch/*.md 2>/dev/null | head -100

# Load project context
cat CONTEXT.md 2>/dev/null

# Any closed issues
ls .scratch/issues/*.md 2>/dev/null | xargs grep -l "status: done\|status: closed" 2>/dev/null
```

---

## Phase 1 — Quick questions

Ask in one message:

```
I'll write the client update. Quick questions:

1. What period does this cover? (e.g. "week of June 9")
2. Client name and project name (for the header)
3. Any blockers or risks the client should know about?
4. What's planned for next week?
5. Any decisions you need from the client?
```

---

## Phase 2 — Translate technical → business language

**Translation rules:**
- CC-001 fixed → "resolved an issue that was causing [user-facing impact]"
- "Refactored the service layer" → don't mention this — not client-relevant
- "Integrated OData endpoint" → "connected the system to [client's system name] to pull live data"
- "Fixed null reference exception" → "resolved an issue that could cause the system to stop responding"
- "Deployed to staging" → "released the latest version to the test environment for review"
- Never mention: CC-IDs, SC-IDs, TypeScript, React, OData, GitHub branches, npm

---

## Phase 3 — Generate report

```markdown
# Project Status Update
**Project:** [Project Name]  
**Client:** [Client Name]  
**Period:** Week of [date]  
**Prepared by:** ACME Software

---

## Summary

[2-3 sentences. Overall progress this week. Are we on track?
Written for a CEO/manager reading on their phone.]

---

## Completed This Week

✅ [Feature/task in plain English — what the user can now do]  
✅ [Feature/task]  
✅ [Bug fixed — described by its user impact, not the technical cause]  

---

## In Progress

🔄 [What's being worked on right now]  
🔄 [Estimated completion: next week / by [date]]  

---

## Planned for Next Week

📋 [What will be worked on next]  
📋 [What will be worked on next]  

---

## Risks & Issues

[Only include if there's something the client needs to know or act on.
Skip this section entirely if there's nothing to flag.]

⚠️ [Risk/issue described in plain English — impact on timeline or delivery]  
**Action needed from client:** [what you need from them, if anything]  

---

## Decisions Needed

[Only include if the client needs to make a decision before work can continue.]

❓ [Decision needed — what are the options, what's the default if no response]  
**Response needed by:** [date]  

---

## Overall Status

| Area | Status |
|------|--------|
| Timeline | 🟢 On track / 🟡 At risk / 🔴 Delayed |
| Budget | 🟢 On track / 🟡 Monitor / 🔴 Exceeded |
| Quality | 🟢 Good / 🟡 Some issues / 🔴 Blocked |

---
*Next update: [date]. Questions? Contact Ashraf at [email/Teams handle].*
```

---

## Phase 4 — Deliver

Ask: *"Deliver as Word doc (.docx), email-ready markdown, or both?"*

For email → strip markdown formatting, keep structure readable in plain text.
For Word → generate proper `.docx` with ACME letterhead style.

> *"Client report ready. Review the 'Risks & Issues' and 'Decisions Needed' sections before sending — these are the parts clients read most carefully."*
