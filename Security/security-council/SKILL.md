---
name: security-council
description: "Deep five-agent security review for any codebase. Covers auth & session, secrets, data access, API surface, and dependencies. Language-agnostic — works on any stack. Reads SECURITY.md for project context (auth provider, data classification, accepted risks) and adjusts severity accordingly. ALWAYS TRIGGER on: 'security review', 'security audit', 'run security council', 'check auth', 'check my security', 'is this secure', 'security check'. Auto-triggered by council-start when changed files touch auth, APIs, database, or external integrations. Issue IDs: SC-001, SC-002..."
---

# Security Council

Five specialist security agents independently review the codebase from different attack angles. Each agent focuses exclusively on their domain. The synthesis identifies the highest-risk issues and produces a clear security verdict.

Issue IDs: `SC-001`, `SC-002`... (separate from CC- and SS- series)

**Reference files — read when indicated:**

| File | Read when |
|------|-----------|
| `references/auth-agent.md` | Before running Auth & Session Agent |
| `references/data-agent.md` | Before running Data Access Agent |
| `references/api-agent.md` | Before running API Surface Agent |
| `references/dependency-agent.md` | Before running Dependency Agent |
| `references/report-format.md` | When writing the report and verdict |

---

## Phase 0 — Context load (always, silently)

```bash
# Load project security baseline
cat SECURITY.md 2>/dev/null

# Load domain vocabulary
cat CONTEXT.md 2>/dev/null

# Detect stack
cat package.json 2>/dev/null | head -30
cat requirements.txt pyproject.toml 2>/dev/null | head -20
cat *.csproj go.mod Cargo.toml 2>/dev/null | head -20

# Check auth-related files
find . -not -path '*/node_modules/*' -not -path '*/.git/*' \
  \( -name "*auth*" -o -name "*login*" -o -name "*session*" \
     -o -name "*token*" -o -name "*oauth*" -o -name "*jwt*" \
     -o -name "*middleware*" -o -name "*guard*" \) | sort

# Check data access files
find . -not -path '*/node_modules/*' -not -path '*/.git/*' \
  \( -name "*repository*" -o -name "*service*" -o -name "*model*" \
     -o -name "*database*" -o -name "*db*" -o -name "*query*" \
     -o -name "*store*" -o -name "*dao*" \) | sort

# Check API/route files
find . -not -path '*/node_modules/*' -not -path '*/.git/*' \
  \( -name "*route*" -o -name "*controller*" -o -name "*api*" \
     -o -name "*endpoint*" -o -name "*handler*" \) | sort
```

**Extract from SECURITY.md:**
- Data classification: internal / client-facing / public internet
- Auth provider in use
- Accepted risks to exclude from flagging
- Known public endpoints

Use data classification to calibrate severity across all agents:
- Public internet → everything at maximum severity
- Client-facing → High and above treated as Blockers
- Internal only → standard severity scale

---

## Phase 1 — Depth selection

Present this menu and wait for reply:

```
How deep should the security review go?

⚡ Quick  
   Auth and secrets only. Fastest path to "is this exploitable right now?"
   Best for: quick check before a PR on a security-sensitive file.

🔍 Standard  ← default
   All 5 agents at medium depth. Full OWASP coverage.
   Best for: any feature touching auth, APIs, or data.

🧠 Deep
   All 5 agents fully deep. Dependency CVE scan included.
   Threat modeling per endpoint. Full attack surface mapping.
   Best for: pre-launch, client deliverables, pen-test prep.
```

If triggered by orchestrator in auto-mode → Standard, skip menu.

---

## Phase 2 — Five security agents

Read the relevant reference file before each agent runs.

### Agent 1 — Auth & Session Agent
*Read `references/auth-agent.md` now.*

**Domain:** Authentication, authorization, session management.

Looks for:
- Missing authentication on protected routes/endpoints
- JWT: algorithm confusion (`alg: none`), weak secrets, missing expiry (`exp`), missing issuer validation
- OAuth: state parameter missing (CSRF), implicit flow used where code flow should be, token leakage in URLs
- Session fixation — session ID not regenerated after login
- Privilege escalation — role checks client-side only, missing server-side validation
- Insecure password handling — plaintext storage, weak hashing (MD5, SHA1, unsalted)
- Missing rate limiting on auth endpoints — brute force possible
- Hardcoded admin credentials or backdoor accounts
- "Remember me" tokens stored insecurely
- Password reset flows — predictable tokens, no expiry, no single-use enforcement

Cross-references `SECURITY.md` for: auth provider in use, expected token format, known public routes.

Issue format: `SC-001 [Auth]`

---

### Agent 2 — Secrets Agent
*Runs a contextual (not just pattern-based) secrets check.*

**Domain:** Credential and secret exposure in context.

Unlike `/secrets-scan` which is pattern-only, this agent understands *context*:
- Is this secret actually reachable by an attacker?
- Is it in a file that gets bundled into client-side code?
- Is it in a config that gets logged?
- Is it passed through a URL parameter (appears in server logs)?

Looks for:
- Secrets in client-side bundles — anything in `public/`, `static/`, or bundled JS
- Secrets passed in URL query parameters
- Secrets logged via `console.log`, `print`, `logger.*`, `Debug.WriteLine`
- Secrets in error responses sent to clients
- API keys with excessive permissions relative to what the code actually uses
- Secrets in Docker `ENV` instructions (visible in image history)
- Secrets in CI/CD config files committed to repo

Cross-references `SECURITY.md` for accepted/known secret patterns.

Issue format: `SC-00N [Secrets]`

---

### Agent 3 — Data Access Agent
*Read `references/data-agent.md` now.*

