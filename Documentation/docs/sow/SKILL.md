---
name: sow
description: "Generates a professional Statement of Work (SOW) or technical proposal from a client requirement description, email, or meeting notes. Pre-project skill — runs before any code is written. Reads ACME templates if available. TRIGGER on: 'write SOW', 'create statement of work', 'write a proposal', 'client proposal', 'scope document', '/sow'. Token-efficient: structured interview first, one clean document output. Never auto-generates — always asks first."
---

# Statement of Work

Generates a professional SOW or technical proposal from a client requirement. Pre-project skill — this runs before `/council-start`, not after. Reads ACME-specific templates from `templates/` if they exist.

---

## Phase 0 — Load templates (silently)

```bash
# Load ACME templates and standard terms
cat templates/sow-example.md 2>/dev/null
cat templates/acme-terms.md 2>/dev/null

# Load any client context from current session
cat CONTEXT.md 2>/dev/null
```

---

## Phase 1 — Structured interview

Ask all questions in one message — don't trickle them:

```
I'll write the SOW. Tell me what you know — answer what you can, skip what you don't:

CLIENT
1. Client name and contact person
2. Industry / what they do (briefly)

PROJECT
3. What does the client want built or implemented?
4. What problem are they trying to solve?
5. Any existing systems involved? (D365, SharePoint, etc.)
6. Any specific technology requirements or constraints?

SCOPE
7. What's explicitly IN scope?
8. What's explicitly OUT of scope? (critical — protect ACME)
9. Any phasing? (Phase 1 now, Phase 2 later?)

DELIVERY
10. Expected timeline or deadline?
11. Any milestones the client has mentioned?
12. What does "done" look like to the client?

COMMERCIAL
13. Rough budget or pricing model? (fixed price / T&M / retainer)
    (Leave blank if not decided — I'll leave a placeholder)

ASSUMPTIONS & DEPENDENCIES
14. What do you need from the client to start? (access, data, decisions)
15. Any assumptions you're making that could affect scope?
```

Wait for answers. The more detail provided, the less the user needs to review afterward.

---

## Phase 2 — Generate SOW

```markdown
# Statement of Work

**Prepared by:** ACME Software (Almoayyed Computers Middle East)  
**Prepared for:** [Client Name]  
**Date:** [today's date]  
**Version:** 1.0  
**Reference:** [SOW-YYYY-XXX]

---

## 1. Executive Summary

[2–3 paragraph summary. What the client needs, what ACME will deliver,
and the expected outcome. Written for a non-technical reader.]

---

## 2. Project Objectives

[Numbered list of what this project achieves for the client.
Outcome-focused, not feature-focused.]

1. [Objective 1]
2. [Objective 2]

---

## 3. Scope of Work

### 3.1 In Scope

[Explicit list of what ACME will deliver. Be specific — vague scope = scope creep.]

- [Deliverable 1]
- [Deliverable 2]

### 3.2 Out of Scope

[Explicit list of what is NOT included. This section protects ACME.]

- [Exclusion 1 — e.g. "Infrastructure setup and hosting"]
- [Exclusion 2 — e.g. "Data migration from legacy system"]
- [Exclusion 3 — e.g. "User training beyond X sessions"]

### 3.3 Future Phases (if applicable)

[Items discussed but deferred to a future engagement.]

---

## 4. Deliverables

| # | Deliverable | Description | Format |
|---|-------------|-------------|--------|
| 1 | [name] | [what it is] | [file / deployment / document] |
| 2 | | | |

---

## 5. Technical Approach

[How ACME will approach the project technically. Stack choices,
methodology (Agile sprints / fixed milestones), integration approach.
Enough detail to demonstrate competence without over-committing.]

---

## 6. Project Timeline

| Milestone | Description | Target Date |
|-----------|-------------|-------------|
| Kickoff | Requirements alignment and environment setup | [date] |
| [Milestone 2] | [description] | [date] |
| UAT | Client testing and feedback period | [date] |
| Go Live | Production deployment | [date] |

**Total estimated duration:** [X weeks]

---

## 7. Assumptions

[Things ACME is assuming to be true. If these change, scope/cost may change.]

1. Client will provide [access / data / decisions] by [date]
2. [Assumption 2]
3. Existing [system] is in working order and accessible

---

## 8. Dependencies

[What ACME needs from the client to deliver:]

| Dependency | Owner | Required by |
|------------|-------|-------------|
| [e.g. Azure AD tenant access] | Client IT | [date] |
| [e.g. D365 sandbox environment] | Client | [date] |

---

## 9. Commercial Terms

**Engagement model:** [Fixed Price / Time & Materials / Retainer]  
**Estimated investment:** [PLACEHOLDER — to be confirmed]  
**Payment terms:** [from acme-terms.md template or ask]  
**Validity:** This proposal is valid for 30 days from the date above.

---

## 10. Acceptance Criteria

[How will both parties know the project is complete and accepted?]

- All deliverables in Section 4 delivered and signed off
- UAT completed with no open Blocker or High issues
- [Any specific functional criteria the client stated]

---

## 11. Terms & Conditions

[From templates/acme-terms.md if available — otherwise placeholder:]
> *Standard ACME Software terms and conditions apply.
> Full T&Cs available on request.*

---

## 12. Sign-off

| | ACME Software | [Client Name] |
|-|---------------|---------------|
| **Name** | | |
| **Title** | | |
| **Signature** | | |
| **Date** | | |

---
*This document was prepared by ACME Software. Contents are confidential.*
```

---

## Phase 3 — Deliver

Ask: *"Deliver as Word document (.docx) or markdown?"*

SOWs going to clients → always `.docx`
Internal reference → markdown is fine

Suggested filename: `SOW-[ClientName]-[ProjectName]-[Date].docx`

> *"SOW complete. Before sending to the client, review Section 3.2 (Out of Scope) and Section 7 (Assumptions) carefully — these are your protection if scope creep happens. Also fill in the commercial terms placeholder in Section 9."*
