# /react:01_plan

Analyze issue and create React-specific implementation plan.

**Agents:** codebase-researcher, framework-docs-researcher, react-feature-library-expert

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
  Run `/react:02_implement` to start implementation.
  ```
- Wait until user explicitly enters the implement command

---

## React-Specific Planning

On top of the base `/common:plan` workflow, consider these React-specific items:

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
   - Invalidation strategy (which queries to invalidate after mutation)

### Phase 2 Additional Research

- Analyze existing Feature Library patterns (codebase-researcher)
- Check similar hooks/apis patterns
- Review Zustand store structure (if global state addition needed)

---

## React-Specific Boundaries

### ✅ Do
- Feature Library structure (apis/, hooks/, types/, consts/)
- Barrel exports (index.ts) in all folders
- Component → Hook → API data flow
- cn() utility for Tailwind class merging

### ⚠️ Ask First
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

Implementation Checklist must include:

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
