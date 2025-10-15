# /plan Command

## Purpose
Analyze requirements and create implementation plans using past learnings + external research.

## Agents
- **best-practices-researcher**: Industry standards
- **framework-docs-researcher**: Framework documentation
- **codebase-researcher**: Internal patterns

---

## Workflow

### Phase 1: Sequential (5 min)
```bash
1. Requirements analysis
2. Extract keywords
3. Search ./CLAUDE_ARCHIVE.md: grep -i "keywords" ./CLAUDE_ARCHIVE.md
```

### Phase 2: Parallel Research 🔀 (5 min)
```bash
# Run simultaneously
Lane 1: best-practices-researcher    # Industry best practices
Lane 2: framework-docs-researcher    # Official docs
Lane 3: codebase-researcher          # Internal patterns
```

### Phase 3: Sequential (5 min)
```bash
4. Consolidate all findings
5. Prioritize sources: ./.claude/llms.txt > ./CLAUDE.md > Archive > External
6. Write plan with archived lessons cited
```

---

## Output Structure

```markdown
### 📋 Implementation Plan: [Feature]

**Knowledge Sources**
- ./CLAUDE_ARCHIVE.md: [lessons found]
- ./CLAUDE.md: [current patterns]
- External: [research findings]

**Archived Lessons Applied** 🎯
✅ [Lesson from PR #XXX]
⚠️  [Conflicting pattern - using current]

**File Structure**
src/
  └── [files]

**Steps**
1. [Step]
2. [Step]

**Testing**
- [Strategy]

**Decisions**
- Choice: [X]
- Rationale: [Why]
- Source: [Citation]
```