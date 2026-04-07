---
name: ci
description: "CI가 빨개지면 호출해주세요. 파이프라인 소화대가 출동합니다. Use when user mentions CI failures, build errors, test failures, or pipeline issues. Do NOT load for: local builds, standard implementation work, reviews, or setup."
description-en: "CI red? Call us. Pipeline fire brigade deploys. Use when user mentions CI failures, build errors, test failures, or pipeline issues. Do NOT load for: local builds, standard implementation work, reviews, or setup."
description-ja: "CI가 빨개지면 호출해주세요. 파이프라인 소화대가 출동합니다. Use when user mentions CI failures, build errors, test failures, or pipeline issues. Do NOT load for: local builds, standard implementation work, reviews, or setup."
allowed-tools: ["Read", "Grep", "Bash", "Task"]
user-invocable: false
context: fork
argument-hint: "[analyze|fix|run]"
---

# CI/CD Skills

CI/CD 파이프라인 관련 문제를 해결하는 스킬 그룹입니다.

---

## 발동 조건

- "CI가 떨어졌다", "GitHub Actions가 실패했다"
- "빌드 에러", "테스트가 통과하지 않는다"
- "파이프라인 고쳐줘"

---

## 기능 상세

| 기능 | 상세 | 트리거 |
|------|------|----------|
| **실패 분석** | See [references/analyzing-failures.md](${CLAUDE_SKILL_DIR}/references/analyzing-failures.md) | "로그 봐줘", "원인 조사해줘" |
| **테스트 수정** | See [references/fixing-tests.md](${CLAUDE_SKILL_DIR}/references/fixing-tests.md) | "테스트 고쳐줘", "수정案的 보여줘" |

---

## 실행 절차

1. **테스트 vs 구현 판단** (Step 0)
2. 사용자의 의도 분류 (분석 or 수정)
3. 복잡도 판단 (아래 참조)
4. 위의 "기능 상세"에서 적절한 참조 파일을 읽거나, ci-cd-fixer 서브에이전트 기동
5. 결과 확인하고 필요시 재실행

### Step 0: 테스트 vs 구현 판단 (품질 판단 게이트)

CI 실패시, 먼저 원인 파악을 구분합니다:

```
CI 실패 보고
    ↓
┌─────────────────────────────────────────┐
│           테스트 vs 구현 판단             │
├─────────────────────────────────────────┤
│  에러의 원인을 분석:                    │
│  ├── 구현이 잘못됨 → 구현 수정          │
│  ├── 테스트가 오래됨 → 사용자에게 확인  │
│  └── 환경 문제 → 환경 수정                │
└─────────────────────────────────────────┘
```

#### 금지 사항 (tampering 방지)

```markdown
⚠️ CI 실패시의 금지 사항

以下の「解決策」は禁止です：

| 금지 | 예 | 올바른 대응 |
|------|-----|-----------|
| 테스트 skip 화 | `it.skip(...)` | 구현 수정 |
| 어설션 삭제 | `expect()` 지움 | 기대값 확인 |
| CI 체크 우회 | `continue-on-error` | 근본 원인 수정 |
| lint 규칙 완화 | `eslint-disable` | 코드 수정 |
```

#### 판단 흐름

```markdown
🔴 CI가 실패하고 있습니다

**판단이 필요합니다**:

1. **구현이 잘못됨** → 구현 수정 ✅
2. **테스트의 기대값이 오래남** → 사용자에게 확인 요청
3. **환경의 문제** → 환경 설정 수정

⚠️ 테스트의 tampering (skip화, 어설션 삭제)은 금지입니다

어떤에 해당합니까?
```

#### 승인이 필요한 경우

테스트/설정 변경이 불가피한 경우:

```markdown
## 🚨 테스트/설정 변경의 승인 요청

### 이유
[왜 이 변경이 필요한지]

### 변경 내용
[차이]

### 대안 검토
- [ ] 구현 수정이 해결되지 않는지 확인함

사용자의 명시적인 승인을 기다립니다
```

### Git log 확장 플래그 활용 (CC 2.1.49+)

CI 실패시의 원인 커밋 특정에 구조화 로그를 활용합니다.

#### 원인 커밋의 특정

```bash
# 구조화된 포맷으로 커밋 분석
git log --format="%h|%s|%an|%ad" --date=short -10

# 토폴로지 순서로 시계열 분석
git log --topo-order --oneline -20

# 변경 파일과 원인의 연결
git log --raw --oneline -5
```

####主な活用場面

| 용도 | 플래그 | 효과 |
|------|--------|------|
| **실패 원인 특정** | `--format="%h|%s"` | 커밋 목록의 구조화 |
| **시계열 추적** | `--topo-order` | 머지 순서를 고려한 추적 |
| **변경 영향 파악** | `--raw` | 파일 변경의 상세 표시 |
| **머지 제외 분석** | `--cherry-pick --no-merges` | 실 커밋만 추출 |

#### 出力例

```markdown
🔍 CI 실패 원인 분석

최근 커밋（구조화）:
| Hash | Subject | Author | Date |
|------|---------|--------|------|
| a1b2c3d | feat: update API | Alice | 2026-02-04 |
| e4f5g6h | test: add tests | Bob | 2026-02-03 |

변경 파일（--raw）:
├── src/api/endpoint.ts (Modified) ← 型エラー発生
├── tests/api.test.ts (Modified)
└── package.json (Modified)

→ a1b2c3d 의 커밋이 원인 가능성大
  型エラー: src/api/endpoint.ts:42
```

## 서브에이전트 연동

다음 조건을 만족하는 경우, Task tool로 ci-cd-fixer를 기동:

- 수정 → 재실행 → 실패의 루프가 **2회 이상** 발생
- 또는 에러가 복수 파일에 걸친 복잡한 케이스

**기동 패턴:**

```
Task tool:
  subagent_type="ci-cd-fixer"
  prompt="CI실패를 진단・수정해주세요. 에러 로그: {error_log}"
```

ci-cd-fixer는 안전 제일로 동작 (기본 dry-run 모드).
세부사항은 `agents/ci-cd-fixer.md`를 참조.

---

## VibeCoder 向け

```markdown
🔧 CI가 고장났을 때의 말법

1. **「CI가 떨어졌다」「빨개졌다」**
   - 자동 테스트가 실패한 상태

2. **「왜 실패한 거야?」**
   - 원인을 조사해줘

3. **「고쳐줘」**
   - 자동으로 수정을 시도

💡 중요: 테스트를「속이는」수정은 금지입니다
   - ❌ 테스트를 없애거나, 스킵하는 것
   - ⭕ 코드를 올바르게 고치는 것

「테스트가 잘못된 것 같다」생각하면,
먼저 확인하고 대응을 결정하자
```
