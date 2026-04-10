---
name: angular-01-plan
description: "Angular project implementation planning. Triggers on 'angular plan', '앵귤러 계획', 'angular 기능 추가', 'angular feature', 'ng plan'. ComponentStore, Module-based architecture."
tools: Read, Bash, Glob, Grep, Agent
---

# Angular Plan

Angular-specific implementation planning. **Base: Reference `/plan` skill for same output structure.**

## ⛔ CRITICAL: STOP AFTER PLAN
Stop after planning is complete. Wait for the user's next command.
- **Plan file path:** `.claude/{YYYYMMDD}/PLAN-{HHMMSS}.md`
- Output: `Plan complete → Next: /angular-02-implement`

---

## Phase 0: Explore First
Read the code first. Parallel exploration via sub-agents:
- `codebase-researcher`: Analyze existing ComponentStore patterns, Module structure
- `best-practices-researcher`: Verify Angular official documentation
- `angular-componentstore-expert`: Check IonBasePage, TanStack Query patterns

## Phase 0.5: Rethink
For new features: "Is this really what the user wants?" Redefine (skip for bug fixes/infrastructure)

---

## Knowledge Sources
- `.claude/llms.txt`: [Project architecture — generated via `/generate-codebase-context`. Reference if present]
- `./CLAUDE.md`: [patterns found]
- `[reference file]`: [patterns extracted]

---

## Angular-Specific Planning

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

### Do
- Module-based architecture (no standalone)
- Register ComponentStore in Module providers
- Plan Page(smart) / Component(presentational) separation
- State design including message pattern

### [Warning] Ask First
- Creating new Shared service
- Modifying Core state-manager
- Changing Guard order

### 🚫 Don't
- Standalone component usage
- Direct API calls from Component
- Register Store in Component providers

---

## Angular-Specific Checklist Items

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

→ Next: `/angular-02-implement`
