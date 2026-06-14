# Data Access Agent — Reference

## Injection prevention checklist

### SQL
- [ ] All queries use parameterized statements or prepared statements
- [ ] No string concatenation or interpolation in SQL: `"SELECT * FROM users WHERE id = " + userId` → BLOCKER
- [ ] ORM raw query fallbacks (`executeRaw`, `$queryRaw`, `raw()`) checked for user input
- [ ] Stored procedures: parameters passed correctly, not concatenated inside the proc

### NoSQL (MongoDB, Firestore, DynamoDB)
- [ ] Query operators (`$where`, `$gt`, `$regex`) not built from user input
- [ ] Mongoose: `find({ [userInput]: value })` pattern checked — key injection risk
- [ ] Input validated as expected type before query — user can't send `{ $gt: "" }` where string expected

### ORM misuse
- [ ] Mass assignment: `Model.create(req.body)` uses allowlist, not full request body
- [ ] `.save()` called after explicit field assignment, not on full user-supplied object
- [ ] Eager loading doesn't expose related models user shouldn't see

## Input validation checklist
- [ ] All incoming data validated against a schema (Zod, Joi, Yup, Pydantic, FluentValidation)
- [ ] Type coercion happens after validation, not before
- [ ] File uploads: MIME type validated server-side (not just extension), size limited
- [ ] File paths: user input never directly used in `fs.readFile`, `open()`, `File()` — path traversal risk

## Data exposure checklist
- [ ] API responses return only needed fields — not full model objects
- [ ] Sensitive fields (`password`, `hash`, `secret`, `token`) excluded from serialization
- [ ] Error messages don't reveal internal structure (table names, file paths, stack traces)
- [ ] Pagination enforced — no unbounded queries on large tables

## IDOR checklist
- [ ] Every resource fetch verifies ownership: `WHERE id = ? AND user_id = currentUser.id`
- [ ] IDs not guessable/sequential where ownership matters — use UUIDs
- [ ] Bulk operations check ownership on every item, not just the first

---

# API Surface Agent — Reference

## Endpoint protection checklist
- [ ] Every non-public endpoint has authentication middleware applied
- [ ] Middleware applied at router level, not just on individual routes (prevents accidental exposure)
- [ ] New routes default to protected — opt-in to public, not opt-out

## Authorization checklist (authz ≠ authn)
- [ ] Being logged in ≠ being authorized — resource-level checks on every endpoint
- [ ] Admin/privileged endpoints have explicit role check, separate from auth check
- [ ] Multi-tenant: tenant ID validated on every cross-tenant operation

## CORS checklist
- [ ] `Access-Control-Allow-Origin` not set to `*` on credentialed endpoints
- [ ] Allowed origins are an explicit allowlist — not regex that can be bypassed
- [ ] `Access-Control-Allow-Methods` restricted to methods actually used

## Rate limiting checklist
- [ ] Auth endpoints (login, register, reset password): strict rate limit
- [ ] Resource-intensive endpoints: rate limited per user
- [ ] Public endpoints: rate limited per IP
- [ ] Rate limit errors return `429` with `Retry-After` header

## HTTP method checklist
- [ ] Routes only accept the HTTP methods they need
- [ ] `OPTIONS` handled correctly for CORS preflight
- [ ] No sensitive operations on `GET` endpoints (GET should be idempotent and safe)

## GraphQL-specific (if applicable)
- [ ] Introspection disabled in production
- [ ] Query depth limiting in place
- [ ] Query complexity limiting in place
- [ ] Mutations require authentication
- [ ] Field-level authorization where needed

## Webhook-specific (if applicable)
- [ ] Incoming webhooks verified with HMAC signature
- [ ] Replay attack protection: timestamp checked, nonce tracked
- [ ] Webhook payload size limited
