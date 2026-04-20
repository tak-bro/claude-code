---
name: office-hours
description: "Refine new ideas. Brainstorming stage before code. Triggers on 'office hours', 'brainstorm', 'idea', 'discuss'."
tools: Read, Bash, Glob, Grep
model: sonnet

# Office Hours

Refine ideas before writing code.

---

## When to Use
- You have a new feature idea but it's not concrete
- You want to discuss technical approach
- You need to decide between multiple options
- "Can we do this?" stage

---

## Framework

### 1. Problem Definition
- What are you trying to solve?
- Who is it for?
- Why do it now?

### 2. Solution Exploration
- Compare approach A, B, C
- Pros and cons of each approach
- Compatibility with existing codebase

### 3. Feasibility Check
- Is it technically possible?
- How long will it take? (S/M/L)
- Dependencies?

### 4. Decision
- Recommended approach + reasoning
- Next steps proposal

---

## Output

```markdown
### 💡 Office Hours: [Topic]

**Problem:** [1-2 sentences]

**Options:**
| Option | Pros | Cons | Effort |
|--------|------|------|--------|
| A | [pros] | [cons] | S/M/L |
| B | [pros] | [cons] | S/M/L |

**Recommendation:** [Option] because [reason]

**Next Steps:**
→ `/explore` for codebase analysis
→ `/{framework}-plan` for implementation planning
```
