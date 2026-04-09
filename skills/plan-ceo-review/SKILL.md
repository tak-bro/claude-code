---
name: plan-ceo-review
model: sonnet
effort: low
description: "Product-level rethinking. 10-star product perspective. Triggers on 'ceo review', 'product review', '제품 리뷰', 'plan review', '계획 리뷰', '10-star', 'rethink'."
context: fork
tools: Read, Bash, Glob, Grep
---

# Plan CEO Review

Re-evaluate implementation plan from a product perspective. Use "10-star product" thinking to maximize user value.

---

## Pre-Review
1. Load plan file (`.claude/{date}/PLAN-*.md`)
2. Verify related requirements/issues

---

## Review Framework

### 1. User Value Check
- Does this feature actually solve a real user problem?
- Does the user want this, or do we want this?
- What is the MVP? (Can scope be reduced?)

### 2. 10-Star Experience
- 1-star: User cannot do what they want at all
- 5-star: Meets basic expectations
- 10-star: "Wow, it even does this?" (doesn't need to be realistic)
- Goal: Realistically implement 7-8 stars

### 3. Scope Sanity Check
- What should we remove from this PR?
- Review "What's NOT in This PR" list
- Check for over-engineering signals

### 4. Risk Assessment
- Technical risks
- Schedule risks
- Dependency risks

---

## Output

```markdown
### [Target] CEO Review: [Feature]

**User Value:** [High/Medium/Low]
**10-Star Rating:** [Current plan = X-star, possible = Y-star]

**Keep:**
- [Good decisions]

**Cut:**
- [Things to remove from scope]

**Add:**
- [Missing user value]

**Risk:**
- [Key risks and mitigation]

**Verdict:** [GO / REVISE / RETHINK]
```

→ GO: `/{framework}-implement`, REVISE: modify plan, RETHINK: `/office-hours`