**Domain:** How the code reads and writes data.

Looks for:
- SQL injection — string concatenation in queries, f-strings with user input
- NoSQL injection — unvalidated objects passed to MongoDB `find()`, Mongoose queries
- ORM misuse — raw query fallback with user input, `executeRawUnsafe`
- Mass assignment — binding entire request body to a model without allowlist
- Missing input validation — no schema validation on incoming data, trusting client-supplied types
- Insecure deserialization — deserializing untrusted data without type checking
- Path traversal — user input used in file paths without sanitization
- IDOR (Insecure Direct Object Reference) — fetching records by ID without ownership check
- Over-fetching — returning full objects when only specific fields are needed
- Missing database-level access controls — app using a superuser DB account

Cross-references `CONTEXT.md` for domain entity names and relationships.
Cross-references `SECURITY.md` for data classification and approved query patterns.

Issue format: `SC-00N [Data]`

---

### Agent 4 — API Surface Agent
*Read `references/api-agent.md` now.*

**Domain:** The exposed API surface and how it's protected.

**Opens with explicit assumptions:**
> *"I'm assuming: [list what endpoints are visible, what auth middleware is applied, what the intended public surface is]. Correct me if wrong."*

Looks for:
- Unauthenticated endpoints that should be protected
- Missing authorization on authenticated endpoints (authn ≠ authz)
- BOLA (Broken Object Level Authorization) — user A can access user B's resources
- BFLA (Broken Function Level Authorization) — regular user can call admin functions
- Mass assignment via API — `PUT /users/:id` accepts `{ role: "admin" }`
- Missing rate limiting — no throttling on sensitive endpoints
- Overly permissive CORS — `Access-Control-Allow-Origin: *` on credentialed endpoints
- HTTP methods not restricted — `DELETE` available where only `GET` should be
- Sensitive data in GET parameters (shows in logs, browser history, referrer headers)
- Missing CSRF protection on state-changing endpoints
- GraphQL: introspection enabled in production, no query depth limiting, no rate limiting
- Webhook endpoints: no signature verification

Cross-references `SECURITY.md` for intentionally public endpoints to avoid false positives.

Issue format: `SC-00N [API]`

---

### Agent 5 — Dependency Agent

**Domain:** Known vulnerabilities in third-party dependencies.

```bash
# Run available dependency auditing tools
npm audit --json 2>/dev/null | head -100
yarn audit --json 2>/dev/null | head -100
pip-audit 2>/dev/null | head -50
safety check 2>/dev/null | head -50
dotnet list package --vulnerable 2>/dev/null | head -50
bundle audit 2>/dev/null | head -50
cargo audit 2>/dev/null | head -50
```

Also check manually:
- Dependencies that haven't been updated in 2+ years (check `package.json` dates)
- Dependencies with known abandoned maintainers
- Transitive dependencies pulling in vulnerable versions
- `devDependencies` that shouldn't be in production builds

At **Quick** depth: skip this agent entirely.
At **Standard** depth: run audit tools only, report Critical and High CVEs.
At **Deep** depth: full analysis including Medium CVEs and outdated packages.

Issue format: `SC-00N [Deps]` with CVE number where available.

---

## Phase 3 — Conflict and coverage check

After all five agents complete:

1. Check for overlapping findings — if two agents flagged the same issue from different angles, merge into one finding and note both perspectives
2. Check coverage against OWASP Top 10 — note which categories were checked and which weren't visible in the code
3. Cross-check against `SECURITY.md` accepted risks — downgrade confirmed accepted risks to 🟢 Low advisory

---

## Phase 4 — Security verdict

```markdown
## 🛡️ Security Council Verdict

**Verdict:** [SECURE / HARDEN BEFORE SHIP / DO NOT SHIP]
**Data classification:** [Internal / Client-facing / Public internet]
**Auth provider:** [from SECURITY.md or detected]

| Severity | Count | SC-IDs |
|----------|-------|--------|
| 🔴 Blocker | N | SC-001, SC-004 |
| 🟠 High | N | SC-002, SC-003 |
| 🟡 Medium | N | SC-005 |
| 🟢 Low | N | SC-006 |

**Verdict logic:**
- Any 🔴 Blocker → DO NOT SHIP
- Any 🟠 High → HARDEN BEFORE SHIP
- Only 🟡 Medium / 🟢 Low → SECURE (with notes)

### OWASP Top 10 coverage
| Category | Status |
|----------|--------|
| A01 Broken Access Control | ✅ Checked |
| A02 Cryptographic Failures | ✅ Checked |
| A03 Injection | ✅ Checked |
| A04 Insecure Design | ⚠️ Partial — no threat model available |
| A05 Security Misconfiguration | ✅ Checked |
| A06 Vulnerable Components | ✅ Checked |
| A07 Auth & Session Failures | ✅ Checked |
| A08 Integrity Failures | ✅ Checked |
| A09 Logging & Monitoring | ⚠️ Not visible in this scope |
| A10 SSRF | ✅ Checked |

### What this review did not cover
[Runtime behavior, infrastructure config, network-level controls,
third-party service security, penetration testing]

### The one thing to fix first
SC-XXX — [highest priority finding]
```

**Verdict → next step:**
- DO NOT SHIP → route each Blocker to `/diagnose`
- HARDEN BEFORE SHIP → offer to fix Highs directly in repo
- SECURE → update `SECURITY.md` with review date and scope

---

## Phase 5 — SECURITY.md update offer

After every clean or fixed review, offer:

> *Want me to update `SECURITY.md` with the results of this review? I'll log the date, scope, findings resolved, and any accepted risks so future reviews have a baseline.*
