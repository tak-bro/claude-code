---
name: dependencies
description: "의존성 업데이트 및 보안 감사. Triggers on 'dependencies', '의존성', 'outdated', 'npm audit', 'update packages', '패키지 업데이트', 'vulnerability', 'security audit packages'."
tools: Read, Bash, Glob, Grep, Write, Edit
---

# Dependencies

의존성 상태를 점검하고 안전하게 업데이트한다.

---

## Phase 1: 현황 파악

```bash
# 패키지 매니저 감지
if [ -f "bun.lockb" ]; then pm="bun"
elif [ -f "pnpm-lock.yaml" ]; then pm="pnpm"
elif [ -f "yarn.lock" ]; then pm="yarn"
else pm="npm"; fi

# outdated 확인
$pm outdated

# 보안 취약점 확인
$pm audit
```

---

## Phase 2: 분류

| 구분 | 설명 | 예시 |
|------|------|------|
| **Patch** | 버그 수정. 안전 | 1.2.3 → 1.2.4 |
| **Minor** | 기능 추가. 대부분 안전 | 1.2.3 → 1.3.0 |
| **Major** | Breaking change. 주의 | 1.2.3 → 2.0.0 |
| **Security** | 취약점 수정. 우선 | audit에서 검출 |

### 업데이트 우선순위
```
Security (취약점) → 즉시
Patch (버그 수정) → 일괄 업데이트
Minor (기능 추가) → 테스트 후 업데이트
Major (breaking) → 별도 PR, migration guide 확인
```

---

## Phase 3: 실행

### Safe Updates (patch + minor)
```bash
# 일괄 업데이트
$pm update

# 또는 개별
$pm install {package}@latest
```

### Major Updates
```bash
# migration guide 확인 후 개별 업데이트
$pm install {package}@{version}
```

### 검증
```bash
$pm run lint
$pm test
$pm run build
```

---

## Phase 4: 보고

```markdown
## Dependencies Report

### Security Fixes
- {package}: {severity} — {description} (fixed: {version})

### Updated (Patch/Minor)
| Package | Before | After | Type |
|---------|--------|-------|------|
| {name} | {old} | {new} | patch/minor |

### Major Updates Pending
| Package | Current | Latest | Breaking Changes |
|---------|---------|--------|-----------------|
| {name} | {old} | {new} | {summary} |

### Skipped (with reason)
- {package}: {reason}

### Verification
- Lint: PASS/FAIL
- Test: PASS/FAIL
- Build: PASS/FAIL
```

---

## 주의사항

- Major 업데이트는 별도 PR로 분리
- lock 파일 변경은 반드시 커밋에 포함
- monorepo(Nx)의 경우 workspace 전체 호환성 확인
