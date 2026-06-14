---
name: msft-security
description: "Microsoft-stack specific security review. Covers SharePoint/SPFx permission leakage, Graph API scope over-permission, Azure AD app registration hygiene, D365/OData endpoint exposure, Azure Key Vault usage, and SPFx web part property security. Runs on top of the main security-council as a bonus layer — only activates when Microsoft stack is detected or explicitly called. TRIGGER on: 'check Microsoft security', 'SPFx security', 'Graph API security', 'Azure security', 'D365 security', 'check permissions', 'msft security'. Auto-triggered by council-start when Microsoft stack detected and security-relevant files changed. Issue IDs: MS-001, MS-002..."
---

# Microsoft Security Council

Specialist security review for Microsoft stack projects. Runs after or alongside `/security-council` — it covers attack surfaces unique to the Microsoft ecosystem that generic security tools miss entirely.

Issue IDs: `MS-001`, `MS-002`... (separate from SC- and SS- series)

Only runs when Microsoft stack is detected:
- SPFx / SharePoint Online
- D365 F&O / Business Central
- Azure Functions / App Service
- Power Automate / Power Apps
- Microsoft Graph API usage
- Azure AD authentication

---

## Phase 0 — Stack detection and context load

```bash
# Detect Microsoft stack
cat package.json 2>/dev/null | grep -i "spfx\|@microsoft\|@pnp\|sharepoint\|graph"
find . -name "*.csproj" -o -name "config/package-solution.json" \
       -o -name "config/deploy-azure-storage.json" 2>/dev/null | head -10
cat SECURITY.md 2>/dev/null
cat CONTEXT.md 2>/dev/null

# SPFx specific
find . -name "package-solution.json" -o -name "*.manifest.json" \
       -o -name "serve.json" 2>/dev/null
cat config/package-solution.json 2>/dev/null

# D365 specific
find . -name "*.xpp" -o -name "*.xml" -path "*/AOT/*" 2>/dev/null | head -20

# Graph API usage
grep -rn "graph.microsoft.com\|@microsoft/microsoft-graph-client\|GraphServiceClient" \
  --include="*.ts" --include="*.js" --include="*.cs" \
  -l 2>/dev/null | head -20

# Azure AD / MSAL
grep -rn "MSAL\|PublicClientApplication\|ConfidentialClientApplication\|@azure/msal" \
  --include="*.ts" --include="*.js" --include="*.cs" \
  -l 2>/dev/null | head -20
```

If no Microsoft stack detected → tell the user and exit gracefully:
> *No Microsoft stack detected in this project. `/msft-security` is designed for SPFx, D365, Graph API, and Azure projects. Run `/security-council` for a general security review.*

---

## Phase 1 — Six specialist checks

### Check 1 — SPFx & SharePoint security

**What to look for:**

**Web part properties exposure:**
- Sensitive values (client IDs, endpoint URLs, tenant IDs) stored in web part properties are visible to any SharePoint admin and can appear in page source
- Flag any property that contains a secret, key, or credential
- Correct approach: Azure Key Vault reference or server-side config

**Permission scope (package-solution.json):**
```bash
cat config/package-solution.json 2>/dev/null | grep -A 5 "webApiPermissionRequests\|permissions"
```
- Flag `Sites.FullControl.All` — almost never needed, use `Sites.ReadWrite.All` or narrower
- Flag `Mail.ReadWrite` when `Mail.Read` would suffice
- Flag `Directory.ReadWrite.All` — extremely broad, flag as 🔴 Blocker unless explicitly justified in SECURITY.md
- Flag any `*.All` permission — check if the narrower version would work
- Flag `User.ReadWrite.All` — can modify any user in the tenant

**SharePoint REST API calls:**
- Calls using `/_api/web/siteusers` or `/_api/site/owner` may expose user PII
- `/_api/search/query` with broad result sources may return content from sites user shouldn't access
- Verify search results are scoped to the user's accessible content

**Issue format:** `MS-001 [SPFx]`

---

### Check 2 — Microsoft Graph API security

**What to look for:**

