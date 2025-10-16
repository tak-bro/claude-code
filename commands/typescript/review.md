# /typescript:review 

## Purpose
Multi-perspective code review, detect recurring issues against tak's strict quality standards.

## Agent
- **tak-typescript-reviewer**: TypeScript quality
- **code-simplicity-reviewer**: YAGNI violations
- **design-implementation-reviewer**: UI vs Figma
- **tak-document-writer**: Documentation style

---

## Workflow

### Phase 1: Sequential (2 min)
```bash
1. Determine review types needed
2. Identify parallel reviews
```

### Phase 2: Parallel Reviews 🔀 (10 min)
```bash
Lane 1: tak-typescript-reviewer      # Type safety
Lane 2: design-implementation-reviewer  # UI fidelity (if UI)
Lane 3: Security + Performance       # Vulnerabilities
Lane 4: Style + Consistency          # ./claude/llms.txt compliance
```

### Phase 3: Parallel Analysis 🔀 (8 min)
```bash
Lane 1: code-simplicity-reviewer     # Find unnecessary complexity
```

### Phase 4: Sequential (2 min)
```bash
3. Generate suggestions
4. Auto-fix simple issues
```

---

## Output Structure

```markdown
### 🔍 Review: [Some Features#]

**Score**: [score] / 10

**Issues Found**
🔴 Critical: [issue] (fix: [solution])
🟡 Important: [issue]
🟢 Nice-to-have: [issue]
```

