---
name: review
description: |
  Pre-landing PR review. Analyzes diff against the base branch for SQL safety,
  race conditions, LLM trust boundary violations, conditional side effects,
  and other structural issues. Fix-first approach: auto-fixes mechanical issues,
  asks about judgment calls.
---

# Pre-Landing PR Review

Analyze the current branch's diff against the base branch for structural issues that tests don't catch.

---

## Step 1: Check branch

1. Run `git branch --show-current` to get the current branch.
2. If on the base branch, output: **"Nothing to review -- you're on the base branch or have no changes against it."** and stop.
3. Run `git fetch origin main --quiet && git diff origin/main --stat` to check if there's a diff. If no diff, output the same message and stop.

---

## Step 1.5: Scope Drift Detection

Before reviewing code quality, check: **did they build what was requested -- nothing more, nothing less?**

1. Read `TODOS.md` (if it exists). Read PR description (`gh pr view --json body --jq .body 2>/dev/null || true`). Read commit messages (`git log origin/main..HEAD --oneline`).
2. Identify the **stated intent** -- what was this branch supposed to accomplish?
3. Run `git diff origin/main...HEAD --stat` and compare files changed against stated intent.

4. Evaluate:

   **SCOPE CREEP detection:**
   - Files changed unrelated to stated intent
   - New features or refactors not mentioned in the plan
   - "While I was in there..." changes

   **MISSING REQUIREMENTS detection:**
   - Requirements not addressed in the diff
   - Test coverage gaps
   - Partial implementations

5. Output:
   ```
   Scope Check: [CLEAN / DRIFT DETECTED / REQUIREMENTS MISSING]
   Intent: <1-line summary>
   Delivered: <1-line summary>
   ```

6. This is **INFORMATIONAL** -- does not block the review.

---

### Plan File Discovery

Search for a plan file relevant to this branch:

```bash
BRANCH=$(git branch --show-current 2>/dev/null | tr '/' '-')
REPO=$(basename "$(git rev-parse --show-toplevel 2>/dev/null)")
# Search common plan file locations
for PLAN_DIR in ".claude/plans" "docs/plans" "."; do
  [ -d "$PLAN_DIR" ] || continue
  PLAN=$(ls -t "$PLAN_DIR"/*.md 2>/dev/null | xargs grep -l "$BRANCH" 2>/dev/null | head -1)
  [ -z "$PLAN" ] && PLAN=$(ls -t "$PLAN_DIR"/*.md 2>/dev/null | xargs grep -l "$REPO" 2>/dev/null | head -1)
  [ -n "$PLAN" ] && break
done
[ -n "$PLAN" ] && echo "PLAN_FILE: $PLAN" || echo "NO_PLAN_FILE"
```

If a plan file is found, extract actionable items and cross-reference against the diff. Classify each as:
- **DONE** -- Clear evidence in the diff
- **PARTIAL** -- Some work exists but incomplete
- **NOT DONE** -- No evidence in the diff
- **CHANGED** -- Different approach, same goal

For PARTIAL or NOT DONE items, investigate WHY (scope cut, context exhaustion, misunderstood requirement, blocked, forgotten).

---

## Step 2: Read the checklist

Read `.claude/skills/review/checklist.md` if it exists. If not available, use the built-in review categories below.

---

## Step 3: Get the diff

```bash
git fetch origin main --quiet
git diff origin/main
```

This includes both committed and uncommitted changes against the latest base branch.

---

## Step 4: Critical pass (core review)

Apply these categories against the diff:

**CRITICAL categories:**
- SQL & Data Safety
- Race Conditions & Concurrency
- LLM Output Trust Boundary
- Shell Injection
- Enum & Value Completeness

**INFORMATIONAL categories:**
- Async/Sync Mixing
- Column/Field Name Safety
- LLM Prompt Issues
- Type Coercion
- View/Frontend
- Time Window Safety
- Completeness Gaps
- Distribution & CI/CD

