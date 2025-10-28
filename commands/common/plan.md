# /plan 

## Purpose
Analyze issue and create implementation plan by referencing existing code patterns.

## Agents
- **codebase-researcher**: Analyze referenced files and extract patterns
- **framework-docs-researcher**: Look up framework-specific solutions (if needed)

---

## Workflow

### Phase 1: Issue Analysis (2 min)
```bash
1. Understand the issue/requirement
2. Identify referenced files to learn from
3. Extract key patterns to apply
```

### Phase 2: Parallel Research 🔀 (3 min)
```bash
# Run simultaneously
Lane 1: codebase-researcher     # Analyze referenced files
Lane 2: framework-docs-researcher  # Framework best practice (optional)
```

### Phase 3: Plan Creation (3 min)
```bash
3. Extract patterns from referenced files
4. Identify what needs modification
5. Create step-by-step implementation plan
```

---

## Output Structure

```markdown
### 📋 Implementation Plan: [Feature]

**Knowledge Sources**
- ./claude/llms.txt: [current patterns] (if exists)
- ./CLAUDE.md: [current patterns] (if exists)
- External: [research findings]

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