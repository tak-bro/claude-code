# /typescript:01_plan

Analyze issue and create TypeScript-specific implementation plan.

**Agents:** codebase-researcher, framework-docs-researcher, tak-typescript-expert

**Extends:** `/common:plan` (use the same output structure)

## ⛔ CRITICAL: STOP AFTER PLAN
**After completing the plan, STOP and wait for user's next command.**
- NEVER automatically start implementation
- **Plan file path:** `.claude/{YYYYMMDD}/PLAN-{HHMMSS}.md` (same as `/common:plan`)
  ```bash
  DATE_DIR=".claude/$(date +%Y%m%d)"
  mkdir -p "$DATE_DIR"
  PLAN_FILE="$DATE_DIR/PLAN-$(date +%H%M%S).md"
  ```
- After creating the plan file, output:
  ```
  ✅ Plan complete: `.claude/{YYYYMMDD}/PLAN-{HHMMSS}.md`
  Run `/typescript:02_implement` to start implementation.
  ```
- Wait until user explicitly enters the implement command

---

## TypeScript-Specific Planning

On top of the base `/common:plan` workflow, consider these TypeScript-specific items:

### Phase 1 Additional Analysis

1. **Type Design**
   - Define core domain types/interfaces
   - Need for discriminated unions (Result<T>, state machines, etc.)
   - Utility type usage (Partial, Pick, Omit, Record, etc.)
   - Generic requirements and complexity level

2. **Module Structure**
   - Which module/folder does this belong to?
   - Barrel export (index.ts) plan
   - Dependencies with existing modules

3. **Error Handling Strategy**
   - Result pattern vs try-catch
   - null/undefined handling (early return)
   - Error type definitions

### Phase 2 Additional Research

- Analyze existing type patterns (codebase-researcher)
- Check similar utilities/services
- Review shared type definitions (avoid duplication)

---

## TypeScript-Specific Boundaries

### ✅ Do
- const + arrow function
- Named exports only
- Early returns for flat structure
- Complex conditions → named variables
- Clear type definitions

### ⚠️ Ask First
- Creating new utility types
- Modifying shared modules
- Generic nesting 3+ levels deep

### 🚫 Don't
- `export default`
- `function` keyword
- `any` without justification
- Deep nesting
- Unnecessary complexity in existing code

---

## TypeScript-Specific Checklist Items

Implementation Checklist must include:

- [ ] Define domain types/interfaces
- [ ] const + arrow function
- [ ] Named exports + barrel exports
- [ ] null/undefined handling (early returns)
- [ ] Complex conditions → named variables
- [ ] Testable structure
- [ ] 5-second naming rule pass
- [ ] No complexity added to existing code

---

## Review Focus Additional Items

- [ ] Type safety (no any or justified)
- [ ] Early returns for flat structure
- [ ] Named variables for readability
- [ ] Consistency with existing patterns
- [ ] Testable structure
