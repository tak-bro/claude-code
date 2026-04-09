---
name: tech-debt
model: sonnet
effort: medium
description: "Session-end cleanup. Find duplicates, dead code, and tech debt introduced during the session. Triggers on 'tech debt', '기술 부채', 'cleanup', '정리', 'end of session', '세션 종료', 'wrap up', 'session cleanup'."
tools: Read, Bash, Glob, Grep, Write, Edit, Agent
---

# Tech Debt

Run at the end of every session. Analyzes changes made during the session, detects duplications introduced, and cleans up.

**Recommended workflow:** After your feature work is done, before `/ship`, run `/tech-debt`.

---

## Workflow

### Phase 1: Session Change Analysis

```bash
# What changed this session? (uncommitted + recent commits)
git diff --name-only HEAD
git diff --cached --name-only
git log --oneline --since="4 hours ago" --name-only | grep -v "^[a-f0-9]"

# Collect all touched files
CHANGED_FILES=$(git diff --name-only HEAD; git diff --cached --name-only) | sort -u
```

### Phase 2: Spawn Analysis Agents

**Agent 1 — Duplication Detection**
- Compare changed files against existing codebase
- Detect if session introduced copy-paste logic
- Find functions/components that now exist in 2+ places
- Check for near-duplicate utility functions

```bash
# Find similar code blocks
for f in $CHANGED_FILES; do
  # Search for similar patterns in other files
  grep -rn "$(head -5 "$f" | grep -oE '[a-zA-Z]{10,}')" --include="*.ts" --include="*.tsx" --include="*.kt" --include="*.swift" | grep -v "$f"
done
```

**Agent 2 — Dead Code & Unused Artifacts**
- Exports added but not imported anywhere
- Files created but not referenced
- Functions defined but never called
- Interfaces/types defined but unused

```bash
# Check for unused exports in changed files
for f in $CHANGED_FILES; do
  exports=$(grep -oE "export (const|function|class|interface|type|enum) [a-zA-Z]+" "$f" | awk '{print $3}')
  for e in $exports; do
    count=$(grep -rn "$e" --include="*.ts" --include="*.tsx" | grep -v "$f" | wc -l)
    if [ "$count" -eq 0 ]; then
      echo "UNUSED: $e in $f"
    fi
  done
done
```

**Agent 3 — Structural Debt**
- File in wrong directory (doesn't follow project conventions)
- Missing barrel exports (index.ts)
- Inconsistent naming patterns introduced
- TODO/FIXME/HACK comments added without tracking

```bash
# TODOs introduced this session
git diff HEAD | grep "^+" | grep -iE "TODO|FIXME|HACK|XXX|TEMP"
```

### Phase 3: Consolidate Findings

Merge results from all agents. Categorize:
1. **Auto-fixable** — dead code removal, barrel export updates
2. **Needs decision** — duplication (extract shared? or intentional?)
3. **Track only** — TODOs, naming inconsistencies

### Phase 4: Auto-Fix (with approval)

For auto-fixable items:
- Remove confirmed dead code
- Create shared utilities for duplications (if approved)
- Update barrel exports
- Clean up temp artifacts

### Phase 5: Verify

(see `_shared/verification-commands.md`)

Ensure cleanup didn't break anything.

---

## Shared Library Creation

When duplications are found across modules:

```
1. Identify the common logic
2. Create/update shared utility in appropriate location:
   - src/shared/ or src/utils/ (web)
   - shared/ module (Kotlin multiplatform)
   - Common/ group (Swift)
3. Replace all duplicates with imports from shared
4. Run tests to confirm
```

---

## Output

```markdown
### 🧹 Tech Debt: Session Cleanup

**Session changes:** [N files, +X/-Y lines]

**Duplications Found:**
| # | Pattern | Locations | Action |
|---|---------|-----------|--------|
| 1 | Date formatting logic | `a.ts`, `b.ts` | → Extract to `shared/date.ts` |

**Dead Code:**
| # | Item | File | Action |
|---|------|------|--------|
| 1 | `unusedHelper()` | `utils.ts` | Remove |

**TODOs Added:** [N]
- `feat.ts:42` — TODO: handle edge case

**Auto-fixed:** [N items]
**Needs decision:** [N items]

**Post-cleanup verification:** [PASS ✅ / FAIL ❌]
```

→ After cleanup, proceed to `/ship` or `/verify`
