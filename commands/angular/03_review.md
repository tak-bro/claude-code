# /angular:review

Review Angular code against ComponentStore-as-Facade pattern and team standards.

**Agents:** angular-componentstore-expert, tak-typescript-reviewer, code-simplicity-reviewer

---

## Workflow

1. Setup (2min): Identify review scope
2. **Parallel** (10min): ComponentStore Review | Architecture | TypeScript Standards
3. **Analysis** (5min): Env+Storage | Subscription Cleanup | Complexity
4. Report (3min): Consolidate, prioritize

---

## Critical Checks

### ComponentStore
- ComponentStore = Facade (no separate Facade)?
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
Component calling API? → 🔴 Must use ComponentStore
Separate Facade class? → 🔴 Remove, ComponentStore IS Facade
providedIn: 'root'? → 🔴 Should be component-scoped
Ionic controllers? → 🔴 Use service wrappers
Missing _${env}? → 🔴 Add env suffix
Default exports? → 🔴 Use named exports
✅ All checks pass
```

---

## Output

```markdown
### 🔍 Review: [Feature]

**Score**: X/10

**Architecture**
✅ ComponentStore as Facade | Component-scoped | Service wrappers
❌ Issue: [description]

**Patterns**
✅ Module-based | Guards ordered | DestroyedService
❌ Issue: [description]

**Critical** 🔴 | **Important** 🟡 | **Nice-to-have** 🟢
```
