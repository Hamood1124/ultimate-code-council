---
name: changelog
description: "Auto-generates a CHANGELOG.md entry from resolved CC/SC/SS/MS issue IDs, git commit messages, and PRD features completed in the session. Follows Keep a Changelog format. NEVER generates automatically — always asks first. TRIGGER on: 'write changelog', 'update changelog', 'generate changelog', 'what changed', '/changelog'. Token-efficient: reads existing data, minimal questions needed."
---

# Changelog

---

## Phase 0 — Tooling check (silently, before anything else)

```bash
# Check if Node.js is available
node --version 2>/dev/null || echo "NODE_NOT_FOUND"

# Check if docx library is installed globally
node -e "require('docx')" 2>/dev/null || echo "DOCX_NOT_INSTALLED"
```

If `NODE_NOT_FOUND`:
> *"Node.js is required to generate Word documents. Install it from nodejs.org, then try again. In the meantime, I can deliver this as Markdown."*
→ Default to Markdown, continue.

If `DOCX_NOT_INSTALLED`:
> *"Installing the docx library — this only happens once."*
```bash
npm install -g docx
```
Then confirm installation succeeded before continuing. If install fails, default to Markdown and note it.

> Note: PDF output is not supported in Claude Code. For PDFs, use the web version at claude.ai instead.

Generates a clean `CHANGELOG.md` entry from the work done in the current session. Reads issue IDs, git history, and PRD to build an accurate, versioned record.

---

## Phase 0 — Load context (silently)

```bash
# Get resolved issues from git
git log --oneline --since="1 day ago" 2>/dev/null
git diff --name-only HEAD~1 HEAD 2>/dev/null

# Check if CHANGELOG.md already exists
cat CHANGELOG.md 2>/dev/null | head -30

# Get current version
cat package.json 2>/dev/null | grep '"version"'
cat *.csproj 2>/dev/null | grep "<Version>"
```

---

## Phase 1 — Ask only what's missing

```
Before I generate the changelog, quick questions:

1. What format do you want?
   - Markdown (.md) — written directly to CHANGELOG.md
   - Word document (.docx) — downloadable

2. What version is this? (e.g. 1.2.0 — or should I suggest one based on the changes?)
3. Any issues resolved that aren't in the git history? (CC-IDs, SC-IDs, MS-IDs)
4. Anything to exclude from the changelog? (internal refactors, draft work)
```

**Version suggestion logic:**
- Only bug fixes (CC/SC Blockers/Highs fixed) → patch bump (1.0.0 → 1.0.1)
- New features added → minor bump (1.0.0 → 1.1.0)
- Breaking changes → major bump (1.0.0 → 2.0.0)
- Ask user to confirm before using suggested version

---

## Phase 2 — Generate

Follow [Keep a Changelog](https://keepachangelog.com) format strictly:

```markdown
## [1.2.0] — 2025-06-15

### Added
- Leave request approval workflow with multi-level routing (PRD: US-001, US-002)
- OData integration with D365 F&O for real-time leave balances

### Fixed
- CC-001 — Null reference on Customer entity when leave balance is zero
- CC-003 — Async race condition in approval notification handler
- SC-001 — OData filter injection via string concatenation (Security)
- SS-001 — API key removed from source, moved to Azure Key Vault

### Security
- SC-002 — JWT issuer validation added to auth middleware
- MS-001 — Graph API scope reduced from Files.ReadWrite.All to Files.Read

### Changed
- Leave request form now shows balance per type before submission
- Approval emails now include leave dates and remaining balance

### Removed
- Deprecated LeaveRequestV1 endpoint removed (replaced by V2 in v1.1.0)
```

---

## Phase 3 — Deliver

Deliver in the format confirmed in Phase 1:
- Markdown → if `CHANGELOG.md` exists, prepend the new entry at the top. If not, create it. Write directly to file.
- Word → generate `.docx` file

> *"Changelog updated. Review the 'Security' section before sharing externally — some SC/SS entries may be better kept internal."*
