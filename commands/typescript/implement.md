# /typescipr:implement

## Purpose
Implement TypeScript features following using TDD, apply `./claude/llms.txt` patterns

## Agents
- **react-figma-ui-engineer**: UI from Figma
- **framework-docs-researcher**: Framework reference
- **tak-typescript-expert**: Use as implementation checklist


---

## Workflow

### Phase 1: Setup (3 min)
```bash
1. Read requirements/plan or user's text
2. Identify parallel components
```

### Phase 2: Parallel Implementation 🔀 (15 min)
```bash
# If components are independent
Lane 1: Core logic (pure functions)
Lane 2: Components/UI 
Lane 3: Tests (TDD)
```

### Phase 3: Integration (3 min)
```bash
4. Integrate components
5. Run tests (10x for probabilistic features)
```

### Phase 4: Parallel Documentation 🔀 (2 min)
```bash
Lane 1: Code comments
Lane 2: README updates
```

---

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