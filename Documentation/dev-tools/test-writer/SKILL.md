---
name: test-writer
description: "Writes tests for existing untested code. Reads functions and writes tests that cover real behavior — not imagined behavior. Runs the Correctness Judge first to understand what the code actually does before writing tests. TRIGGER on: 'write tests for this', 'add tests', 'no tests exist', 'test this function', 'test coverage', '/test-writer'. Works on any language. Never auto-generates — always asks scope first."
---

# Test Writer

Writes tests for existing code that has none. Different from `/tdd` (which writes tests before code) — this works backward from existing code to build a test suite around it.

---

## Phase 0 — Load context (silently)

```bash
cat CONTEXT.md 2>/dev/null

# Detect test framework
cat package.json 2>/dev/null | grep -E '"jest"|"vitest"|"mocha"|"jasmine"|"@testing-library"'
find . -name "pytest.ini" -o -name "conftest.py" -o -name "*.test.ts" \
       -o -name "*.spec.ts" 2>/dev/null | head -10

# Check existing test coverage
npx jest --coverage --passWithNoTests 2>/dev/null | tail -20
pytest --co -q 2>/dev/null | tail -20

# Find untested files
find . -not -path '*/node_modules/*' -not -path '*/__tests__/*' \
       -not -path '*/test*' -not -name "*.test.*" -not -name "*.spec.*" \
       \( -name "*.ts" -o -name "*.tsx" -o -name "*.js" -o -name "*.py" \
          -o -name "*.cs" \) | head -30
```

---

## Phase 1 — Scope the request

```
What do you want tests for?

A) Specific file(s) — tell me which
B) All untested files — I found [N] files with no tests
C) Specific function or class — paste it or tell me where it is
D) Just the critical paths — auth, data access, business logic only

Also: are there any behaviors you know are important to test
that might not be obvious from reading the code?
```

---

## Phase 2 — Read and understand before writing

Before writing a single test, run the Correctness Judge mentally on the target code:

- What does this function/component actually do?
- What are the happy paths?
- What are the edge cases?
- What can go wrong?
- What does it depend on (and what needs to be mocked)?

Only write tests for behaviors that are actually present in the code — not behaviors that should be there. (If something should be there but isn't, flag it as a CC-level finding instead.)

---

## Phase 3 — Write tests

**Test structure rules:**
- One test file per source file: `LeaveService.ts` → `LeaveService.test.ts`
- Group by behavior, not by function name: `describe('when leave balance is zero')` not `describe('getLeaveBalance')`
- Arrange / Act / Assert structure on every test
- Test names read as sentences: `it('returns zero when employee has no remaining leave')`
- Mock external dependencies (API calls, DB, D365) — test the logic, not the integration

**Per-language format:**

TypeScript/Jest:
```typescript
describe('LeaveService', () => {
  describe('when calculating leave balance', () => {
    it('returns remaining balance for the requested leave type', () => {
      // Arrange
      const service = new LeaveService(mockD365Client);
      // Act
      const balance = service.getBalance('EMP001', 'Annual');
      // Assert
      expect(balance).toBe(10);
    });

    it('returns zero when employee has exhausted leave', () => { ... });
    it('throws when employee ID does not exist', () => { ... });
  });
});
```

Python/pytest:
```python
class TestLeaveService:
    def test_returns_remaining_balance_for_requested_type(self, mock_d365):
        # Arrange / Act / Assert pattern
```

---

## Phase 4 — Coverage report

After writing tests, run them:

```bash
npx jest --coverage 2>/dev/null | tail -30
pytest --cov 2>/dev/null | tail -20
```

Report:
```
Tests written: [N]
Coverage added: [X]% → [Y]%
Uncovered paths remaining: [list]
```

> *"Tests written. Run them now to confirm they all pass. If any fail, it means the code doesn't do what I thought it did — those failures are finding real bugs."*
