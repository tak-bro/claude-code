# /angular:review

Review Angular code against ComponentStore-as-Facade pattern.

**Agents:** angular-componentstore-expert, tak-typescript-reviewer, code-simplicity-reviewer

---

## Workflow

1. Setup (2min): Identify scope
2. **Parallel** (10min): ComponentStore | Architecture | TypeScript
3. Analysis (5min): Env+Storage | Cleanup | Complexity
4. Report (3min): Consolidate, prioritize

---

## Checks

### ComponentStore
- ComponentStore = Facade (no separate)?
- Component-scoped providers?
- Effects → API pattern?

### Architecture
- Module-based (NOT standalone)?
- Component → ComponentStore → API?
- Service wrappers (NOT Ionic controllers)?

### Storage & Cleanup
- LocalStorage keys: `_${env}`?
- DestroyedService cleanup?

### Standards
- Named exports ONLY?
- const + arrow functions?
- Guards: App → Auth → Feature?

---

## Decision Tree

```
Component → API? → 🔴 Use ComponentStore
Separate Facade? → 🔴 ComponentStore IS Facade
providedIn: 'root'? → 🔴 Component-scoped
Ionic controllers? → 🔴 Service wrappers
Missing _${env}? → 🔴 Add suffix
Default exports? → 🔴 Named exports
✅ All pass
```

---

## Output

```markdown
### 🔍 [Feature] - X/10

**Architecture:** ✅ ComponentStore=Facade | Scoped | Wrappers
**Patterns:** ✅ Module | Guards | DestroyedService
**Storage:** ✅ Env suffixes | Cleanup

🔴 Critical | 🟡 Important | 🟢 Nice-to-have
```
