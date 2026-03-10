# /common:debug

Analyze error messages and stack traces, then provide solutions.

**Agents:** codebase-researcher, framework-docs-researcher

---

## Workflow

### Phase 1: Error Analysis

1. **Parse Error Message**
   - Error type (TypeError, ReferenceError, HttpError, etc.)
   - Location (file, line, column)
   - Stack trace interpretation

2. **Error Classification**

   | Type | Example | Approach |
   |------|---------|----------|
   | Runtime | TypeError, null reference | Stack trace → source code tracing |
   | Build | TypeScript, Webpack, Vite | Error message → config/type check |
   | Lint | ESLint, Prettier | Rule check → code fix |
   | Test | Jest, Vitest assertion | Expected vs actual comparison |
   | Network | CORS, 401, 500 | API config/auth check |

### Phase 2: Related Code Search

```bash
# Parallel execution (using Task tool)
Lane 1: codebase-researcher        # Analyze error file and related code
Lane 2: framework-docs-researcher  # Research official docs for error (if needed)
```

**codebase-researcher scope:**
- Error source file
- Error function's callers (call chain)
- Related type definitions
- Similar pattern files (working code)

**framework-docs-researcher scope (when applicable):**
- Official docs related to error message
- Breaking changes (if error after version upgrade)
- Known issues / GitHub issues

### Phase 3: Root Cause Analysis

1. **Direct cause** → Code that triggers the error
2. **Root cause** → Why that code is problematic
3. **Related factors** → Environment, config, dependencies

### Phase 4: Solution Proposal

---

## Output Structure

```markdown
### Debug Report: [Error Summary]

---

**Error Summary**
- Type: [Runtime/Build/Lint/Test/Network]
- Message: `[error message]`
- Location: `[file:line]`

---

**Cause Analysis**

| | Description |
|---|------|
| Direct cause | [Code/situation triggering the error] |
| Root cause | [Why that code is problematic] |
| Related factors | [Environment, config, dependencies] |

---

**Solution**

Recommended fix:
[Code change + reason]

Alternative:
[Other approaches if applicable]

---

**Verification**
```bash
# Commands to verify fix
{pm} run build
{pm} test -- --testPathPattern="affected-file"
```

---

**Prevention**
- [Suggestions to prevent similar errors]
```

---

## Common Error Patterns Reference

### TypeScript

| Error | Common Cause | Solution |
|-------|-------------|----------|
| `TS2322: Type 'X' is not assignable to 'Y'` | Type mismatch | Type narrowing, type guard |
| `TS2345: Argument of type 'X' is not assignable` | Function argument type mismatch | Check argument types |
| `TS7006: Parameter implicitly has 'any' type` | Missing type declaration | Add explicit type |
| `TS2339: Property 'X' does not exist` | Accessing non-existent property | Check type definition |

### Angular

| Error | Common Cause | Solution |
|-------|-------------|----------|
| `NullInjectorError: No provider for X` | DI not registered | Add to Module providers |
| `ExpressionChangedAfterItHasBeenCheckedError` | Value changed during CD | Async handling, OnPush |
| `Can't bind to 'X' since it isn't a known property` | Missing Module import | Import the module |

### React

| Error | Common Cause | Solution |
|-------|-------------|----------|
| `Too many re-renders` | Infinite re-render loop | Check useEffect deps |
| `Cannot update during render` | State change during render | Move to useEffect |
| `Invalid hook call` | Hook rules violation | Move outside conditionals |
| `Hydration mismatch` | SSR/CSR mismatch | Handle dynamic content |
