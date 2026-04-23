---
name: retro
description: "Weekly retrospective. Commit-based analysis. Triggers on 'retro', 'retrospective', 'weekly review', 'what did I do'."
tools: Read, Bash, Glob, Grep
model: sonnet

# Retro

Generate weekly retrospective based on Git log.

---

## Workflow

### Phase 1: Data Collection
```bash
# Recent 1 week commits
git log --oneline --since="1 week ago" --author="$(git config user.email)"

# Change statistics
git diff --stat HEAD~$(git log --oneline --since="1 week ago" | wc -l)..HEAD

# File change frequency
git log --since="1 week ago" --name-only --pretty=format: | sort | uniq -c | sort -rn | head -20
```

### Phase 2: Analysis
- Completed features/bug fixes
- Most frequently changed files (hotspots)
- Commit pattern analysis (feat/fix/refactor ratio)
- Work time analysis

### Phase 3: Insights
- Repeating patterns (automation opportunity?)
- Hotspot files (refactoring needed?)
- Technical debt accumulation areas

---

## Output

```markdown
### 📊 Weekly Retro: [Date Range]

**Summary**
- Commits: [N]
- Files changed: [N]
- Lines: +[X] / -[Y]

**Completed**
- [feat] [Feature descriptions]
- [fix] [Bug fix descriptions]

**Hotspots** (most changed files)
1. `path/to/file` — [N changes]
2. `path/to/file` — [N changes]

**Patterns**
- [Observation about work patterns]

**Action Items**
- [ ] [Improvement suggestion]
- [ ] [Automation opportunity]
- [ ] [Refactoring target]
```

→ If code quality issues found, go to `/health`
