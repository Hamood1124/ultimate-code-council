---
name: env-setup
description: "Validates that the local environment matches what the project needs. Reads package.json, .env.example, requirements.txt, Dockerfile etc. and reports exactly what's missing, what versions are wrong, and what to run to fix it. TRIGGER on: 'check my environment', 'env setup', 'something is missing', 'works on my machine', 'set up environment', '/env-setup'. Works on any stack."
---

# Env Setup

Checks your local environment against what the project needs. Tells you exactly what's missing, what versions are wrong, and gives you the exact commands to fix it.

---

## Phase 0 — Read project requirements (silently)

```bash
# What does the project need?
cat package.json 2>/dev/null | grep -E '"node"|"engines"|"packageManager"'
cat .nvmrc .node-version 2>/dev/null
cat .env.example .env.sample 2>/dev/null
cat requirements.txt pyproject.toml 2>/dev/null
cat *.csproj 2>/dev/null | grep "TargetFramework"
cat go.mod 2>/dev/null | head -5
cat Cargo.toml 2>/dev/null | grep "edition\|rust-version"
cat Dockerfile 2>/dev/null | head -20
cat docker-compose.yml 2>/dev/null | head -30
cat README.md 2>/dev/null | grep -A 20 "Prerequisites\|Requirements\|Setup\|Install"
```

---

## Phase 1 — Check local environment

```bash
# Runtime versions
node --version 2>/dev/null
npm --version 2>/dev/null
python --version 2>/dev/null || python3 --version 2>/dev/null
dotnet --version 2>/dev/null
go version 2>/dev/null
rustc --version 2>/dev/null
docker --version 2>/dev/null
git --version 2>/dev/null

# Check .env file exists and has required keys
ls .env 2>/dev/null || echo "NO_ENV_FILE"
if [ -f .env.example ] && [ -f .env ]; then
  # Find keys in .env.example that are missing from .env
  diff <(grep -oP '^[^=]+' .env.example | sort) \
       <(grep -oP '^[^=]+' .env | sort) 2>/dev/null
fi

# Check dependencies installed
ls node_modules 2>/dev/null || echo "NO_NODE_MODULES"
ls venv .venv 2>/dev/null || echo "NO_VENV"

# Check ports if docker-compose exists
cat docker-compose.yml 2>/dev/null | grep "ports:"
```

---

## Phase 2 — Report

```markdown
## Environment Check — [Project Name]

### ✅ OK
- Node.js: v20.11.0 (required: >=18.0.0) ✅
- npm: 10.2.4 ✅
- Git: 2.43.0 ✅

### ❌ Issues Found

#### Missing: .env file
Your project needs a `.env` file. A `.env.example` exists as a template.
**Fix:**
```bash
cp .env.example .env
```
Then fill in the values — see SECURITY.md for where each secret lives.

#### Missing env variables (in .env.example but not in .env)
- `D365_CLIENT_SECRET` — Azure Key Vault: [vault-name]/secrets/d365-client-secret
- `GRAPH_API_TENANT_ID` — Ask Ashraf for the tenant ID

#### Wrong version: Python 3.9.0 (required: >=3.11)
**Fix:**
```bash
# Using pyenv:
pyenv install 3.11.0
pyenv local 3.11.0
```

#### Dependencies not installed
**Fix:**
```bash
npm install
```

#### Docker not running
Docker Desktop appears to be installed but not running.
**Fix:** Start Docker Desktop, then:
```bash
docker-compose up -d
```

### ⚠️ Warnings
- npm version 8.x detected — project was developed with npm 10.x.
  May work but consider upgrading: `npm install -g npm@latest`

### 🔧 Run this to fix everything in order:
```bash
cp .env.example .env           # 1. Create env file
npm install                    # 2. Install dependencies
# Fill in .env values          # 3. Manual step — see SECURITY.md
docker-compose up -d           # 4. Start services (if applicable)
npm run dev                    # 5. Start the app
```
```

---

## Phase 3 — Verify after fixes

If the user says they've applied the fixes:

```bash
# Re-run the checks
node --version && npm --version
ls node_modules | wc -l
cat .env | grep -v "=" | head  # Check for empty values
npm run build 2>/dev/null | tail -10
```

Report updated status.

> *"Environment looks good. If you still have issues, the most common cause is missing or incorrect values in your .env file — double-check each value against SECURITY.md."*
