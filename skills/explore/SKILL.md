---
name: explore
model: haiku
description: "Read-only code analysis. Explore before plan. Triggers on 'explore', '탐색', 'analyze code', '코드 분석', 'read code', 'understand code', '코드 이해'."
context: fork
tools: Read, Bash, Glob, Grep
---

# Explore

Analyze code in read-only mode before planning. Executed as a sub-agent to avoid polluting main context.

**Context: fork** — Runs in a separate context. Does not consume main session tokens.

---

## When to Use
- When joining a new project
- Before working on new modules/features
- When you need to understand code structure
- Pre-planning investigation

## Workflow

### Phase 1: Structure Discovery
```bash
# Understand project structure
find . -type f -name "*.ts" -o -name "*.tsx" -o -name "*.js" | head -50
cat package.json | jq '.dependencies, .devDependencies'
```

### Phase 2: Pattern Extraction
- Identify existing architecture patterns
- Verify state management approach
- Analyze routing structure
- Identify common utilities

### Phase 3: Convention Discovery
- Naming conventions
- File/folder structure
- Import/Export patterns
- Test patterns

---

## Output

```markdown
### [Review] Explore: [Module/Feature]

**Architecture**
- Pattern: [MVC/Feature Library/ComponentStore/etc.]
- State: [TanStack Query/Zustand/ComponentStore/etc.]
- Routing: [pattern]

**Key Files**
| File | Purpose | Patterns |
|------|---------|----------|
| `path/to/file` | [role] | [patterns used] |

**Conventions**
- Exports: [named/default]
- Functions: [arrow/function keyword]
- State: [management pattern]
- Tests: [framework + pattern]

**Dependencies**
- [key dependency]: [version] — [purpose]

**Notes for Plan**
- [Important constraint or pattern to follow]
- [Anti-pattern to avoid]
```

→ Next: `/{framework}-plan`
