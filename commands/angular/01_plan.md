# /angular:01_plan

Analyze issue and create Angular-specific implementation plan.

**Agents:** codebase-researcher, framework-docs-researcher, angular-componentstore-expert

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
  Run `/angular:02_implement` to start implementation.
  ```
- Wait until user explicitly enters the implement command

---

## Angular-Specific Planning

On top of the base `/common:plan` workflow, consider these Angular-specific items:

### Phase 1 Additional Analysis

1. **Module Structure**
   - Which Feature Module does this belong to?
   - Need to create new module?
   - What's needed from SharedModule?

2. **Routing Design**
   - Routes to register in routing module
   - Guard order: App → Auth → Feature
   - Lazy loading applicability

3. **ComponentStore Design**
   - State interface definition
   - Required Selectors, Updaters, Effects
   - Message pattern type definition

### Phase 2 Additional Research

- Analyze existing ComponentStore patterns (codebase-researcher)
- Check IonBasePage pattern for Ionic projects
- Verify TanStack Query usage and existing QueryService patterns

---

## Angular-Specific Boundaries

### ✅ Do
- Module-based architecture (no standalone)
- Register ComponentStore in Module providers
- Plan Page(smart) / Component(presentational) separation
- State design including message pattern

### ⚠️ Ask First
- Creating new Shared service
- Modifying Core state-manager
- Changing Guard order

### 🚫 Don't
- Standalone component usage
- Direct API calls from Component
- Register Store in Component providers

---

## Angular-Specific Checklist Items

Implementation Checklist must include:

- [ ] Create/modify Feature Module
- [ ] Register in Routing Module
- [ ] Define ComponentStore state interface
- [ ] Define Message pattern types
- [ ] Page/Component separation
- [ ] (Ionic) IonBasePage extends requirement
- [ ] (Ionic) IonicRouteStrategy verification
- [ ] (TanStack Query) Query keys factory requirement

---

## Review Focus Additional Items

- [ ] Store registered in Module providers
- [ ] destroyed$ cleanup pattern
- [ ] No Store injection in presentational components
- [ ] (Ionic) ionViewWillEnter lifecycle usage
