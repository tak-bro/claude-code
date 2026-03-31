---
name: typescript-01-plan
description: "TypeScript 프로젝트 구현 계획. Triggers on 'typescript plan', 'ts plan', '타입스크립트 계획', 'ts 기능 추가'."
tools: Read, Bash, Glob, Grep, Agent
---

# TypeScript Plan

TypeScript-specific implementation planning. **Base: Reference `/plan` skill.**

## ⛔ CRITICAL: STOP AFTER PLAN
- Output: `Plan complete → Next: /typescript-02-implement`

---

## Phase 0: Explore First + Phase 0.5: Rethink
(see `/plan` base skill)

## Knowledge Sources
- `.claude/llms.txt`: [Must reference if exists]
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

→ Next: `/typescript-02-implement`
