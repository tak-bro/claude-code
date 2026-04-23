---
name: document-release
description: |
  Post-ship documentation update. Audits all project docs against the branch diff,
  auto-fixes factual updates, asks about risky changes, polishes CHANGELOG voice,
  checks cross-doc consistency.
---

# Document Release: Post-Ship Documentation Update

Runs **after code is committed** but **before the PR merges**. Ensures every documentation file is accurate, up to date, and user-friendly.

You are mostly automated. Make obvious factual updates directly. Stop and ask only for risky or subjective decisions.

**Only stop for:** Risky/questionable changes, VERSION bump decisions, new TODOS, cross-doc narrative contradictions.

**Never stop for:** Factual corrections from the diff, updating paths/counts/versions, fixing stale cross-references, CHANGELOG voice polish, marking TODOS complete.

**NEVER:** Overwrite/replace CHANGELOG entries, bump VERSION without asking, use Write tool on CHANGELOG.md.

---

## Step 1: Pre-flight & Diff Analysis

1. Check branch -- abort if on base branch.

2. Gather context:
```bash
git diff main...HEAD --stat
git log main..HEAD --oneline
git diff main...HEAD --name-only
```

3. Discover all doc files:
```bash
find . -maxdepth 2 -name "*.md" -not -path "./.git/*" -not -path "./node_modules/*" | sort
```

4. Classify changes: new features, changed behavior, removed functionality, infrastructure.

---

## Step 2: Per-File Documentation Audit

Read each doc file and cross-reference against the diff:

**README.md:** Features accurate? Install/setup instructions valid? Examples current?

**ARCHITECTURE.md:** Diagrams match code? Design decisions accurate? (Be conservative.)

**CONTRIBUTING.md:** Walk through setup as a new contributor. Would each step succeed?

**CLAUDE.md / project instructions:** Structure section matches? Commands accurate?

**Other .md files:** Read, determine purpose, check against diff.

Classify updates as:
- **Auto-update** -- factual corrections clearly warranted by the diff
- **Ask user** -- narrative changes, section removal, large rewrites, ambiguous relevance

---

## Step 3: Apply Auto-Updates

Make all clear factual updates using the Edit tool. For each file modified, output a one-line summary of what specifically changed.

**Never auto-update:** README introduction, ARCHITECTURE philosophy, security model descriptions, entire section removals.

---

## Step 4: Ask About Risky Changes

For each risky update, ask with context, the specific decision, recommendation, and options including "Skip."

---

## Step 5: CHANGELOG Voice Polish

**CRITICAL -- NEVER CLOBBER CHANGELOG ENTRIES.**

Only modify wording within existing entries. Never delete, reorder, or replace.

If CHANGELOG was modified in this branch:
- **Sell test:** Would a user think "I want to try that"?
- Lead with what the user can now DO, not implementation details
- "You can now..." not "Refactored the..."
- Flag entries that read like commit messages

---

## Step 6: Cross-Doc Consistency

1. README feature list matches CLAUDE.md descriptions?
2. ARCHITECTURE components match CONTRIBUTING's project structure?
3. CHANGELOG version matches VERSION file?
4. **Discoverability:** Is every doc file reachable from README or CLAUDE.md?

---

## Step 7: TODOS.md Cleanup

If TODOS.md exists:
1. Mark items completed by this branch's diff
2. Flag items whose references changed significantly
3. Check diff for TODO/FIXME/HACK comments that should be captured

---

## Step 8: VERSION Bump Question

**NEVER bump without asking.**

If VERSION exists and wasn't bumped, ask with recommendation (usually Skip for docs-only changes). If already bumped, verify it covers the full scope of changes.

---

## Step 9: Commit & Output

1. Stage modified doc files by name (never `git add -A`)
2. Commit: `docs: update project documentation`
3. Push to current branch
4. Update PR body with a `## Documentation` section listing what changed

**Final output:**
```
Documentation health:
  README.md       [status] ([details])
  ARCHITECTURE.md [status] ([details])
  CONTRIBUTING.md [status] ([details])
  CHANGELOG.md    [status] ([details])
  TODOS.md        [status] ([details])
  VERSION         [status] ([details])
```

---

## Important Rules

- **Read before editing.** Always read the full file first.
- **Never clobber CHANGELOG.** Polish wording only.
- **Never bump VERSION silently.** Always ask.
- **Be explicit about what changed.** Every edit gets a one-line summary.
- **Discoverability matters.** Every doc should be reachable from README or CLAUDE.md.
