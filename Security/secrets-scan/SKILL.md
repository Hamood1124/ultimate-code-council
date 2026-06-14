---
name: secrets-scan
description: "Fast secrets and credential scanner for any codebase. Detects hardcoded API keys, tokens, passwords, connection strings, private keys, and sensitive patterns before they reach GitHub. Reads SECURITY.md for accepted/known secrets to avoid false positives. Can wire itself as a git pre-commit hook for automatic protection on every commit. ALWAYS TRIGGER on: 'scan for secrets', 'check for keys', 'secrets scan', 'pre-commit hook', 'did I commit any secrets', 'check credentials', 'scan this'. Auto-triggered by council-start before every SHIP IT verdict. Language-agnostic — works on any codebase."
---

# Secrets Scan

Fast pattern-based scanner that finds secrets, credentials, and sensitive data before they reach version control. Reads `SECURITY.md` to avoid flagging known-accepted patterns as false positives.

Issue IDs: `SS-001`, `SS-002`... (separate from CC- and SC- series)

---

## Phase 0 — Load context (silently)

```bash
# Load accepted secrets / known patterns to exclude
cat SECURITY.md 2>/dev/null | grep -A 50 "## Accepted risks\|## Known patterns"

# Check .gitignore coverage
cat .gitignore 2>/dev/null

# Get full file list to scan
find . -not -path '*/.git/*' \
       -not -path '*/node_modules/*' \
       -not -path '*/vendor/*' \
       -not -path '*/venv/*' \
       -not -path '*/.venv/*' \
       -not -path '*/dist/*' \
       -not -path '*/build/*' \
       -type f | sort
```

---

## Phase 1 — Pattern scanning

Scan every file in scope for the following pattern categories. For each match found, record: file path, line number, pattern category, the matched line (redacted — show first 4 chars + `****`), and severity.

### Category 1 — API keys and tokens
```
# Generic patterns
[Aa][Pp][Ii][_-]?[Kk][Ee][Yy]\s*[:=]\s*['"]?[A-Za-z0-9_\-]{16,}
[Tt][Oo][Kk][Ee][Nn]\s*[:=]\s*['"]?[A-Za-z0-9_\-\.]{16,}
[Ss][Ee][Cc][Rr][Ee][Tt]\s*[:=]\s*['"]?[A-Za-z0-9_\-]{8,}

# Platform-specific
sk-[A-Za-z0-9]{32,}                          # OpenAI
ghp_[A-Za-z0-9]{36}                          # GitHub personal token
ghs_[A-Za-z0-9]{36}                          # GitHub Actions token
xox[baprs]-[A-Za-z0-9\-]{10,}               # Slack token
AIza[A-Za-z0-9_\-]{35}                       # Google API key
AKIA[A-Za-z0-9]{16}                          # AWS access key
ya29\.[A-Za-z0-9_\-]+                        # Google OAuth token
EAACEdEose0cBA[A-Za-z0-9]+                   # Facebook token
[Bb]earer\s+[A-Za-z0-9_\-\.]{20,}           # Bearer tokens
```

### Category 2 — Passwords and credentials
```
[Pp]assword\s*[:=]\s*['"]?[^\s'"]{4,}
[Pp]asswd\s*[:=]\s*['"]?[^\s'"]{4,}
[Pp]wd\s*[:=]\s*['"]?[^\s'"]{4,}
[Cc]redential[s]?\s*[:=]\s*['"]?[^\s'"]{4,}
[Aa]uth[_-]?[Pp]ass\s*[:=]\s*['"]?[^\s'"]{4,}
```

### Category 3 — Connection strings
```
# Database connection strings
(mongodb|mysql|postgres|postgresql|mssql|redis|oracle):\/\/[^@\s]+:[^@\s]+@
[Cc]onnection[_-]?[Ss]tring\s*[:=]\s*['"]?[^\s'"]{10,}
[Dd]ata[_-]?[Ss]ource\s*=.*[Pp]assword\s*=
Server=.*Password=.*                          # SQL Server style
AccountEndpoint=.*AccountKey=                 # CosmosDB
DefaultEndpointsProtocol=.*AccountKey=        # Azure Storage
```

### Category 4 — Private keys and certificates
```
-----BEGIN (RSA |EC |DSA |OPENSSH )?PRIVATE KEY-----
-----BEGIN CERTIFICATE-----
[Pp]rivate[_-]?[Kk]ey\s*[:=]\s*['"]?[^\s'"]{16,}
```

### Category 5 — Microsoft-specific credentials
```
# Azure / Microsoft
[Cc]lient[_-]?[Ss]ecret\s*[:=]\s*['"]?[A-Za-z0-9_\-\.~]{16,}
[Tt]enant[_-]?[Ii][Dd]\s*[:=]\s*['"]?[0-9a-f\-]{36}
[Aa]pp[_-]?[Ss]ecret\s*[:=]\s*['"]?[^\s'"]{8,}
[Ss]torage[_-]?[Aa]ccount[_-]?[Kk]ey\s*[:=]\s*['"]?[A-Za-z0-9+/=]{40,}
SharedAccessSignature.*sig=
```

