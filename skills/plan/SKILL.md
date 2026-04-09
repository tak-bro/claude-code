---
name: plan
description: "Project implementation planning (Base). Referenced by all framework plans. Triggers on 'plan', '계획', '구현 계획', 'implementation plan', '설계'. Issue analysis → parallel research → plan creation."
tools: Read, Bash, Glob, Grep, Agent
---

# Implementation Plan (Base)

All framework-specific plan skills (`/angular-01-plan`, `/react-01-plan`, `/ts-01-plan`) reference this base.

## ⛔ CRITICAL: STOP AFTER PLAN
**Stop after planning is complete. Wait for the user's next command.**
- Never automatically start implementation
- After creating the plan file, output:
  ```
  Plan complete: `.claude/{YYYYMMDD}/PLAN-{HHMMSS}.md`
  → Next: `/{framework}-implement`
  ```

---

## Phase 0: Explore First (NEW — Claude Philosophy)
**Read the code first before planning.** Parallel exploration via sub-agents:

```
Task(subagent_type: "codebase-researcher", prompt: "Analyze [target module] structure, patterns, conventions")
Task(subagent_type: "best-practices-researcher", prompt: "Research [technology] for [specific topic]")
```

- Planning without reading code causes hallucinations
- Delegate exploration to sub-agents to protect the main context

## Phase 0.5: Rethink (NEW)
**Only for new features** (skip for bug fixes/infrastructure):
- "Is this really what the user wants?" Redefine once
- 10-star product: Think 10x beyond user expectations (implement realistically)
- Watch for over-engineering: Plan only "what's needed now"

## Phase 1: Issue Analysis
1. Understand issue/requirements
2. Identify files to reference
3. Verify tech stack and versions
4. **Detect package manager** (see `_shared/verification-commands.md`)

## Phase 2: Parallel Research 🔀
Run sub-agents simultaneously:
| Lane | Agent | Purpose |
|------|-------|---------|
| 1 | `codebase-researcher` | Analyze reference files, extract patterns |
| 2 | `best-practices-researcher` | Framework best practices (optional) |

## Phase 3: Plan Creation
1. Create plan file in `.claude/{date}/` folder (don't overwrite existing files):
   ```bash
   DATE_DIR=".claude/$(date +%Y%m%d)"
   mkdir -p "$DATE_DIR"
   PLAN_FILE="$DATE_DIR/PLAN-$(date +%H%M%S).md"
   ```
2. Extract patterns from reference files
3. Define boundaries (Do/Don't)
4. Write implementation checklist
5. Define review focus areas

## Plan Mode Usage (Claude Philosophy)
- Enter Plan Mode with Shift+Tab
- Iterate through plan 2-3 times
- Use `think hard` or `ultrathink` for complex decisions
- After sufficient iteration in Plan Mode, switch to auto-accept

---

## Knowledge Sources
- `.claude/llms.txt`: [Project architecture, file manifest, development guidelines — generated via `/generate-codebase-context`. Reference if present]
- `./CLAUDE.md`: [patterns found]
- `[reference file]`: [patterns extracted]

---

## Output Structure

> [Warning] Always write to `.claude/{YYYYMMDD}/PLAN-{HHMMSS}.md`. Never overwrite existing plan files.

```markdown
# [Plan] Implementation Plan: [Feature]

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
- `.claude/llms.txt`: [architecture summary]
- `./CLAUDE.md`: [patterns found]
- `[reference file]`: [patterns extracted]
- External: [research findings]

---

## Boundaries

Do:
- [What this implementation SHOULD do]
- [Patterns to follow]

[Warning] Ask First:
- [Decisions requiring confirmation]
- [Trade-offs to discuss]

🚫 Don't:
- [What to avoid]
- [Anti-patterns for this feature]

---

## What's NOT in This PR
- [Explicit list of things intentionally excluded]
- [Future work that is NOT part of this plan]

---

## Files to Create/Modify

| File | Action | Purpose |
|------|--------|---------|
| `path/to/file.ts` | Create | [Purpose] |
| `path/to/existing.ts` | Modify | [What changes] |

---

## Implementation Checklist

- [ ] [Step 1]
- [ ] [Step 2]
- [ ] Named exports only
- [ ] Barrel exports (index.ts)
- [ ] Verification commands pass

---

## Verification Commands
(see _shared/verification-commands.md)

---

## Review Focus
- [ ] [This feature's critical check point]
- [ ] [Edge case to verify]

---

## Decisions

| Choice | Options | Selected | Rationale |
|--------|---------|----------|-----------|
| [Decision] | A, B | A | [Why] |
```

→ Next: `/{framework}-implement`
