# /cleanup Command

## Purpose
Keep ./CLAUDE.md lean (200-500 lines) by archiving old patterns.

## When to Run
- Manually
- 📊 When ./CLAUDE.md > 500 lines
- 📅 Weekly (every Friday)
- 🔄 End of sprint/quarter

---

## Process

### 1. Analyze
```bash
Current: 650 lines
Lessons: 45 entries
Checklist: 12 items
Status: ⚠️  Needs cleanup
```

### 2. Archive (90+ days)
```bash
Move to ./CLAUDE_ARCHIVE.md:
- Lessons older than 90 days
- Patterns absorbed into checklist
```

### 3. Merge Duplicates
```bash
3 entries about "Error Handling"
→ Merge into 1 comprehensive pattern
Saved: 85 lines
```

### 4. Remove Obsolete
```bash
Delete:
- Deprecated tech patterns
- Not referenced in 60+ days
Saved: 45 lines
```

### 5. Promote to .claude/llms.txt
```bash
Frequently referenced (10+ times):
- Core patterns → .claude/llms.txt
```

---

## Rules

### Archive (90-day rule)
```
IF age > 90 days
AND not in checklist
AND not referenced in 30 days
THEN archive
```

### Merge
```
IF same category
AND same problem
THEN merge
```

### Delete
```
IF not referenced in 60 days
AND obsolete tech
THEN delete
```

### Promote
```
IF referenced 10+ times
AND core architecture
THEN promote to llms.txt
```

---

## Output Structure

```markdown
### 🧹 Cleanup Complete

**Before**: 650 lines, 45 lessons
**After**: 380 lines, 20 lessons

**Actions**
✅ Archived: 25 lessons
✅ Merged: 3 patterns (-85 lines)
✅ Removed: 3 obsolete (-45 lines)
✅ Promoted: 2 to llms.txt

**Next cleanup**: 2025-11-01
```