---
name: estimate
description: "Estimates development effort in hours/days per task from a PRD, issue list, or project description. Flags risks that could inflate the estimate. Sits between /sow and /council-start in the lifecycle. TRIGGER on: 'estimate this', 'how long will this take', 'effort estimate', 'time estimate', 'story points', '/estimate'. Token-efficient: reads PRD and issues first, asks minimal questions, delivers a structured breakdown."
---

# Estimate

Estimates development effort from a PRD, issue list, or project description. Built for consulting contexts where accurate estimates directly affect pricing and client trust.

---

## Phase 0 — Load context (silently)

```bash
cat docs/prd/*.md .scratch/*.md 2>/dev/null
cat CONTEXT.md 2>/dev/null

# Get existing issues if any
ls .scratch/issues/ 2>/dev/null && cat .scratch/issues/*.md 2>/dev/null | head -100

# Detect stack for complexity calibration
cat package.json 2>/dev/null | head -20
cat *.csproj 2>/dev/null | head -10
```

---

## Phase 1 — Calibration questions

Ask in one message:

```
I'll estimate the effort. A few calibration questions:

1. Who's doing the work? (senior dev / mid / junior / mixed team)
2. Is this greenfield or modifying existing code?
3. Any integrations with external systems? (D365, SharePoint, APIs)
4. Is testing / QA included in the estimate or separate?
5. Any hard deadline that might affect the approach?
```

---

## Phase 2 — Generate estimate

Break into tasks. For each task:

```markdown
### [Task name]
**Description:** [what needs to be done]
**Complexity:** Low / Medium / High / Very High
**Estimate:** [X hours] — [X days at 6hr effective day]
**Risks:** [anything that could make this take longer]
**Assumptions:** [what needs to be true for this estimate to hold]
```

**Complexity calibration:**
| Complexity | Hours | Typical for |
|------------|-------|-------------|
| Low | 2–4h | Simple UI change, config update, minor bug fix |
| Medium | 4–12h | New form, API endpoint, integration with known system |
| High | 12–24h | Complex business logic, new integration, multi-step workflow |
| Very High | 24h+ | Architecture change, major feature, unknown territory |

**Add standard buffers:**
- Code review + testing: +20% per task
- Integration testing: +1–2 days for any task with external dependencies
- UAT support: +1 day per milestone
- Buffer for unknowns: +15% on total

---

## Phase 3 — Summary table

```markdown
## Estimate Summary

| Task | Complexity | Hours | Days |
|------|------------|-------|------|
| [Task 1] | Medium | 8h | 1.5d |
| [Task 2] | High | 16h | 2.5d |
| Code review + testing | — | +20% | |
| UAT support | — | 8h | 1d |
| Buffer (15%) | — | Xh | Xd |
| **TOTAL** | | **Xh** | **Xd** |

**Estimated timeline:** X weeks (assuming X developer(s))
**Confidence:** High / Medium / Low — [reason]

### Key risks that could inflate this estimate
1. [Risk 1 — e.g. "D365 sandbox access delayed"]
2. [Risk 2 — e.g. "Requirement X is not fully defined"]

### Assumptions this estimate depends on
1. [Assumption 1]
2. [Assumption 2]
```

> *"Estimate complete. Review the risks section — these are the items most likely to cause overruns. Consider adding them to your SOW's Assumptions section."*
