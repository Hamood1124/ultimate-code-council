# Dependency Agent — Reference

## Severity mapping for CVEs

| CVSS Score | Council Severity |
|------------|-----------------|
| 9.0 – 10.0 Critical | 🔴 Blocker |
| 7.0 – 8.9 High | 🟠 High |
| 4.0 – 6.9 Medium | 🟡 Medium (Standard+) |
| 0.1 – 3.9 Low | 🟢 Low (Deep only) |

## Per-ecosystem audit commands

```bash
# Node.js / npm
npm audit --json

# Node.js / yarn
yarn audit --json

# Node.js / pnpm
pnpm audit --json

# Python
pip-audit
safety check -r requirements.txt

# .NET
dotnet list package --vulnerable

# Ruby
bundle audit check --update

# Rust
cargo audit

# Go
govulncheck ./...

# Java / Maven
mvn dependency-check:check

# Java / Gradle
gradle dependencyCheckAnalyze
```

## What to look for beyond CVEs
- Packages not updated in 2+ years — check `npm outdated`, `pip list --outdated`
- Packages with abandoned maintainers — check npm registry for deprecation notices
- Packages with suspicious recent updates (supply chain attack signal)
- `devDependencies` present in production bundle — unnecessary attack surface
- Pinned versions vs ranges — `^` and `~` allow auto-update to vulnerable versions

## False positive handling
- If a vulnerable package is a devDependency and not bundled into production → downgrade one severity level
- If the specific vulnerability requires a code path not used in this project → flag as Suspected, note the unused path
- If `SECURITY.md` lists an accepted CVE with justification → downgrade to 🟢 Low advisory

---

# Security Council — Report Format

## Individual agent finding format

```markdown
### SC-001 — [Short title] [🔴 Blocker] [Confirmed] [Auth]
**Location:** `src/middleware/auth.ts` line 45
**OWASP:** A07:2021 — Identification and Authentication Failures
**What's wrong:** JWT verification does not validate the `iss` (issuer) claim.
  Any valid JWT from any issuer will be accepted.
**Attack scenario:** An attacker with a JWT from a different service
  (e.g. a staging environment) can authenticate to production.
**Why it matters:** [Data classification from SECURITY.md] — this is
  a client-facing application. Impact is High.
**Fix:** Add issuer validation to the JWT verify options:
  `{ issuer: process.env.JWT_ISSUER }`
**Stack-specific note:** [if applicable]
```

## Severity + confidence matrix

| Confidence | Blocker | High | Medium | Low |
|------------|---------|------|--------|-----|
| Confirmed | Always report | Always report | Standard+ | Deep only |
| Suspected | Always report | Always report | Standard+ | Deep only |
| Advisory | Standard+ | Standard+ | Deep only | Deep only |

## OWASP category tags

Use these exact tags on every finding:

- `[A01]` Broken Access Control
- `[A02]` Cryptographic Failures
- `[A03]` Injection
- `[A04]` Insecure Design
- `[A05]` Security Misconfiguration
- `[A06]` Vulnerable and Outdated Components
- `[A07]` Auth & Authentication Failures
- `[A08]` Software and Data Integrity Failures
- `[A09]` Security Logging and Monitoring Failures
- `[A10]` Server-Side Request Forgery

## SECURITY.md update template

After a clean or resolved review, offer to append:

```markdown
## Review log

### [DATE] — Security Council review
- **Scope:** [files reviewed]
- **Depth:** [Quick / Standard / Deep]
- **Data classification:** [Internal / Client-facing / Public]
- **Findings:** [N Blockers, N Highs, N Mediums, N Lows]
- **Resolved:** [SC-IDs fixed]
- **Accepted risks:** [SC-IDs accepted with justification]
- **Verdict:** [SECURE / HARDEN BEFORE SHIP / DO NOT SHIP]
- **Reviewed by:** Claude Code Security Council
```