```bash
# Find all Graph API calls
grep -rn "graph.microsoft.com\|/v1.0/\|/beta/" \
  --include="*.ts" --include="*.js" --include="*.cs" 2>/dev/null
```

**Scope over-permission:**
- Map every Graph API endpoint called to the minimum required permission scope
- Flag where broader scopes are used than necessary

Common over-permission patterns:
| What code does | Minimum scope needed | Often requested instead |
|---------------|---------------------|------------------------|
| Read user profile | `User.Read` | `User.ReadWrite` or `User.ReadWrite.All` |
| Read team messages | `ChannelMessage.Read.All` | `Group.ReadWrite.All` |
| Send email | `Mail.Send` | `Mail.ReadWrite` |
| Read calendar | `Calendars.Read` | `Calendars.ReadWrite` |
| Read files | `Files.Read.All` | `Files.ReadWrite.All` |

**Token handling:**
- Access tokens stored in `localStorage` → 🔴 Blocker (XSS accessible)
- Access tokens stored in `sessionStorage` → 🟠 High
- Tokens logged to console → 🔴 Blocker
- Tokens passed in URL parameters → 🔴 Blocker (appear in logs)
- Correct storage: in-memory (MSAL handles this) or `httpOnly` cookie

**Beta endpoint usage:**
- `/beta/` endpoints are not supported for production use by Microsoft
- Flag any `/beta/` calls with 🟡 Medium — suggest `/v1.0/` equivalent

**Issue format:** `MS-00N [Graph]`

---

### Check 3 — Azure AD app registration hygiene

```bash
# Find app registration config
grep -rn "clientId\|tenantId\|clientSecret\|authority" \
  --include="*.ts" --include="*.js" --include="*.json" --include="*.cs" \
  --include="*.env*" 2>/dev/null | grep -v node_modules | grep -v ".git"
```

**What to look for:**

**Client secret in code:**
- `clientSecret` hardcoded anywhere → 🔴 Blocker
- Correct approach: Azure Key Vault, managed identity, or environment variable (never committed)

**Public client vs confidential client:**
- SPFx web parts are public clients — they cannot safely hold a client secret
- If SPFx code contains a `clientSecret`, it's always a 🔴 Blocker — secrets in browser bundles are public
- Backend services should use confidential client with managed identity where possible

**Redirect URIs:**
- Wildcard redirect URIs (`https://*.sharepoint.com/*`) are overly permissive
- Localhost redirect URIs left in production app registration → 🟠 High
- Note: Claude Code cannot check the actual Azure portal — flag for manual review

**Multi-tenant apps:**
- If app is multi-tenant (`signInAudience: "AzureADMultipleOrgs"`), verify token issuer validation is tenant-specific
- Missing tenant ID validation in multi-tenant app → 🔴 Blocker

**Issue format:** `MS-00N [AzureAD]`

---

### Check 4 — D365 F&O / Business Central security

**What to look for:**

**OData endpoint exposure:**
```bash
grep -rn "\/data\/\|ODataV4\|\/api\/data\|ODataServiceClient" \
  --include="*.ts" --include="*.js" --include="*.cs" --include="*.xpp" 2>/dev/null
```

- OData `$filter` built with string concatenation → 🔴 Blocker (injection)
- OData `$select` not used — returning full entities when partial data needed → 🟠 High (over-exposure)
- Cross-company data access without explicit company filter → 🟠 High
- Service accounts with System Administrator role used in integrations → 🟠 High

**X++ security (D365 F&O):**
- `RunBase` classes missing permission checks
- Direct SQL via `Statement`/`ResultSet` instead of X++ select → 🔴 Blocker
- `SysEntryPointAttribute(false)` on public methods that should be protected → 🟠 High
- Sensitive data in `Info` logs visible to all users

**Business Central AL security:**
- `InherentPermissions` not set on codeunits accessing sensitive tables
- `DataClassification` not set on sensitive fields → 🟡 Medium
- External web service calls without certificate validation

**Issue format:** `MS-00N [D365]` or `MS-00N [BC]`

---

### Check 5 — Azure Key Vault and secret management

