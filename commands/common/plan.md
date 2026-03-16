# /common:plan

## Purpose
Analyze issue and create implementation plan with clear boundaries and checklists for downstream workflow.

## ⛔ CRITICAL: STOP AFTER PLAN
**After completing the plan, STOP and wait for user's next command.**
- NEVER automatically start implementation
- After creating the plan file, output:
  ```
  ✅ Plan complete: `.claude/{YYYYMMDD}/PLAN-{HHMMSS}.md`
  Run `/{framework}:02_implement` to start implementation.
  ```
- Wait until user explicitly enters the implement command

## Agents
- **codebase-researcher**: Analyze referenced files and extract patterns
- **framework-docs-researcher**: Look up framework-specific solutions (if needed)

### How to invoke agents
```
# Run agents in parallel using Task tool with correct subagent_type
Task(subagent_type: "codebase-researcher", prompt: "Analyze [files/patterns] and extract implementation patterns")
Task(subagent_type: "framework-docs-researcher", prompt: "Research [technology] docs for [specific topic]")
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
Run agents simultaneously using Task tool:
| Lane | Agent | Purpose |
|------|-------|---------|
| 1 | `codebase-researcher` | Analyze referenced files |
| 2 | `framework-docs-researcher` | Framework best practices (optional) |

### Phase 3: Plan Creation
1. **Create plan file in `.claude/{date}/` folder** (never overwrite existing):
   ```bash
   # Always write to .claude/YYYYMMDD/ folder with timestamp
   DATE_DIR=".claude/$(date +%Y%m%d)"
   mkdir -p "$DATE_DIR"
   PLAN_FILE="$DATE_DIR/PLAN-$(date +%H%M%S).md"
   ```
2. Extract patterns from referenced files
3. Define boundaries (Do/Don't)
4. Create implementation checklist
5. Define review focus areas

> **Note:** Reference framework-specific plans if available (`/angular:01_plan`, `/react:01_plan`, `/typescript:01_plan`)

---

## Output Structure

> ⚠️ Always write to `.claude/{YYYYMMDD}/PLAN-{HHMMSS}.md`. Never overwrite existing plan files.

```markdown
# 📋 Implementation Plan: [Feature]

> **Created:** YYYY-MM-DD HH:MM
> **Package Manager:** {pm}

---

## Tech Stack
(check `package.json` for exact versions)
- [Framework]: [version from package.json]
- [State]: [version from package.json]
- [Build]: [tool]

---

## Knowledge Sources
- `./CLAUDE.md`: [patterns found]
- `[reference file]`: [patterns extracted]
- External: [research findings]

---

## Boundaries

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

## Files to Create/Modify

| File | Action | Purpose |
|------|--------|---------|
| `path/to/file.ts` | Create | [Purpose] |
| `path/to/existing.ts` | Modify | [What changes] |

---

## Implementation Checklist
(→ used in 02_implement phase)

- [ ] [Step 1]
- [ ] [Step 2]
- [ ] [Feature-specific requirement]
- [ ] Named exports only
- [ ] Barrel exports (index.ts)
- [ ] Verification commands pass

---

## Verification Commands

```bash
{pm} run lint
{pm} test
```

---

## Review Focus
(→ checked in 03_review phase)

- [ ] [This feature's critical check point]
- [ ] [Edge case to verify]
- [ ] [Performance consideration]

---

## Decisions

| Choice | Options | Selected | Rationale |
|--------|---------|----------|-----------|
| [Decision] | A, B | A | [Why] |
```
