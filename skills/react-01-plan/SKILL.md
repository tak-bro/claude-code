---
name: react-01-plan
description: "React 프로젝트 구현 계획 수립. Triggers on 'react plan', '리액트 계획', 'react feature', 'react 기능 추가'. Feature Library, TanStack Query, Zustand, Nx monorepo."
tools: Read, Bash, Glob, Grep, Agent
---

# React Plan

React-specific implementation planning. **Base: Reference `/plan` skill for same output structure.**

## ⛔ CRITICAL: STOP AFTER PLAN
Stop after planning. Wait for user's next command.
- **Plan file path:** `.claude/{YYYYMMDD}/PLAN-{HHMMSS}.md`
- Output: `Plan complete → Next: /react-02-implement`

---

## Phase 0: Explore First
- `codebase-researcher`: Analyze existing Feature Library patterns, hooks
- `best-practices-researcher`: React/TanStack Query official documentation
- `react-feature-library-expert`: Verify existing library structure

## Phase 0.5: Rethink
For new features: "Is this really what the user wants?" Redefine the requirement.

---

## Knowledge Sources
- `.claude/llms.txt`: [Project architecture — must reference if exists]
- `./CLAUDE.md`: [patterns found]

---

## React-Specific Planning

### Phase 1 Additional Analysis

1. **Feature Library Placement**
   - Which Feature Library does this belong to?
   - Need to create new lib? (`npx nx g @nx/js:lib`)
   - Should it be added to existing lib?

2. **Data Flow Design**
   - Component → Hook → TanStack Query → API path
   - What API endpoints are needed?
   - Server state vs Client state distinction

3. **State Management Strategy**
   - TanStack Query: Server data (lists, details, CRUD)
   - Zustand: Global client state (UI state, user preferences)
   - useState: Local state only (form inputs, toggles)

4. **Query Key Strategy**
   - Use `createQueryKeys` factory
   - Check for conflicts with existing query keys
   - Invalidation strategy

### Phase 2 Additional Research
- Analyze existing Feature Library patterns (codebase-researcher)
- Check similar hooks/apis patterns
- Review Zustand store structure

---

## React-Specific Boundaries

### Do
- Feature Library structure (apis/, hooks/, types/, consts/)
- Barrel exports (index.ts) in all folders
- Component → Hook → API data flow
- cn() utility for Tailwind class merging

### [Warning] Ask First
- Creating new Zustand store
- Modifying shared hooks
- New query key namespace

### 🚫 Don't
- Direct API calls from Component
- Shared state with useState
- Missing barrel exports
- Over-abstraction (WET > DRY)

---

## React-Specific Checklist Items

- [ ] Create/verify Feature Library
- [ ] Define API functions (apis/)
- [ ] Define Query keys factory (hooks/)
- [ ] Define TanStack Query hooks (hooks/)
- [ ] Define Types (types/)
- [ ] Barrel exports in all folders
- [ ] State management: Zustand(global) / TanStack Query(server) / useState(local)
- [ ] ~200 lines guideline, ~5 business props

---

## Review Focus Additional Items

- [ ] Component → Hook → API flow compliance
- [ ] useState not used for shared state
- [ ] Barrel exports completeness
- [ ] No query key conflicts
- [ ] Tailwind classes organized with cn()

## Plan Mode Usage
- Shift+Tab → Plan Mode → Review 2-3 times
- `think hard` for state management decisions

→ Next: `/react-02-implement`