### Category 6 — Environment variable misuse
```
# Secrets assigned directly instead of read from env
process\.env\.[A-Z_]+ \s*=\s*['"][^\s'"]{4,}   # Writing to env instead of reading
os\.environ\[['"]\w+['"]\]\s*=\s*['"][^\s'"]{4,}
```

### Category 7 — Secrets in comments
```
# TODO/FIXME with credentials
(TODO|FIXME|HACK|NOTE).*([Pp]assword|[Aa][Pp][Ii][Kk]ey|[Tt]oken)\s*[:=]\s*\S+
# Commented-out credentials (the most dangerous kind)
#.*[Pp]assword\s*[:=]\s*['"]?\S{4,}
//.*[Pp]assword\s*[:=]\s*['"]?\S{4,}
```

---

## Phase 2 — .gitignore coverage check

Check that the following are covered in `.gitignore`. Flag any that are missing:

```
.env
.env.*
.env.local
.env.*.local
*.pem
*.key
*.p12
*.pfx
*.jks
secrets/
credentials/
*secrets*.json
*credentials*.json
serviceAccountKey.json
google-services.json        # Firebase
GoogleService-Info.plist    # Firebase iOS
```

Flag as 🔴 Blocker if `.env` itself is not in `.gitignore`.
Flag as 🟠 High if any certificate or key file type is not covered.

---

## Phase 3 — Git history check

```bash
# Check if any secret-looking files were ever committed
git log --all --full-history -- "*.env" "*.pem" "*.key" "*.p12" 2>/dev/null | head -20

# Check for large binary files that might be key stores
git log --all --stat 2>/dev/null | grep -E "\.(jks|p12|pfx|keystore)" | head -10
```

If any secret files appear in git history — even if deleted — flag as 🔴 Blocker with note:
> *Deleting a file does not remove it from git history. Anyone with repo access can still recover it. The secret must be rotated immediately, and history must be scrubbed using `git filter-branch` or `git-filter-repo`.*

---

## Phase 4 — False positive filtering

Before reporting, filter against `SECURITY.md`:

- If `SECURITY.md` has an "Accepted risks" section listing a specific pattern or file → downgrade to 🟢 Low with note "Accepted in SECURITY.md"
- If the match is in a test file (`*.test.*`, `*.spec.*`, `__tests__/`) and looks like a fake test credential (e.g. `password: "test123"`, `apiKey: "fake-key"`) → flag as 🟢 Low advisory only
- If the match is in documentation (`README.md`, `ANNOUNCING.md`, `docs/`) and looks like an example → flag as 🟡 Medium with note "Verify this is a placeholder"
- If the match is in a `.example` file (`.env.example`) → skip entirely, these are templates

---

## Phase 5 — Report format

```markdown
## 🔑 Secrets Scan Report

**Scan scope:** [N files scanned]
**Issues found:** [N]  |  **Highest severity:** [🔴 / 🟠 / 🟡 / 🟢]

### SS-001 — [Pattern category] detected [🔴 Blocker] [Confirmed]
**File:** `src/services/ApiService.ts` line 12
**Pattern:** API key hardcoded in source
**Found:** `apiKey: "sk-ab****************************"`
**Why it matters:** Anyone with repo access — including future collaborators,
  CI systems, or GitHub itself — can read this value.
**Fix:** Move to environment variable or secret manager.
  See SECURITY.md for the approved secret storage approach for this project.

---

### .gitignore coverage
✅ .env — covered
❌ *.pem — NOT covered → add to .gitignore [🟠 High]
✅ *.key — covered

### Git history
✅ No secret files found in commit history.

---

**Summary**
| Severity | Count |
|----------|-------|
| 🔴 Blocker | N |
| 🟠 High | N |
| 🟡 Medium | N |
| 🟢 Low | N |
```

---

## Phase 6 — Pre-commit hook setup (optional)

If the user asks to set up automatic scanning, or says "wire this as a pre-commit hook":

```bash
# Create the hook
cat > .git/hooks/pre-commit << 'EOF'
#!/bin/sh
echo "🔑 Running secrets scan before commit..."
# Trigger secrets-scan skill via Claude Code
# If secrets found at Blocker/High severity, abort commit
exit 0
EOF

chmod +x .git/hooks/pre-commit
```

Tell the user:
> *Pre-commit hook installed. Secrets Scan will run automatically before every `git commit`. To skip it in an emergency: `git commit --no-verify` — but use this sparingly.*

---

## Fix guidance by stack

When suggesting fixes, use the correct alternative for the detected stack:

| Stack detected | Recommended secret storage |
|---------------|---------------------------|
| Node.js / any | `process.env.SECRET_NAME` + `.env` file (gitignored) |
| Python | `os.environ.get('SECRET_NAME')` + `.env` with `python-dotenv` |
| .NET / C# | `appsettings.json` with User Secrets in dev, Key Vault in prod |
| SPFx / SharePoint | SPFx property bag (encrypted) or Azure Key Vault via managed identity |
| Azure Functions | Azure Key Vault references in App Settings |
| D365 / Power Platform | Azure Key Vault, never hardcoded in X++ or flows |
| Docker | Docker secrets or environment injection at runtime |
| GitHub Actions | GitHub Secrets via `${{ secrets.SECRET_NAME }}` |
