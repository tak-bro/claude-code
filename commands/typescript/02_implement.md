# /typescript:implement

Implement TypeScript features following team standards.

**Agents:** tak-typescript-expert, react-figma-ui-engineer (if UI), framework-docs-researcher (if needed)

---

## Workflow

1. Setup (3min): Read requirements, identify parallel components
2. **Parallel** (15min): Core logic | Components/UI | Tests
3. Integration (3min): Integrate, run tests
4. Validation (2min): Standards, lint/tests

---

## Standards

### CRITICAL
- [ ] Named exports ONLY (no `export default`)
- [ ] Barrel exports (index.ts)
- [ ] Imports: external → internal → relative → types
- [ ] const + arrow (no `function`)

### Type Safety
- [ ] No `any` (or justified + comment)
- [ ] null/undefined handled (early returns)
- [ ] Complex conditions → named variables

### Quality
- [ ] Clear names (5-second rule)
- [ ] Testable structure
- [ ] Didn't complicate existing code

---

## Output

```markdown
### ✅ [Feature]

**Files:** ✨ New: [...] | 🔧 Modified: [...]
**Tests:** ✓ [...]
**Standards:** Named exports | Barrel | Type safety | const+arrow
```
