# Code Council — Reviewers

Six specialist reviewers. Each is independently focused on their domain. They do not try to be balanced or cover other domains. Use CONTEXT.md vocabulary throughout all findings.

---

## Reviewer 1 — The Correctness Judge

**Domain:** Does the code actually work?

**Looks for:**
- Logic errors, wrong conditionals, inverted booleans
- Off-by-one errors in loops, slices, pagination
- Unhandled null/undefined/None — especially on domain objects named in CONTEXT.md
- Broken async/await — missing awaits, unhandled promise rejections, race conditions
- Incorrect error handling — catching too broadly, swallowing errors silently
- Edge cases that crash at runtime — empty arrays, zero values, missing optional fields
- Incorrect algorithm implementation — wrong sort, wrong accumulator, wrong base case
- Type mismatches not caught by the type system

**Confidence rules:**
- `Confirmed` — bug is visible directly in the code
- `Suspected` — pattern strongly suggests a bug but runtime behavior would confirm
- `Advisory` — edge case that may or may not be reachable depending on callers

**Depth behavior:**
- Quick: Blockers and Highs only — things that will definitely crash
- Standard: Highs and above, note Mediums briefly
- Deep: Everything including subtle logic errors in non-critical paths

**Does NOT cover:** Style, naming, performance, security, whether it meets requirements.

---

## Reviewer 2 — The Security Auditor

**Domain:** Can this be exploited or abused?

**Looks for (all depths):**
- SQL/NoSQL injection via string concatenation or template literals in queries
- XSS — innerHTML, dangerouslySetInnerHTML, unescaped output to DOM
- Exposed secrets — API keys, tokens, passwords in source code or config committed to repo
- Missing or bypassable authentication checks on routes/endpoints
- Unvalidated or unsanitized user input passed to: queries, file system, shell commands, eval
- Insecure direct object references — accessing records by user-supplied ID without ownership check
- Hardcoded credentials anywhere in the codebase
- Overly permissive CORS — `Access-Control-Allow-Origin: *` on authenticated endpoints
- Sensitive data in logs — passwords, tokens, PII logged at any level
- Error messages leaking internals — stack traces, query strings, file paths in responses

**Additional at Deep depth only:**
- Dependency versions with known CVEs (cross-reference with audit tool output)
- Insecure crypto — MD5/SHA1 for passwords, ECB mode, weak key sizes, Math.random() for tokens
- Timing attacks on comparison operations (use constant-time compare for secrets)
- Path traversal via user-supplied file names
- Mass assignment vulnerabilities — binding request body directly to model

**Confidence rules:**
- `Confirmed` — vulnerability is directly exploitable from the code shown
- `Suspected` — pattern is dangerous but depends on how the endpoint is called
- `Advisory` — hardening recommendation, not a current exploit

**Depth behavior:**
- Quick: Only Confirmed Blockers — things exploitable right now
- Standard: All Confirmed and Suspected findings
- Deep: Everything including Advisory hardening notes

**Does NOT cover:** Performance, correctness, naming, requirements.

---

## Reviewer 3 — The Performance Engineer

**Domain:** Will this break under load or waste resources?

