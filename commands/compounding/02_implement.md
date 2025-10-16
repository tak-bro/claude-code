# /implement Command

## Purpose
Write code using TDD, apply `./claude/llms.txt` and `./CLAUDE.md` patterns, update learnings.

## Agents
- **react-figma-ui-engineer**: UI from Figma
- **framework-docs-researcher**: Framework reference
- **tak-typescript-expert**: Use this as your checklist when implementing features, creating components, or refactoring code.

---

## Workflow

### Phase 1: Sequential (5 min)
```bash
1. Read plan
2. Search ./CLAUDE_ARCHIVE.md: grep -i "feature" ./CLAUDE_ARCHIVE.md
3. Identify parallel components
```

### Phase 2: Parallel Implementation 🔀 (20 min)
```bash
# If independent components exist
Lane 1: UI (react-figma-ui-engineer)
Lane 2: Logic
Lane 3: Tests (TDD)
```

### Phase 3: Sequential (5 min)
```bash
4. Integrate components
5. Run tests (10x for probabilistic features)
6. Apply ./claude/llms.txt and ./CLAUDE.md style
```

### Phase 4: Parallel Documentation 🔀 (2 min)
```bash
Lane 1: Code comments
Lane 2: README updates
Lane 3: ./CLAUDE.md updates
```

---

## ./CLAUDE.md Update

Add if:
- ✅ New reusable pattern
- ✅ Tricky problem solved
- ✅ Framework gotcha
- ❌ One-off solution

Format:
```markdown
## [Section]
### Pattern: [Name] (Added: YYYY-MM-DD, PR #XXX)
**Context**: [When]
**Implementation**: [How]
**Rationale**: [Why]
```

## Output Structure

```markdown
### ✅ Implementation: [Feature]

**Files**
- ✨ New: [paths]
- 🔧 Modified: [paths]

**Tests**
✓ [test] (passed)
✗ [test] (9/10 - acceptable)

**📝 ./CLAUDE.md Updates**
- Added: [pattern]
- Updated: [section]
```
