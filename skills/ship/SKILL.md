---
name: ship
model: sonnet
effort: medium
description: "Prepare for deployment. Test → lint → doc sync → push → create PR. Triggers on 'ship', 'deploy', 'deploy', 'push', 'PR', 'pull request', 'merge ready'."
tools: Read, Bash, Glob, Grep, Write, Edit
---

# Ship

Push code that has passed all validations and create a PR.

---

## Pre-Ship Checklist

### Verification Commands
(see `_shared/verification-commands.md`)

### Doc Sync
- [ ] README.md needs update?
- [ ] CLAUDE.md needs update?
- [ ] CHANGELOG needs update?
- [ ] API documentation changed?

### Git Hygiene
- [ ] Follow Conventional Commits?
- [ ] Squash WIP commits?
- [ ] Follow branch naming convention?

---

## Workflow

### Phase 1: Final Verification
(see `_shared/verification-commands.md` — run test, lint, build)

### Phase 2: Commit & Push
```bash
# [Warning] Follow CLAUDE.md rule "Never push develop/main"
git add -A
git commit -m "feat(scope): description"
git push origin HEAD
```

### Phase 3: PR Description Generation
Auto-generated content:
- Summary (change summary)
- Changes (changes per file)
- How to Test (testing method)
- Screenshots (if UI changes)
- Checklist

---

## [Warning] Safety Rules
- **NEVER push to main or develop directly**
- **NEVER create PR without user confirmation**
- Only generate PR description, confirm with user before creating actual PR

---

## Output

```markdown
### [Ship] Ship: [Feature]

**Pre-Ship:**
- [x] Tests pass
- [x] Lint pass
- [x] Build pass
- [x] Doc sync checked

**Branch:** `feat/PROJ-XXX-feature-name`
**Commits:** [N commits]

**PR Description (Draft):**
[Generated PR description]

[Warning] Ready to create PR? (y/n)
```
