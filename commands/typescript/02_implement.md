# /typescript:implement

Implement TypeScript features following team standards.

**Agents:** tak-typescript-expert, react-figma-ui-engineer (if UI), framework-docs-researcher (if needed)

---

## Workflow

1. Setup (3min): Read requirements, identify parallel components
2. **Parallel Implementation** (15min): Core logic | Components/UI | Tests
3. Integration (3min): Integrate, run tests
4. Validation (2min): Verify standards, lint/tests

---

## Team Standards

### CRITICAL
- [ ] Named exports ONLY (no `export default`)
- [ ] Barrel exports (index.ts) where appropriate
- [ ] Imports: external → internal → relative → types
- [ ] const + arrow functions (no `function`)

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
### ✅ Implementation: [Feature]

**Files**
- ✨ New: [paths]
- 🔧 Modified: [paths]

**Tests**
✓ [test] (passed)

**Standards**
✅ Named exports | Barrel exports | Type safety | null handling | const+arrow
```