```bash
# Check for Key Vault usage
grep -rn "KeyVaultClient\|SecretClient\|@azure/keyvault\|KeyVault" \
  --include="*.ts" --include="*.js" --include="*.cs" 2>/dev/null | head -20

# Check for managed identity
grep -rn "ManagedIdentityCredential\|DefaultAzureCredential\|ChainedTokenCredential" \
  --include="*.ts" --include="*.js" --include="*.cs" 2>/dev/null | head -20

# Check app settings / config files for secrets
find . -name "appsettings*.json" -o -name "local.settings.json" \
       -o -name "appsettings.Development.json" 2>/dev/null | xargs grep -l "secret\|key\|password\|connectionstring" 2>/dev/null
```

**What to look for:**

- `local.settings.json` committed to repo → 🔴 Blocker (Azure Functions local config)
- `appsettings.Development.json` with real secrets committed → 🔴 Blocker
- Key Vault used but with client secret instead of managed identity → 🟠 High (managed identity is preferred)
- Key Vault secret version hardcoded (won't rotate automatically) → 🟡 Medium
- Azure Storage connection strings in config instead of managed identity → 🟠 High
- Service bus connection strings in config instead of managed identity → 🟠 High

**Correct pattern (prefer managed identity):**
```csharp
// Good — managed identity, no secrets in code
var client = new SecretClient(
    new Uri("https://myvault.vault.azure.net/"),
    new DefaultAzureCredential()
);

// Bad — client secret in code
var client = new SecretClient(..., new ClientSecretCredential(tenantId, clientId, clientSecret));
```

**Issue format:** `MS-00N [KeyVault]`

---

### Check 6 — General Azure security hygiene

```bash
# Check for Azure connection strings
grep -rn "DefaultEndpointsProtocol\|AccountKey\|SharedAccessSignature" \
  --include="*.ts" --include="*.js" --include="*.cs" --include="*.json" \
  --include="*.config" 2>/dev/null | grep -v node_modules | grep -v ".git"

# Check CORS config in Azure Functions
grep -rn "cors\|CORS\|AllowedOrigins" \
  --include="*.json" --include="*.cs" 2>/dev/null | head -20
```

**What to look for:**

- Azure Storage `AccountKey` in code or config → 🔴 Blocker (use SAS tokens or managed identity)
- `SharedAccessSignature` tokens hardcoded → 🔴 Blocker
- Azure Functions CORS set to `*` → 🟠 High
- `host.json` with `authLevel: "anonymous"` on functions that should be protected → 🔴 Blocker
- Application Insights instrumentation key in client-side code → 🟡 Medium (use connection string with managed identity)

**Issue format:** `MS-00N [Azure]`

---

## Phase 2 — Microsoft security verdict

```markdown
## 🏢 Microsoft Security Verdict

**Stack detected:** [SPFx / D365 / Graph API / Azure Functions / etc.]
**Verdict:** [SECURE / HARDEN BEFORE SHIP / DO NOT SHIP]

| Area | Status | Issues |
|------|--------|--------|
| SPFx & SharePoint | ✅ / ⚠️ / 🔴 | MS-XXX |
| Graph API | ✅ / ⚠️ / 🔴 | MS-XXX |
| Azure AD | ✅ / ⚠️ / 🔴 | MS-XXX |
| D365 / BC | ✅ / ⚠️ / 🔴 | MS-XXX |
| Key Vault | ✅ / ⚠️ / 🔴 | MS-XXX |
| Azure config | ✅ / ⚠️ / 🔴 | MS-XXX |

| Severity | Count | MS-IDs |
|----------|-------|--------|
| 🔴 Blocker | N | MS-001 |
| 🟠 High | N | MS-002 |
| 🟡 Medium | N | MS-003 |
| 🟢 Low | N | |

### The one thing to fix first
MS-XXX — [highest priority finding]

### What this review did not cover
- Azure portal configuration (app registrations, RBAC assignments) — requires manual review
- Conditional Access policies — requires Azure AD admin access
- SharePoint site permissions — requires SharePoint admin access
- Power Platform connectors — Phase 3
```
