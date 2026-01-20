# /plan

## Purpose
Analyze issue and create implementation plan with clear boundaries and checklists for downstream workflow.

## Agents
- **codebase-researcher**: Analyze referenced files and extract patterns
- **framework-docs-researcher**: Look up framework-specific solutions (if needed)

---

## Workflow

### Phase 1: Issue Analysis (2 min)
1. Understand the issue/requirement
2. Identify referenced files to learn from
3. Determine tech stack and versions

### Phase 2: Parallel Research 🔀 (3 min)
```bash
# Run simultaneously
Lane 1: codebase-researcher     # Analyze referenced files
Lane 2: framework-docs-researcher  # Framework best practice (optional)
```

### Phase 3: Plan Creation (3 min)
1. Extract patterns from referenced files
2. Define boundaries (Do/Don't)
3. Create implementation checklist
4. Define review focus areas

---

## Output Structure

```markdown
### 📋 Implementation Plan: [Feature]

---

**Tech Stack**
- [Framework] [version] (e.g., React 18.x, Angular 18.x)
- [State] [version] (e.g., TanStack Query v5, ComponentStore)
- [Build] [tool] (e.g., Vite, Nx)

---

**Knowledge Sources**
- `./CLAUDE.md`: [patterns found]
- `[reference file]`: [patterns extracted]
- External: [research findings]

---

**Boundaries**

✅ Do:
- [What this implementation SHOULD do]
- [Patterns to follow]

🚫 Don't:
- [What to avoid]
- [Anti-patterns for this feature]

---

**Files to Create/Modify**

| File | Action | Purpose |
|------|--------|---------|
| `path/to/file.ts` | Create | [Purpose] |
| `path/to/existing.ts` | Modify | [What changes] |

---

**Implementation Checklist** (→ implement 단계에서 사용)

- [ ] [Step 1]
- [ ] [Step 2]
- [ ] [Feature-specific requirement]
- [ ] Named exports only
- [ ] Barrel exports (index.ts)
- [ ] Verification commands pass

---

**Verification Commands**

```bash
# {pm} = npm, yarn, pnpm, bun (use project's package manager)
{pm} run typecheck    # or: ng build
{pm} run lint
{pm} test
```

---

**Review Focus** (→ review 단계에서 확인)

- [ ] [This feature's critical check point]
- [ ] [Edge case to verify]
- [ ] [Performance consideration]

---

**Decisions**

| Choice | Options | Selected | Rationale |
|--------|---------|----------|-----------|
| [Decision] | A, B | A | [Why] |
```