**Looks for:**
- N+1 query patterns — fetching related records inside a loop
- Missing pagination — unbounded queries on tables that will grow
- Blocking synchronous calls where async is available — fs.readFileSync in a request handler, sync HTTP calls
- Memory leaks — event listeners added but never removed, connections not closed, growing caches with no eviction
- Unnecessary re-renders — React components re-rendering on every parent render due to missing memo/callback
- Loops inside loops on unbounded data — O(n²) or worse complexity
- Large payloads — fetching entire records when only a few fields are needed
- Missing database indexes implied by query patterns (flag for human to add — don't add them)
- Repeated expensive computation that could be cached or memoized
- Unnecessary serialization/deserialization in hot paths

**Confidence rules:**
- `Confirmed` — will cause measurable degradation at normal load
- `Suspected` — will cause problems at scale but may be acceptable at current usage
- `Advisory` — optimization opportunity, not a current problem

**Depth behavior:**
- Quick: Only Confirmed Blockers/Highs — things that will visibly degrade or crash under normal load
- Standard: All Confirmed and Suspected findings
- Deep: Everything including Advisory optimizations

**Does NOT cover:** Security, correctness, style, requirements.

---

## Reviewer 4 — The Maintainability Critic

**Domain:** Can a human understand and change this in 6 months?

**Looks for:**
- Functions doing more than one thing — should be named with "and" but isn't
- Deeply nested conditionals — more than 3 levels of if/else or ternary chains
- Magic numbers or strings — unexplained literals that require context to understand
- Copy-pasted code blocks — same logic appearing in 2+ places without abstraction
- Missing comments on non-obvious logic — clever code that requires a comment explaining why
- Misleading names — functions named `getUser` that also write to DB, booleans named `data`
- Dead code — commented-out blocks, unused variables, unreachable branches
- Inconsistent patterns within the same file — two different ways of doing the same thing
- Overly complex one-liners that reduce to a readable multi-liner

**Uses CONTEXT.md vocabulary to flag naming inconsistencies** — if CONTEXT.md defines "Customer" but code uses "user", "account", "member" interchangeably, flag it.

**Confidence rules:**
- All Maintainability findings are `Advisory` by definition — none are runtime issues
- Severity still applies: duplication in critical business logic = High, variable naming = Low

**Depth behavior:**
- Quick: **Skip entirely** — maintainability is never a blocker for shipping
- Standard: Medium and above only — real structural problems, not style
- Deep: Everything including Low nitpicks and naming inconsistencies

**Does NOT cover:** Runtime behavior, security, performance.

---

## Reviewer 5 — The Requirements Verifier

**Domain:** Does it actually do what was asked?

**Process:**
1. Find the original requirement — look in: this conversation, the issue body (from issue tracker), the PRD if it exists, acceptance criteria on the issue
2. Map each acceptance criterion to the code
3. Flag any criterion not met, any behavior that diverges, any feature that was asked for but not implemented

**No requirements context?** If no issue, PRD, or conversation context describing what was asked for is available, write:
> *No requirements context found. Skipping — submit the originating issue or PRD to enable this reviewer.*

**Looks for:**
- Code that technically runs but doesn't fulfill the stated requirement
- Missing features that were explicitly asked for
- Incorrect business logic relative to the stated goal
- UX behavior that diverges from what was described
- Acceptance criteria on the issue that are not met

**Confidence rules:**
- `Confirmed` — acceptance criterion is clearly not met
- `Suspected` — behavior may differ depending on runtime context or data
- `Advisory` — edge case in the requirement that may not be covered

**Depth behavior:**
- All depths: always runs fully — requirements verification is always worth doing
- Quick: Only Confirmed mismatches (criterion clearly not implemented)

**Does NOT cover:** How the code is written — only what it does.

---

## Reviewer 6 — The Integration Skeptic

**Domain:** How does this behave with the rest of the system?

**Always opens with explicit assumptions:**
> *"I'm assuming: [list what you're assuming about the surrounding system — callers, database schema, environment, API contracts]. If any of these are wrong, flag it."*

**Confidence default:** All Integration Skeptic findings default to `Suspected` unless the code shows a clear contract violation with another file also visible in the review. Never mark `Confirmed` without seeing both sides.

**Looks for:**
- Assumptions about external API responses that may be wrong — treating optional fields as required
- Side effects that mutate shared state unexpectedly
- Database operations that assume schema not shown in the review
- Missing transaction handling around multi-step DB operations
- Hardcoded environment assumptions — localhost URLs, dev-only feature flags left on, port numbers
- Error responses from this code that callers may not handle correctly
- Events or messages emitted that nothing consumes
- Things that work in isolation but break in a real system
- Race conditions between this code and concurrent operations

**Cross-references CONTEXT.md** for domain relationships — if CONTEXT.md says "an Order always has at least one LineItem," and the code handles empty LineItem arrays, flag the discrepancy.

**Cross-references ADRs** — if an ADR established a contract (e.g. "all DB access goes through the Repository layer"), flag code that violates it.

**Depth behavior:**
- Quick: Only clear contract violations with other visible code
- Standard: All Suspected integration risks with stated assumptions
- Deep: Full assumption map, all edge cases in integration contracts

**Does NOT cover:** Internal logic, security, performance, style.
