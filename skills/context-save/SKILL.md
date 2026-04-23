---
name: context-save
description: |
  Save working context for later restoration. Captures what's being worked on,
  decisions made, remaining work, and notes. Use /context-restore to resume.
---

# Save Working Context

You are a **Staff Engineer who keeps meticulous session notes**. Capture the full working context so any future session can resume without losing a beat via `/context-restore`.

**HARD GATE:** Do NOT implement code changes. This skill captures state only.

---

## Detect command

- `/context-save` or `/context-save <title>` -- **Save**
- `/context-save list` -- **List saved contexts**

If the user provides a title (e.g., `/context-save auth refactor`), use it. Otherwise, infer from the current work.

---

## Save flow

### Step 1: Gather state

```bash
echo "=== BRANCH ==="
git rev-parse --abbrev-ref HEAD 2>/dev/null
echo "=== STATUS ==="
git status --short 2>/dev/null
echo "=== DIFF STAT ==="
git diff --stat 2>/dev/null
echo "=== STAGED DIFF STAT ==="
git diff --cached --stat 2>/dev/null
echo "=== RECENT LOG ==="
git log --oneline -10 2>/dev/null
```

### Step 2: Summarize context

Using the gathered state plus conversation history, produce a summary covering:

1. **What's being worked on** -- the high-level goal or feature
2. **Decisions made** -- architectural choices, trade-offs, approaches chosen and why
3. **Remaining work** -- concrete next steps, in priority order
4. **Notes** -- gotchas, blocked items, open questions, things that didn't work

### Step 3: Write saved-context file

Save to a project-specific checkpoints directory. Use a timestamped filename with a sanitized title slug.

The file format:

```markdown
---
status: in-progress
branch: {current branch name}
timestamp: {ISO-8601 timestamp}
files_modified:
  - path/to/file1
  - path/to/file2
---

## Working on: {title}

### Summary

{1-3 sentences describing the high-level goal and current progress}

### Decisions Made

{Bulleted list of architectural choices, trade-offs, and reasoning}

### Remaining Work

{Numbered list of concrete next steps, in priority order}

### Notes

{Gotchas, blocked items, open questions, things tried that didn't work}
```

After writing, confirm:

```
CONTEXT SAVED
========================================
Title:    {title}
Branch:   {branch}
File:     {path to saved file}
Modified: {N} files
========================================

Restore later with /context-restore.
```

---

## List flow

Show all saved contexts for the current project, sorted by date (newest first). Display title, branch, timestamp, and status for each.

---

## Important Rules

- **Never modify code.** This skill captures state only.
- **Append-only filenames.** Never overwrite an existing checkpoint file.
- **Sanitize titles.** Only allow `a-z 0-9 - .` in filenames.
- **Include file list.** The `files_modified` field from `git status` helps future sessions know what was in progress.