**Enum & Value Completeness requires reading code OUTSIDE the diff.** When the diff introduces a new enum value, use Grep to find all references, then check if the new value is handled everywhere.

## Confidence Calibration

Every finding MUST include a confidence score (1-10):

| Score | Meaning | Display rule |
|-------|---------|-------------|
| 9-10 | Verified by reading specific code | Show normally |
| 7-8 | High confidence pattern match | Show normally |
| 5-6 | Moderate, could be false positive | Show with caveat |
| 3-4 | Low confidence | Appendix only |
| 1-2 | Speculation | Only if severity P0 |

**Finding format:** `[SEVERITY] (confidence: N/10) file:line -- description`

---

## Step 4.5: Specialist Dispatch

### Detect stack and scope

```bash
STACK=""
[ -f Gemfile ] && STACK="${STACK}ruby "
[ -f package.json ] && STACK="${STACK}node "
[ -f requirements.txt ] || [ -f pyproject.toml ] && STACK="${STACK}python "
[ -f go.mod ] && STACK="${STACK}go "
[ -f Cargo.toml ] && STACK="${STACK}rust "
echo "STACK: ${STACK:-unknown}"
DIFF_INS=$(git diff origin/main --stat | tail -1 | grep -oE '[0-9]+ insertion' | grep -oE '[0-9]+' || echo "0")
DIFF_DEL=$(git diff origin/main --stat | tail -1 | grep -oE '[0-9]+ deletion' | grep -oE '[0-9]+' || echo "0")
DIFF_LINES=$((DIFF_INS + DIFF_DEL))
echo "DIFF_LINES: $DIFF_LINES"
```

### Select specialists

**Always-on (50+ changed lines):** Testing, Maintainability
**Conditional:** Security (if auth or backend changes), Performance (if backend or frontend), Data Migration (if migrations), API Contract (if API changes), Design (if frontend)

**If DIFF_LINES < 50:** Skip specialists entirely.

For each selected specialist, review the diff from that specialist's perspective, focusing on their domain-specific checklist.

### Compute PR Quality Score

`quality_score = max(0, 10 - (critical_count * 2 + informational_count * 0.5))`

---

## Step 5: Fix-First Review

**Every finding gets action -- not just critical ones.**

### Step 5a: Classify each finding

For each finding, classify as:
- **AUTO-FIX:** Mechanical, low-risk fixes (typos, missing imports, obvious null checks)
- **ASK:** Judgment calls, architectural decisions, anything with tradeoffs

### Step 5b: Auto-fix all AUTO-FIX items

Apply each fix directly. Output: `[AUTO-FIXED] [file:line] Problem -> what you did`

### Step 5c: Batch-ask about ASK items

Present ASK items for user input:

```
I auto-fixed N issues. M need your input:

1. [CRITICAL] file:line -- Problem description
   Fix: Recommended fix
   -> A) Fix  B) Skip

RECOMMENDATION: Fix both -- [reason]
```

### Step 5d: Apply user-approved fixes

Apply fixes for approved items.

### Verification of claims

Before producing final output:
- If you claim "this pattern is safe" -> cite the specific line
- If you claim "this is handled elsewhere" -> read and cite the handling code
- If you claim "tests cover this" -> name the test file and method
- Never say "likely handled" or "probably tested" -- verify or flag as unknown

---

## Step 5.5: TODOS cross-reference

Read `TODOS.md` (if it exists). Cross-reference:
- Does this PR close any open TODOs?
- Does this PR create work that should become a TODO?
- Are there related TODOs that provide context?

---

## Step 5.6: Documentation staleness check

Cross-reference the diff against documentation files. If code changes affect features described in docs that weren't updated, flag as INFORMATIONAL.

---

## Important Rules

- **Read the FULL diff before commenting.** Do not flag issues already addressed in the diff.
- **Fix-first, not read-only.** AUTO-FIX items are applied directly. ASK items only after user approval. Never commit, push, or create PRs -- that's /ship's job.
- **Be terse.** One line problem, one line fix. No preamble.
- **Only flag real problems.** Skip anything that's fine.
