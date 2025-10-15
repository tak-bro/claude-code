# Compounding Engineering Workflow

## Complete Flow

```bash
/plan "feature"
  ↓ (13 min)
/implement  
  ↓ (30 min)
/review
  ↓ (25 min)
/implement --fix-review-issues
  ↓ (10 min)
/review --strict
  ↓ (5 min)
git add CLAUDE.md
git commit & push

Total: 83 min (vs 175 min sequential = 52% faster)
```

---

## 3-Terminal Setup

```
┌──────────────┬──────────────┬──────────────┐
│ LEFT         │ MIDDLE       │ RIGHT        │
│ Planning 🔵  │ Implement 🟡 │ Review 🟢    │
│              │              │              │
│ /plan        │ /implement   │ /review      │
└──────────────┴──────────────┴──────────────┘
```

---

## Knowledge System

```
.claude/llms.txt (50-100 lines)
├─ Tech stack
├─ Core principles
└─ File structure

CLAUDE.md (200-500 lines)
├─ Coding Style
├─ Review Checklist
├─ Testing Strategy
├─ Current Focus
└─ Recent Lessons (20 max)

CLAUDE_ARCHIVE.md (unlimited)
├─ 2025 Q3 (45 lessons)
├─ 2025 Q2 (38 lessons)
└─ 2025 Q1 (32 lessons)
```

---

## Key Principles

### Parallel Execution
```
✅ Can Parallel:
- Multiple research agents
- UI + Backend + Tests
- Multiple review perspectives
- Multiple issue fixes

❌ Must Sequential:
- Integration (after components)
- Synthesis (after research)
- Consolidation (after reviews)
```

### Archive Search
```bash
# Always search before planning/reviewing
grep -i "feature|keyword" CLAUDE_ARCHIVE.md

# Find recurring issues
grep -i "error|bug|issue" CLAUDE_ARCHIVE.md
```

### CLAUDE.md Updates
```
Add if:
✅ Reusable pattern
✅ 3rd+ occurrence (recurring)
✅ Critical prevention

Don't add:
❌ One-off solution
❌ Obvious pattern
```

### Cleanup Schedule
```
Weekly: /cleanup
Goal: Keep 200-500 lines
Archive: 90+ days old
```

---

## Time Savings

| Phase | Sequential | Parallel | Saved |
|-------|-----------|----------|-------|
| Plan | 15 min | 13 min | 2 min |
| Implement | 60 min | 30 min | 30 min |
| Review | 45 min | 25 min | 20 min |
| Fix | 50 min | 10 min | 40 min |
| **Total** | **170 min** | **78 min** | **92 min** |

**Result: 54% faster** 🚀

---

## Compounding Effect

```
Month 1:
- CLAUDE.md: 15 patterns
- Speed: 1x

Month 6:
- CLAUDE.md: 25 patterns (active)
- Archive: 120 patterns (searchable)
- Speed: 2x (past mistakes auto-avoided)

Month 12:
- CLAUDE.md: 30 patterns (active)
- Archive: 250 patterns (searchable)
- Speed: 3x (most issues prevented)
```

---

## Remember

🎯 Every implementation should:
1. Search archive (avoid past mistakes)
2. Update CLAUDE.md (teach system)
3. Run in parallel (maximize speed)
4. Cleanup weekly (keep lean)

**Build systems that build systems.** 🚀