---
name: design-consultation
description: "UI/UX design research + DESIGN.md generation. Triggers on 'design', 'UI', 'UX', 'design consultation', 'design doc', 'DESIGN.md'."
tools: Read, Bash, Glob, Grep, Write
---

# Design Consultation

Perform UI/UX design research and generate DESIGN.md.

---

## Workflow

### Phase 1: Requirements Gathering
- User personas
- Core user flows
- Design constraints (existing design system, brand guide)

### Phase 2: Research
- Investigate UI patterns in similar products/features
- Accessibility requirements (WCAG)
- Responsive requirements

### Phase 3: Design Decisions
- Component structure
- State-based UI (loading, error, empty, success)
- Interaction patterns (hover, focus, animation)
- Responsive breakpoints

### Phase 4: DESIGN.md Generation

---

## Output: DESIGN.md

```markdown
# Design: [Feature Name]

## User Flow
1. [Step 1] → [Step 2] → [Step 3]

## Components
| Component | Purpose | States |
|-----------|---------|--------|
| [name] | [role] | loading, error, empty, success |

## Design Tokens
- Colors: [from design system]
- Typography: [sizes, weights]
- Spacing: [scale]

## Responsive Breakpoints
- Mobile: < 768px
- Tablet: 768px - 1024px
- Desktop: > 1024px

## Accessibility
- [ ] Keyboard navigation
- [ ] Screen reader support
- [ ] Color contrast (WCAG AA)
- [ ] Focus indicators

## Interactions
- [Element]: [hover/click/focus behavior]
```

→ Next: `/{framework}-plan` (refer to DESIGN.md for planning)
