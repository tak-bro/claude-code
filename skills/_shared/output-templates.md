# Output Templates

## Plan Output
```
Plan complete: `.claude/{YYYYMMDD}/PLAN-{HH-MM-SS}.md`
→ Next: `/{framework}-02-implement` to start implementation.
```

## Implement Output
```markdown
### Implemented: [Feature]

**Files Created/Modified:**
- ✨ `path/to/new-file.ts`
- [Fix] `path/to/modified-file.ts`

**Checklist Completed:**
- [x] [Item 1]
- [x] [Item 2]

**Verification:** (if available)
- [x] `{pm} run lint` ✓
- [x] `{pm} test` ✓

**Notes for Review:**
- [Implementation decisions]
- [Areas needing attention]
```

## Review Output
```markdown
### [Review] Review: [Feature] - Score: X/10

**Summary:**
- [Critical]: X issues (must fix)
- [Important]: X issues (should fix)
- [Nice-to-have]: X issues (optional)

## [Critical] Issues

### C1: [Issue Title]
- **File:** `path/to/file.ts:line`
- **Problem:** [Description]
- **Fix:**
```typescript
// Before
[code]

// After
[code]
```

## Fix Checklist (→ run /{framework}-fix)

**[Critical] (Must Fix):**
- [ ] C1: [Description] (file:line)

**[Important] (Should Fix):**
- [ ] I1: [Description] (file:line)
```

## Fix Output
```markdown
### Fixed: [Feature]

**Fixes Applied:**
- [x] [Critical] C1: [Description]
- [x] [Important] I1: [Description]

**Verification:** (if available)
- [x] `{pm} run lint` ✓
- [x] `{pm} test` ✓
- [x] `{pm} run build` ✓

**Final Score:** X/10 → 10/10
```

## Test Output
```markdown
### Test Results: [Feature Name]

**Coverage**
| File | Statements | Branches | Functions | Lines |
|------|-----------|----------|-----------|-------|
| `file.ts` | 95% | 90% | 100% | 95% |

**Test Count**
- Pass: [N]
- Fail: [N]
```
