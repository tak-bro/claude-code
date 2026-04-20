---
name: verify
description: "Verify code changes by running, testing, and visually checking. Auto-fix failures. Triggers on 'verify', '검증', 'check changes', '변경 확인', 'does it work', '동작 확인', 'test this'."
tools: Read, Bash, Glob, Grep, Write, Edit, Agent, mcp__playwright
model: sonnet

# Verify

Verify code changes actually work — run the app, test from multiple angles, auto-fix failures. Uses Playwright MCP for browser verification and Claude Chrome extension for visual checks.

---

## Workflow

### Phase 1: Identify What Changed

```bash
# What files changed in this session?
git diff --name-only HEAD
git diff --cached --name-only

# Categorize changes
# → UI components? → needs visual verification
# → API endpoints? → needs request testing
# → Business logic? → needs unit test verification
# → Config/infra? → needs build verification
```

### Phase 2: Static Verification

```bash
# Lint
{pm} run lint 2>&1 | tail -20

# Type check
{pm} run typecheck 2>&1 || npx tsc --noEmit 2>&1 | tail -20

# Build
{pm} run build 2>&1 | tail -20
```

**If any fail → attempt auto-fix → re-run. Max 3 attempts.**

### Phase 3: Test Verification

```bash
# Run tests related to changed files
{pm} test -- --findRelatedTests $(git diff --name-only HEAD | tr '\n' ' ')

# If no related tests, run full suite
{pm} test 2>&1 | tail -30
```

**If tests fail → read failure output → fix → re-run. Max 3 attempts.**

### Phase 4: Runtime Verification (if applicable)

For UI changes, use Playwright MCP:

```
1. Start dev server: {pm} run dev (or ng serve, etc.)
2. Navigate to affected routes
3. Take screenshots of key states
4. Verify visual correctness using Claude Chrome extension
5. Test user interactions (click, input, navigation)
6. Check console for errors
```

For API changes:
```bash
# Start server and test endpoints
curl -s http://localhost:{port}/api/{endpoint} | jq .
```

### Phase 5: Edge Case Verification

- Empty state handling
- Error state handling
- Loading state handling
- Boundary values
- Permission/auth scenarios (if relevant)

### Phase 6: Auto-Fix Loop

When a failure is detected:
1. Read the error output
2. Identify root cause
3. Apply fix
4. Re-run the failing check
5. Repeat until pass or max 3 attempts exhausted
6. If 3 attempts fail, report to user with all attempted fixes

---

## CLI Tools Available

Reference in `_shared/verification-commands.md` for framework-specific commands.

Additional tools:
- `npx playwright test` — Playwright test runner
- `npx playwright show-report` — View test report
- Exit codes: 0 = pass, non-zero = fail

---

## Safety Rules
- **NEVER skip static verification (lint/type/build)**
- **NEVER report PASS if any check failed**
- **NEVER modify test expectations to make tests pass** (unless the test is genuinely wrong)
- Auto-fix only touches source code, never test fixtures unless explicitly told

---

## Output

```markdown
### ✅ Verify: [Scope]

**Changes:** [N files, +X/-Y lines]

| Check | Status | Details |
|-------|--------|---------|
| Lint | ✅ PASS | 0 warnings |
| Types | ✅ PASS | — |
| Build | ✅ PASS | — |
| Tests | ✅ PASS | 42 pass, 0 fail |
| Visual | ✅ PASS | Screenshots verified |
| Console | ✅ PASS | No errors |

**Auto-fixes applied:** [N] (list if any)

**Verdict:** [ALL PASS ✅ / NEEDS ATTENTION ⚠️]
```

→ If ALL PASS, go to `/ship`. If NEEDS ATTENTION, describe remaining issues.
