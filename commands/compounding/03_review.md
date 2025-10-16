# /review Command

## Purpose
Multi-perspective code review, detect recurring issues, update ./CLAUDE.md.

## Agents
- **tak-typescript-reviewer**: TypeScript quality
- **code-simplicity-reviewer**: YAGNI violations
- **design-implementation-reviewer**: UI vs Figma
- **tak-document-writer**: Documentation style

---

## Workflow

### Phase 1: Sequential (2 min)
```bash
1. Determine review types needed
2. Identify parallel reviews
```

### Phase 2: Parallel Reviews 🔀 (10 min)
```bash
Lane 1: tak-typescript-reviewer      # Type safety
Lane 2: design-implementation-reviewer  # UI fidelity (if UI)
Lane 3: Security + Performance       # Vulnerabilities
Lane 4: Style + Consistency          # ./CLAUDE.md compliance
```

### Phase 3: Sequential (3 min)
```bash
3. Consolidate findings
4. Apply ./CLAUDE.md checklist
5. Search archive: grep -i "issue" ./CLAUDE_ARCHIVE.md
```

### Phase 4: Parallel Analysis 🔀 (8 min)
```bash
Lane 1: code-simplicity-reviewer     # Find unnecessary complexity
Lane 2: Archive search               # Recurring issue check
```

### Phase 5: Sequential (2 min)
```bash
6. Generate suggestions
7. Auto-fix simple issues
8. Update ./CLAUDE.md
```

---

## CLAUDE.md Update

Add if:
- ✅ Recurring issue (3rd+ occurrence)
- ✅ Critical bug pattern
- ✅ New prevention rule
- ❌ One-time issue

Format:
```markdown
## Review Checklist
### [Issue] (Added: YYYY-MM-DD, PR #XXX)
**Problem**: [What]
**Solution**: [How to prevent]
**Detection**: [How to find]

## Lessons Learned
### [Title] (Added: YYYY-MM-DD)
**Issue**: [What caught]
**Root Cause**: [Why]
**Prevention**: [How to avoid]
```

---

## Output Structure

```markdown
### 🔍 Review: [PR #XXX]

**Score**: 7/10

**Issues Found**
🔴 Critical: [issue] (fix: [solution])
🟡 Important: [issue]
🟢 Nice-to-have: [issue]

**📝 CLAUDE.md Updates**
## Review Checklist
### [New check added]

## Lessons Learned  
### [New lesson added]

**Recurring Issues** 🚨
⚠️  [Issue] found 6 times total:
- Archive: PR #156, #178, #215
- Recent: PR #235, #238, #241
→ Enhanced to ENFORCED rule
```
