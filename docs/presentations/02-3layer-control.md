# AI 3-Layer 제어 — "하지마"보다 코드로 막기

## 핵심 메시지
프롬프트에 "절대 하지마"라고 써봤자 AI는 가끔 무시한다. 구조적으로 막아야 한다.

---

## 문제: 프롬프트만으로는 부족

```
CLAUDE.md: "main 브랜치에 직접 push 하지마"
AI: (긴 작업 중 까먹고) git push origin main
```

프롬프트는 가이드라인이지 강제가 아니다.
컨텍스트가 길어지면 초반 지시를 잊는다.

---

## 해결: 3-Layer 제어 모델

```
Layer 1: settings.json deny     ← 하드 블록 (절대 실행 불가)
Layer 2: hooks exit 2           ← 조건부 차단 (상황에 따라 차단)
Layer 3: CLAUDE.md              ← 소프트 가이드 (AI가 참고)
```

위에서 아래로 갈수록 강제력이 약해진다.

---

## Layer 1: settings.json deny (하드 블록)

특정 명령 자체를 실행 불가능하게 막음.

```json
{
  "deny": [
    "rm -rf",
    "git push --force"
  ]
}
```

AI가 아무리 실행하려 해도 시스템 레벨에서 차단. 프롬프트 무시와 무관.

---

## Layer 2: hooks exit 2 (조건부 차단)

"특정 조건일 때만" 차단. 코드로 로직 작성.

```bash
# 보호 브랜치에서 파일 수정 시 차단
BRANCH=$(git rev-parse --abbrev-ref HEAD)
if [ "$BRANCH" = "main" ] || [ "$BRANCH" = "develop" ]; then
  echo "[차단] 보호 브랜치에서 파일 수정 불가" >&2
  exit 2  # exit 2 = AI에게 차단 사실 전달
fi
```

- `exit 0`: 허용
- `exit 2`: 차단 + AI에게 이유 전달 (AI가 다른 방법 시도)
- 다른 exit code: 차단 + 에러

실제 사용 예시:
- 보호 브랜치 수정 차단
- 커밋 메시지 포맷 강제
- 특정 파일 패턴 수정 금지

---

## Layer 3: CLAUDE.md (소프트 가이드)

AI가 매 세션 로드하는 지침서. 강제력은 없지만 대부분 잘 따름.

```markdown
## Code Style
- Named exports only (no export default)
- const + arrow functions
- Function body: max 80 lines
```

적합한 것: 코드 스타일, 워크플로우 가이드, 컨벤션
부적합한 것: 절대 하면 안 되는 것 (Layer 1, 2로 올려야 함)

---

## 실전 판단 기준

| 상황 | Layer | 이유 |
|------|-------|------|
| main push 방지 | Layer 2 (hook) | 브랜치 조건 판단 필요 |
| rm -rf 방지 | Layer 1 (deny) | 무조건 차단 |
| 코드 스타일 강제 | Layer 3 (CLAUDE.md) | 가이드라인 수준 |
| 커밋 메시지 포맷 | Layer 2 (hook) | 포맷 검증 로직 필요 |
| force push 방지 | Layer 1 (deny) | 무조건 차단 |
| 파일 80줄 제한 | Layer 3 (CLAUDE.md) | 유연하게 적용 |

---

## 핵심 원리

1. **중요도에 따라 레이어를 올린다** — "하면 안 되는 것"은 deny/hook, "하지 않는 게 좋은 것"은 CLAUDE.md
2. **hook은 exit 2를 쓴다** — AI에게 차단 이유를 알려줘야 대안을 찾음
3. **CLAUDE.md는 200줄 이하** — 길어지면 AI가 뒷부분을 무시

---

## 한줄 요약
> AI를 프롬프트로 제어하려 하지 마라. deny로 막고, hook으로 조건 걸고, CLAUDE.md는 가이드만.
