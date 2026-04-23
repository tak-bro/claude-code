---
name: context-restore
description: |
  Restore saved working context from /context-save. Loads the most recent
  checkpoint and presents it so the user can resume work.
---

# Restore Saved Working Context

You are a **Staff Engineer reading a colleague's session notes** to pick up exactly where they left off. Load the most recent saved context and present it clearly.

**HARD GATE:** Do NOT implement code changes. This skill only reads saved context files.

**Default:** Load the most recent saved context across ALL branches (enables cross-branch resume).

---

## Detect command

- `/context-restore` -- load the most recent saved context (any branch)
- `/context-restore <title-fragment-or-number>` -- load a specific saved context
- `/context-restore list` -- tell the user to use `/context-save list` instead

---

## Restore flow

### Step 1: Find saved contexts

Look for checkpoint files in the project's checkpoints directory. List all `.md` files sorted by filename (YYYYMMDD-HHMMSS prefix gives chronological order).

Search across all branches -- do NOT filter by current branch.

### Step 2: Load the right file

- If the user specified a title fragment or number: find the matching file
- Otherwise: load the most recent file (newest YYYYMMDD-HHMMSS prefix)

Read the chosen file and present:

```
RESUMING CONTEXT
========================================
Title:       {title}
Branch:      {branch from frontmatter}
Saved:       {timestamp, human-readable}
Status:      {status}
========================================

### Summary
{summary from saved file}

### Remaining Work
{remaining work items}

### Notes
{notes}
```

If the current branch differs from the saved context's branch, note this: "This context was saved on branch `{branch}`. You are currently on `{current branch}`. You may want to switch branches."

### Step 3: Offer next steps

- A) Continue working on the remaining items
- B) Show the full saved file
- C) Just needed the context, thanks

If A, summarize the first remaining work item and suggest starting there.

---

## If no saved contexts exist

"No saved contexts yet. Run `/context-save` first to save your current working state."

---

## Important Rules

- **Never modify code.** This skill only reads saved files.
- **Always search across all branches by default.** Cross-branch resume is the whole point.
- **"Most recent" = filename YYYYMMDD-HHMMSS prefix**, not filesystem mtime.
