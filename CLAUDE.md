# Global Rules

## Never Forget
- DO NOT PUSH develop or main branch directly
- DO NOT CREATE PR yourself — generate PR description, let user create
- DO NOT modify files on protected branches (main/develop/master) without switching to feature branch

## Workflow

### Project Onboarding
```
/generate-codebase-context  → Generate .claude/llms.txt
/explore                    → Read-only analysis of specific modules
```

### Feature Development
```
/{framework}-01-plan → /{framework}-02-implement → /{framework}-03-review → /{framework}-04-fix → /qa → /ship
```

### Periodic
```
/retro     → Weekly retrospective
/insights  → Workflow improvement analysis
```

## Quick Start (Build/Test)
```bash
# Detect package manager
if [ -f "bun.lockb" ]; then pm="bun"
elif [ -f "pnpm-lock.yaml" ]; then pm="pnpm"
elif [ -f "yarn.lock" ]; then pm="yarn"
else pm="npm"; fi

# Common commands
$pm install          # Install dependencies
$pm run lint         # Lint
$pm test             # Test
$pm run build        # Build
```

## Code Style
- Named exports only (no `export default`). 예외: App entrypoint, config (e.g. `export default App`, `export default i18n`)
- const + arrow functions (no `function` keyword)
- Early returns for flat structure
- Complex conditions → named variables
- Barrel exports (index.ts) for all modules
- `any` requires justification comment

## Code Quality Limits
- Function body: max 80 lines (넘으면 분리)
- Cyclomatic complexity: max 8
- Function parameters: max 4 (넘으면 object parameter)
- Nesting depth: max 3 levels (넘으면 early return)
- Zero warnings policy: 모든 warning 수정 또는 inline ignore + 사유 주석
- Replace, don't deprecate: 새 구현이 대체하면 이전 코드 완전 삭제

## Auto Error Logging

When the user corrects or points out Claude's mistakes, automatically log them in the `## Error Log` section below.

**Log Format:**
```
- [YYYY-MM-DD] [Framework/Area] What was wrong → Correct approach
```

**Log Criteria:**
- When user explicitly corrects something
- When same mistake is repeated 2+ times
- When project conventions are violated

**Do NOT Log:**
- User's requirement changes (not a mistake)
- One-time typos

---

## Error Log

<!-- Claude will automatically add entries below -->

@RTK.md
