---
name: qa
description: "QA testing. Impact analysis based on diff → testing. Triggers on 'qa', 'QA', 'quality assurance', 'quality assurance', 'integration test', 'integration test', 'regression test'."
tools: Read, Bash, Glob, Grep, Agent
---

# QA

Quality verification after implementation completion. Assess the scope of impact based on diff and test.

---

## Workflow

### Phase 1: Diff Analysis
```bash
# Identify changed files
git diff --name-only HEAD~1
git diff --stat HEAD~1
```

### Phase 2: Impact Analysis
- Track imports/exports of changed files
- Identify affected routes
- Identify related test files

### Phase 3: Test Execution
(see `_shared/verification-commands.md`)
- Run affected tests first, then full suite if possible

### Phase 4: Regression Check
- Verify no impact on existing functionality
- Execute full test suite (if possible)

### Phase 5: Browser Test (Optional)
- Coordinate with `/browse` skill for UI verification
- Test key user flows

---

## Output

```markdown
### [PASS] QA Report: [Feature]

**Changes:** [N files, +X/-Y lines]

**Impact Analysis:**
- Routes affected: [list]
- Components affected: [list]

**Test Results:**
- Unit: [PASS] [N] pass / [FAIL] [N] fail
- Lint: [PASS] pass / [FAIL] [issues]
- Build: [PASS] pass / [FAIL] [errors]

**Regression:** [No issues / Issues found]

**Verdict:** [PASS / FAIL]
```

→ If PASS, go to `/ship`, if FAIL, go to `/{framework}-fix`
