---
name: ts-01-plan
description: "TypeScript 프로젝트 구현 계획. Triggers on 'typescript plan', 'ts plan', '타입스크립트 계획', 'ts 기능 추가'."
tools: Read, Bash, Glob, Grep, Agent
---

# TypeScript Plan

TypeScript-specific implementation planning. **Base: Reference `/plan` skill for same output structure.**

## ⛔ CRITICAL: STOP AFTER PLAN
Stop after planning. Wait for user's next command.
- **Plan file path:** `.claude/{YYYYMMDD}/PLAN-{HH-MM-SS}.md` (run `date` command for exact time with seconds)
- Output: `Plan complete → Next: /ts-02-implement`

---

## Phase 0: Explore First
- `codebase-researcher`: Analyze existing type patterns, module structure, conventions
- `best-practices-researcher`: TypeScript official docs, utility type patterns
- `tak-typescript-expert`: Verify existing tak convention compliance

## Phase 0.5: Rethink
For new features: "Is this really what the user wants?" Redefine the requirement.

---

## Knowledge Sources
- `.claude/llms.txt`: [Project architecture — must reference if exists]
- `./CLAUDE.md`: [patterns found]

---

## TypeScript-Specific Planning

### Phase 1 Additional Analysis

1. **Type Design**
   - Core domain types/interfaces
   - Discriminated unions (Result<T>, state machines)
   - Utility type usage (Partial, Pick, Omit, Record)
   - Generic requirements and complexity

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
- Review shared type definitions

---

## TypeScript-Specific Boundaries

### Do
- const + arrow function
- Named exports only
- Early returns for flat structure
- Complex conditions → named variables

### [Warning] Ask First
- Creating new shared type definitions
- Modifying existing barrel exports
- New utility type abstractions

### 🚫 Don't
- `export default` (except App entrypoint, config)
- `function` keyword
- `any` without justification
- Deep nesting

---

## TypeScript-Specific Checklist Items

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

→ Next: `/ts-02-implement`
