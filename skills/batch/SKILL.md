---
name: batch
model: sonnet
description: "Parallelize tasks across isolated work trees. Triggers on 'batch', '배치', 'parallel', '병렬', 'migrate', '마이그레이션', 'bulk update', '일괄'."
context: fork
tools: Read, Bash, Glob, Grep, Write, Edit, Agent
---

# Batch

Parallelize repeatable tasks using isolated git work trees. Each agent works independently, preventing conflicts.

---

## When to Use
- Library/framework migration across multiple modules
- Bulk file transformations (rename, restructure, format)
- Applying the same pattern to N files/components
- Multi-module refactoring where changes don't depend on each other

---

## Workflow

### Phase 1: Plan (mandatory before execution)

Analyze the task and produce a plan:

```
1. Current State — what exists now
2. Units of Work — break task into independent, parallelizable units
3. Per-Unit Instructions — exact steps each agent will execute
4. Verification — how to confirm each unit succeeded
5. Merge Strategy — how to combine results
```

**Present plan to user. Do NOT proceed without approval.**

### Phase 2: Work Tree Setup

```bash
# Detect base branch
BASE=$(git rev-parse --abbrev-ref HEAD)

# Create isolated work trees per unit
for i in $(seq 1 $UNIT_COUNT); do
  BRANCH="batch/${TASK_NAME}-unit-${i}"
  git worktree add "../worktree-${i}" -b "$BRANCH" "$BASE"
done
```

### Phase 3: Spawn Agents

For each unit of work, spawn a dedicated agent:
- Give each agent its unit-specific prompt + shared context
- Each agent operates exclusively in its own work tree
- Agents report progress back to the main agent

### Phase 4: Monitor & Collect

- Track each agent's completion status
- Collect results and any errors
- If an agent fails, report the failure (do NOT auto-retry without approval)

### Phase 5: Merge

```bash
# Merge each work tree branch back
for i in $(seq 1 $UNIT_COUNT); do
  BRANCH="batch/${TASK_NAME}-unit-${i}"
  git merge "$BRANCH" --no-edit
done

# Cleanup work trees
for i in $(seq 1 $UNIT_COUNT); do
  git worktree remove "../worktree-${i}" --force
  git branch -d "batch/${TASK_NAME}-unit-${i}"
done
```

### Phase 6: Verify merged result
(see `_shared/verification-commands.md`)

---

## Safety Rules
- **NEVER skip the plan phase**
- **NEVER merge without running verification on the merged result**
- Each unit must be truly independent — if there are dependencies, execute sequentially
- Maximum 5 parallel agents unless user specifies otherwise
- Clean up ALL work trees after completion, even on failure

---

## Output

```markdown
### ⚡ Batch: [Task Name]

**Units:** [N] parallel tasks
**Strategy:** [work tree per unit]

| # | Unit | Status | Changes | Notes |
|---|------|--------|---------|-------|
| 1 | [desc] | ✅ DONE | +X/-Y | — |
| 2 | [desc] | ✅ DONE | +X/-Y | — |

**Merge:** [CLEAN / CONFLICTS — list]
**Verification:** [PASS / FAIL]

**Total:** [N files changed, +X/-Y lines]
```

