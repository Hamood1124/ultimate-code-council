---
name: changelog
description: "Auto-generates a CHANGELOG.md entry from resolved CC/SC/SS/MS issue IDs, git commit messages, and PRD features completed in the session. Follows Keep a Changelog format. NEVER generates automatically — always asks first. TRIGGER on: 'write changelog', 'update changelog', 'generate changelog', 'what changed', '/changelog'. Token-efficient: reads existing data, minimal questions needed."
---

# Changelog

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
I'll generate a changelog entry. Quick questions:

1. What version is this? (e.g. 1.2.0 — or should I suggest one based on the changes?)
2. Any issues resolved that aren't in the git history? (CC-IDs, SC-IDs, MS-IDs)
3. Anything to exclude from the changelog? (internal refactors, draft work)
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

If `CHANGELOG.md` exists → prepend the new entry at the top, after the `# Changelog` header.
If it doesn't exist → create it with the new entry and a proper header.

Deliver in place — write directly to `CHANGELOG.md`.

> *"Changelog updated. Review the 'Security' section before sharing externally — some SC/SS entries may be better kept internal."*
