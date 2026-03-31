# Claude Philosophy

Core principles automatically loaded in all sessions.

## Core Principles

### 1. Explore Before Plan
- Planning without reading code causes hallucinations
- Use `/explore` or subagents to explore code first
- Delegate exploration to subagents to protect main context

### 2. Iterate Sufficiently in Plan Mode
- Enter Plan Mode with Shift+Tab
- Review plan 2-3 times
- Actively use `think hard` / `ultrathink`
- After sufficient iteration in Plan Mode, switch to auto-accept

### 3. Separate Context with Subagents
- Separate large-scale exploration and research to subagents
- Perform only core work in main context
- Utilize `use subagents` keyword

### 4. Review in Different Context than Implementation
- Run review skill with `context: fork`
- Test-Time Compute: same model but separated context = better results
- Smart Review Routing: assign specialized reviewers by domain

### 5. Deterministic Control via hooks/settings.json
- hooks exit 2 is more reliable than "never do this" in CLAUDE.md
- 3-layer control: settings.json deny (hard) → hooks exit 2 (conditional) → CLAUDE.md (guide)
- Critical restrictions must be enforced with hooks

### 6. Self-Validate Before Completion
- Stop hook automatically displays validation prompt
- Verify tests pass, remove debug code, follow conventions
- Check documentation drift (README, CLAUDE.md, CHANGELOG)

### 7. Update Documentation Together with Code
- Doc Sync: update related documentation simultaneously with code changes
- CLAUDE.md: add only reusable patterns, 3+ repetitions, critical safeguards
- Maintain below 200 lines through weekly cleanup
