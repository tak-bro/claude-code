---
name: insights
description: "Weekly workflow improvement analysis. Triggers on 'insights', 'workflow analysis', 'improvement'."
tools: Read, Bash, Glob, Grep
---

# Insights

Analyze weekly workflow and suggest improvements.

---

## Workflow

### Phase 1: Data Collection
```bash
# Recent commit patterns
git log --oneline --since="1 week ago" | head -30

# Repeating file patterns
git log --since="1 week ago" --name-only --pretty=format: | sort | uniq -c | sort -rn | head -10

# Commit type distribution
git log --oneline --since="1 week ago" | grep -oP "^[a-f0-9]+ \K[a-z]+" | sort | uniq -c | sort -rn

# Check CLAUDE.md error log
cat .claude/CLAUDE.md 2>/dev/null | grep -A1 "Error Log"
```

### Phase 2: Pattern Analysis
- Identify repeating tasks (automation candidates)
- Analyze hotspot files (refactoring candidates)
- Analyze error log patterns (process improvement candidates)
- Identify repeating issues in reviews

### Phase 3: Recommendations
- Suggest new skill/command additions
- Suggest CLAUDE.md rule additions
- Suggest hook additions
- Suggest workflow improvements

---

## Output

```markdown
### 💡 Weekly Insights: [Date Range]

**Automation Opportunities**
- [Repeating task] → [Automation method]

**Refactoring Targets**
- `path/to/hotspot` — [N changes this week, reason]

**Process Improvements**
- [Current pain point] → [Suggested improvement]

**CLAUDE.md Suggestions**
- Add: [new rule based on repeated mistakes]
- Remove: [outdated rule]

**Skill/Hook Suggestions**
- New skill: [description]
- New hook: [description]

**Priority Actions**
1. [Most impactful improvement]
2. [Next improvement]
3. [Third improvement]
```

→ If automation needed, create skill or hook
