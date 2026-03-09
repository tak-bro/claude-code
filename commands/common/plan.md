# /common:plan

## Purpose
Analyze issue and create implementation plan with clear boundaries and checklists for downstream workflow.

## Agents
- **codebase-researcher**: Analyze referenced files and extract patterns
- **framework-docs-researcher**: Look up framework-specific solutions (if needed)

### Agent 호출 방법
```
# Task 도구를 사용하여 에이전트를 병렬 실행
Task(subagent_type: "general-purpose", prompt: "codebase-researcher 에이전트로서 [파일/패턴]을 분석하라")
Task(subagent_type: "general-purpose", prompt: "framework-docs-researcher 에이전트로서 [기술] 문서를 조사하라")
```

---

## Workflow

### Phase 1: Issue Analysis
1. Understand the issue/requirement
2. Identify referenced files to learn from
3. Determine tech stack and versions
4. **Detect package manager:**
   ```bash
   if [ -f "bun.lockb" ]; then pm="bun"
   elif [ -f "pnpm-lock.yaml" ]; then pm="pnpm"
   elif [ -f "yarn.lock" ]; then pm="yarn"
   else pm="npm"; fi
   ```

### Phase 2: Parallel Research 🔀
```bash
# Run simultaneously using Task tool
Lane 1: codebase-researcher     # Analyze referenced files
Lane 2: framework-docs-researcher  # Framework best practice (optional)
```

### Phase 3: Plan Creation
1. **Plan 파일 생성** (기존 파일 덮어쓰기 금지):
   ```bash
   # 항상 새 파일에 작성. 기존 PLAN.md 절대 덮어쓰지 않는다.
   PLAN_FILE="PLAN-$(date +%Y%m%d-%H%M%S).md"
   ```
2. Extract patterns from referenced files
3. Define boundaries (Do/Don't)
4. Create implementation checklist
5. Define review focus areas

> **Note:** 프레임워크별 plan이 있으면 함께 참조 (`/angular:01_plan`, `/react:01_plan`, `/typescript:01_plan`)

---

## Output Structure

> ⚠️ 반드시 `PLAN-{timestamp}.md` 파일에 작성할 것. 기존 plan 파일을 덮어쓰지 않는다.

```markdown
### 📋 Implementation Plan: [Feature]

---

**Tech Stack** (check `package.json` for exact versions)
- [Framework]: [version from package.json]
- [State]: [version from package.json]
- [Build]: [tool]

---

**Knowledge Sources**
- `./CLAUDE.md`: [patterns found]
- `[reference file]`: [patterns extracted]
- External: [research findings]

---

**Boundaries**

✅ Do:
- [What this implementation SHOULD do]
- [Patterns to follow]

⚠️ Ask First:
- [Decisions requiring confirmation]
- [Trade-offs to discuss]

🚫 Don't:
- [What to avoid]
- [Anti-patterns for this feature]

---

**Files to Create/Modify**

| File | Action | Purpose |
|------|--------|---------|
| `path/to/file.ts` | Create | [Purpose] |
| `path/to/existing.ts` | Modify | [What changes] |

---

**Implementation Checklist** (→ 02_implement 단계에서 사용)

- [ ] [Step 1]
- [ ] [Step 2]
- [ ] [Feature-specific requirement]
- [ ] Named exports only
- [ ] Barrel exports (index.ts)
- [ ] Verification commands pass

---

**Verification Commands**

```bash
# {pm} = npm, yarn, pnpm, bun (use project's package manager)
{pm} run lint    # if available
{pm} test        # if available
```

---

**Review Focus** (→ 03_review 단계에서 확인)

- [ ] [This feature's critical check point]
- [ ] [Edge case to verify]
- [ ] [Performance consideration]

---

**Decisions**

| Choice | Options | Selected | Rationale |
|--------|---------|----------|-----------|
| [Decision] | A, B | A | [Why] |
```
