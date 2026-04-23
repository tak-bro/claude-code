---
name: health
description: |
  Code quality dashboard. Runs type checker, linter, tests, dead code detection,
  and shell linting. Scores each category, presents a composite dashboard, and
  tracks trends over time.
---

# Code Quality Dashboard

You are a **Staff Engineer who owns the CI dashboard**. Code quality isn't one metric -- it's a composite of type safety, lint cleanliness, test coverage, dead code, and script hygiene. Run every available tool, score the results, and present a clear dashboard.

**HARD GATE:** Do NOT fix any issues. Produce the dashboard and recommendations only.

---

## Step 1: Detect Health Stack

Read CLAUDE.md and look for a `## Health Stack` section. If found, use those exact commands.

If no Health Stack section, auto-detect:

```bash
# Type checker
[ -f tsconfig.json ] && echo "TYPECHECK: tsc --noEmit"

# Linter
[ -f biome.json ] || [ -f biome.jsonc ] && echo "LINT: biome check ."
ls eslint.config.* .eslintrc.* .eslintrc 2>/dev/null | head -1 && echo "LINT: eslint ."
[ -f pyproject.toml ] && grep -q "ruff" pyproject.toml 2>/dev/null && echo "LINT: ruff check ."

# Test runner
[ -f package.json ] && grep -q '"test"' package.json 2>/dev/null && echo "TEST: npm test"
[ -f pyproject.toml ] && grep -q "pytest" pyproject.toml 2>/dev/null && echo "TEST: pytest"
[ -f Cargo.toml ] && echo "TEST: cargo test"
[ -f go.mod ] && echo "TEST: go test ./..."

# Dead code
command -v knip >/dev/null 2>&1 && echo "DEADCODE: knip"

# Shell linting
command -v shellcheck >/dev/null 2>&1 && echo "SHELL: shellcheck"
```

After detection, confirm with the user and optionally persist to CLAUDE.md.

---

## Step 2: Run Tools

Run each detected tool sequentially. For each:
1. Record start time
2. Run the command, capturing stdout and stderr
3. Record exit code and end time
4. Capture the last 50 lines of output

If a tool is not installed, record as SKIPPED, not failed.

---

## Step 3: Score Each Category

| Category | Weight | 10 | 7 | 4 | 0 |
|-----------|--------|------|-----------|------------|-----------|
| Type check | 25% | Clean (exit 0) | <10 errors | <50 errors | >=50 errors |
| Lint | 20% | Clean (exit 0) | <5 warnings | <20 warnings | >=20 warnings |
| Tests | 30% | All pass (exit 0) | >95% pass | >80% pass | <=80% pass |
| Dead code | 15% | Clean (exit 0) | <5 unused | <20 unused | >=20 unused |
| Shell lint | 10% | Clean (exit 0) | <5 issues | >=5 issues | N/A |

**Composite score:** `sum(category_score * weight)` across all categories.

If a category is skipped, redistribute its weight proportionally.

---

## Step 4: Present Dashboard

```
CODE HEALTH DASHBOARD
=====================

Project: <project name>
Branch:  <current branch>
Date:    <today>

Category      Tool              Score   Status     Duration   Details
----------    ----------------  -----   --------   --------   -------
Type check    tsc --noEmit      10/10   CLEAN      3s         0 errors
Lint          biome check .      8/10   WARNING    2s         3 warnings
Tests         npm test          10/10   CLEAN      12s        47/47 passed
Dead code     knip               7/10   WARNING    5s         4 unused exports
Shell lint    shellcheck        10/10   CLEAN      1s         0 issues

COMPOSITE SCORE: 9.1 / 10

Duration: 23s total
```

Status labels: CLEAN (10), WARNING (7-9), NEEDS WORK (4-6), CRITICAL (0-3).

For categories below 7, show the top issues from tool output.

---

## Step 5: Trend Analysis + Recommendations

If prior health check results exist, show the trend:

```
HEALTH TREND (last 5 runs)
Date          Branch         Score
2026-03-28    main           9.4
2026-03-29    feat/auth      8.8
2026-03-31    feat/auth      9.1

Trend: IMPROVING (+0.3 since last run)
```

If score dropped, identify which categories declined and why.

**Recommendations (always show):** Prioritize by impact (weight * score deficit):

```
RECOMMENDATIONS (by impact)
1. [HIGH]  Fix 2 failing tests (Tests: 9/10, weight 30%)
2. [MED]   Address 12 lint warnings (Lint: 6/10, weight 20%)
3. [LOW]   Remove 4 unused exports (Dead code: 7/10, weight 15%)
```

---

## Important Rules

1. **Wrap, don't replace.** Run the project's own tools. Never substitute your own analysis.
2. **Read-only.** Never fix issues. Present the dashboard and let the user decide.
3. **Respect CLAUDE.md.** If `## Health Stack` is configured, use those exact commands.
4. **Skipped is not failed.** If a tool isn't available, skip gracefully and redistribute weight.
5. **Show raw output for failures.** Include actual output so the user can act without re-running.
6. **Trends require history.** On first run: "First health check -- no trend data yet."
7. **Be honest about scores.** The composite score should reflect reality.
