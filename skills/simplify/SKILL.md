---
name: simplify
description: "Simplify and clean up code. Remove duplicates, dead code, unnecessary complexity. Triggers on 'simplify', '단순화', 'clean up', '정리', 'refactor', 'reduce complexity', '복잡도'."
tools: Read, Bash, Glob, Grep, Write, Edit, Agent
---

# Simplify

Refine code for clarity while preserving functionality. Spawns 3 agents to evaluate code across multiple dimensions.

---

## Workflow

### Phase 1: Scope Detection

```bash
# If user specifies files/dirs, use those. Otherwise use recent changes:
git diff --name-only HEAD~1
# Or analyze full project:
find src/ -name "*.ts" -o -name "*.tsx" -o -name "*.kt" -o -name "*.swift" | head -100
```

### Phase 2: Spawn 3 Evaluation Agents

**Agent 1 — Duplication & Dead Code**
- Identify duplicated logic across files
- Find unused exports, unreachable code, dead branches
- Detect copy-paste patterns that should be abstracted

**Agent 2 — Complexity & Readability**
- Functions exceeding 30 lines
- Cyclomatic complexity hotspots (deeply nested conditionals)
- Unclear naming, magic numbers, missing types
- Overly abstract patterns (unnecessary indirection)

**Agent 3 — Structure & Dependencies**
- Circular dependencies
- Files with too many responsibilities
- Import chains that could be shortened
- Opportunities for co-location

### Phase 3: Consolidate & Rank

Merge findings from all 3 agents. Rank by impact:
1. **High** — Duplicated logic, dead code (safe to remove)
2. **Medium** — Complexity reduction (refactor needed)
3. **Low** — Style/naming improvements

**Present findings to user. Do NOT auto-fix without approval.**

### Phase 4: Apply Changes

After approval, apply changes and run verification:
(see `_shared/verification-commands.md`)

---

## What This Does NOT Do
- Change public API signatures without explicit approval
- Remove code that "looks" unused but might be dynamically referenced
- Rewrite working code just for style preferences

---

## Output

```markdown
### 🧹 Simplify: [Scope]

**Analyzed:** [N files]

| # | Category | File | Finding | Impact | Action |
|---|----------|------|---------|--------|--------|
| 1 | Duplication | `a.ts`, `b.ts` | Same validation logic | High | Extract to shared util |
| 2 | Dead Code | `old.ts` | Unused export `foo` | High | Remove |
| 3 | Complexity | `handler.ts:45` | 5-level nesting | Medium | Extract early returns |

**Summary:**
- Duplications: [N]
- Dead code: [N items removable]
- Complexity hotspots: [N]
- Estimated line reduction: -[X] lines

**Proceed with fixes? (y/n)**
```

