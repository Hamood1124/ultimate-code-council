# Security Baseline — [Project Name]

> This file is read by the Code Council, Security Council, Secrets Scan, and Microsoft Security skills.
> Keep it updated after every security review. Do not commit real secrets here.

---

## Project classification

**Data classification:** [Internal only / Client-facing / Public internet]
**Compliance requirements:** [None / GDPR / PCI-DSS / ISO 27001 / other]
**Last security review:** [Date — updated automatically by security-council]

---

## Authentication

**Auth provider:** [Azure AD / Auth0 / JWT (custom) / Session-based / Other]
**Token format:** [JWT / Opaque / Session cookie]
**Token storage:** [httpOnly cookie / In-memory (MSAL) / Other]
**Session timeout:** [idle: Xmin / absolute: Xhr]
**MFA required:** [Yes / No / Admin only]

---

## Authorization model

**Permission model:** [RBAC / ABAC / ACL / Other]
**Roles defined:** [List roles — e.g. Admin, Manager, Employee, ReadOnly]
**Multi-tenant:** [Yes / No]
**Resource ownership model:** [Describe how user-to-resource ownership is enforced]

---

## Secret storage (approved approaches)

> Do NOT put actual secrets here. Document the approved pattern only.

| Environment | Approved storage |
|-------------|-----------------|
| Development | `.env` file (gitignored) |
| Staging | [Azure Key Vault / Environment variables / Other] |
| Production | [Azure Key Vault / Managed Identity / Other] |

**Key Vault URL (if applicable):** `https://[vault-name].vault.azure.net/`
**Managed identity used:** [Yes / No]

---

## Intentionally public endpoints

> List endpoints that are intentionally unauthenticated. Security Council will not flag these.

- `GET /api/health` — health check, no auth required
- `POST /api/webhook/[provider]` — verified by HMAC signature, no user auth
- [add more as needed]

---

## Known dependency exceptions

> CVEs accepted with justification. Security Council will downgrade these to Low advisory.

| Package | CVE | Accepted because | Review date |
|---------|-----|-----------------|-------------|
| [package] | CVE-XXXX-XXXXX | [justification] | [date] |

---

## Microsoft stack (if applicable)

**Azure AD Tenant ID:** [tenant ID — not a secret, safe to store here]
**App Registration Client ID:** [client ID — not a secret, safe to store here]
**Graph API scopes in use:**
- `User.Read` — read signed-in user profile
- [list all scopes actually used]

**SharePoint permission scopes (package-solution.json):**
- [list all webApiPermissionRequests]

---

## Accepted risks

> Risks reviewed and accepted. Security scans will downgrade these findings.

| Risk | Accepted because | Review date | Accepted by |
|------|-----------------|-------------|-------------|
| [description] | [justification] | [date] | [name] |

---

## Review log

> Updated automatically by /security-council after each review.

| Date | Depth | Findings | Verdict | Reviewer |
|------|-------|----------|---------|----------|
| [date] | Standard | 0 Blockers, 1 High, 2 Medium | SECURE | Claude Code Security Council |
