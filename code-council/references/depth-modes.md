# Code Council — Depth Modes

## The menu (show this to the user, wait for reply)

```
How deep should I go?

⚡ Quick Scan
   Runs one focused pass per reviewer. Flags Blockers and Highs only.
   Skips: maintainability, style, naming, advisory notes, optimizations.
   Runs available tooling (tsc, eslint, pytest) and treats errors as Confirmed.
   Best for: "Is this safe to push right now?"
   Time: fast.

🔍 Standard Review  ← default
   Full reviewer pass at medium depth. Flags Highs and above.
   Notes Mediums briefly. Includes improvement suggestions on Highs.
   Runs full tooling suite.
   Best for: PR reviews, handovers, client deliverables, end of TDD session.
   Time: moderate.

🧠 Deep Review
   Every reviewer goes fully deep. Flags everything including Lows.
   Maintainability Critic runs fully. Integration Skeptic maps all assumptions.
   Security Auditor checks CVEs, crypto, timing attacks.
   "What I didn't review" section included.
   Runs full tooling suite + dependency audit.
   Best for: auditing legacy code, pre-launch, security-sensitive features, 
             weekly architecture check.
   Time: thorough.

Reply Quick, Standard, or Deep — or just say go for Standard.
```

## Natural language detection

| User says | Depth |
|-----------|-------|
| "quick look", "fast check", "just scan", "quick pass" | Quick |
| "deep dive", "full audit", "go deep", "thorough", "full review" | Deep |
| "go", "run it", "standard", nothing specified | Standard |
| Triggered by orchestrator auto-mode | Standard |

## Per-reviewer depth behavior summary

| Reviewer | Quick | Standard | Deep |
|----------|-------|----------|------|
| Correctness Judge | Blockers + Highs | Highs + note Mediums | Everything |
| Security Auditor | Confirmed Blockers only | All Confirmed + Suspected | + CVEs, crypto, timing |
| Performance Engineer | Confirmed crash-level only | All Confirmed + Suspected | + Advisory optimizations |
| Maintainability Critic | **SKIP** | Medium+ only | Everything incl. Lows |
| Requirements Verifier | Confirmed mismatches only | Full pass | Full pass |
| Integration Skeptic | Clear violations only | All risks + assumptions | Full assumption map |

## Severity thresholds by depth

| Severity | Quick | Standard | Deep |
|----------|-------|----------|------|
| 🔴 Blocker | Always report | Always report | Always report |
| 🟠 High | Always report | Always report | Always report |
| 🟡 Medium | Skip | Note briefly | Full detail |
| 🟢 Low | Skip | Skip | Report |
