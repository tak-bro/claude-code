# /common:plan

## Purpose
Analyze issue and create implementation plan with clear boundaries and checklists for downstream workflow.

## ⛔ CRITICAL: STOP AFTER PLAN
**After completing the plan, STOP and wait for user's next command.**
- NEVER automatically start implementation
- After creating the plan file, output: "Plan complete. Run `/{framework}:02_implement` to start implementation."
- Wait until user explicitly enters the implement command

## Agents
- **codebase-researcher**: Analyze referenced files and extract patterns
- **framework-docs-researcher**: Look up framework-specific solutions (if needed)

### How to invoke agents
```
# Run agents in parallel using Task tool
Task(subagent_type: "general-purpose", prompt: "As codebase-researcher, analyze [files/patterns]")
Task(subagent_type: "general-purpose", prompt: "As framework-docs-researcher, research [technology] docs")
```

---

## Workflow

### Phase 1: Issue Analysis
1. Understand the issue/requirement
2. Identify referenced files to learn from
3. Determine tech stack and versions
4. **Detect package manager:**
   ```bash
   if [ -f "bun.lockb" ]; then pm="bun"
   elif [ -f "pnpm-lock.yaml" ]; then pm="pnpm"
   elif [ -f "yarn.lock" ]; then pm="yarn"
   else pm="npm"; fi
   ```

### Phase 2: Parallel Research 🔀
```bash
# Run simultaneously using Task tool
Lane 1: codebase-researcher     # Analyze referenced files
Lane 2: framework-docs-researcher  # Framework best practice (optional)
```

### Phase 3: Plan Creation
1. **Create plan file** (never overwrite existing):
   ```bash
   # Always write to new file. Never overwrite existing PLAN.md.
   # If .claude folder exists, write there.
   PLAN_FILE=".claude/PLAN-$(date +%Y%m%d-%H%M%S).md"
   ```
2. Extract patterns from referenced files
3. Define boundaries (Do/Don't)
4. Create implementation checklist
5. Define review focus areas

> **Note:** Reference framework-specific plans if available (`/angular:01_plan`, `/react:01_plan`, `/typescript:01_plan`)

---

## Output Structure

> ⚠️ Always write to `PLAN-{timestamp}.md`. Never overwrite existing plan files.

```markdown
### 📋 Implementation Plan: [Feature]

---

**Tech Stack** (check `package.json` for exact versions)
- [Framework]: [version from package.json]
- [State]: [version from package.json]
- [Build]: [tool]

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

⚠️ Ask First:
- [Decisions requiring confirmation]
- [Trade-offs to discuss]

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

**Implementation Checklist** (→ used in 02_implement phase)

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
{pm} run lint    # if available
{pm} test        # if available
```

---

**Review Focus** (→ checked in 03_review phase)

- [ ] [This feature's critical check point]
- [ ] [Edge case to verify]
- [ ] [Performance consideration]

---

**Decisions**

| Choice | Options | Selected | Rationale |
|--------|---------|----------|-----------|
| [Decision] | A, B | A | [Why] |
```
