---
name: harness-review
description: "Harness v3 통합 리뷰 스킬. 코드·플랜·스코프를 다각적으로 리뷰. 다음으로 실행: 리뷰, 코드 리뷰, 플랜 리뷰, 스코프 분석, 보안, 품질 체크, harness-review. 구현·새기능·버그수정·세팅·릴리스에는 사용하지 않음."
description-en: "Unified review skill for Harness v3. Multi-angle code, plan, and scope review. Use when user mentions: review, code review, plan review, scope analysis, security, performance, quality checks, PRs, diffs, harness-review. Do NOT load for: implementation, new features, bug fixes, setup, or release."
description-ja: "Harness v3 통합 리뷰 스킬. 코드·플랜·스코프를 다각적으로 리뷰. 다음으로 실행: 리뷰, 코드 리뷰, 플랜 리뷰, 스코프 분석, 보안, 품질 체크, harness-review. 구현·새기능·버그수정·세팅·릴리스에는 사용하지 않음."
allowed-tools: ["Read", "Grep", "Glob", "Bash", "Task"]
argument-hint: "[code|plan|scope] [--dual] [--security]"
context: fork
effort: high
---

# Harness Review (v3)

Harness v3의 통합 리뷰 스킬입니다.
다음 기존 스킬을 통합합니다:

- `harness-review` — 코드·플랜·스코프 다각적 리뷰
- `codex-review` — Codex CLI에 의한 세컨드 오피니언
- `verify` — 빌드 검증·에러 복구·리뷰 수정 적용
- `troubleshoot` — 에러·장애의 진단과 수리

## Quick Reference

| 사용자 입력 | 서브명령어 | 동작 |
|------------|------------|------|
| "리뷰해줘" / "review" | `code`（자동） | 코드 리뷰（최근 변경） |
| "`harness-plan` 실행 후" | `plan`（자동） | 플랜 리뷰 |
| "스코프 확인" | `scope`（자동） | 스코프 분석 |
| `harness-review code` | `code` | 코드 리뷰 강제 |
| `harness-review plan` | `plan` | 플랜 리뷰 강제 |
| `harness-review scope` | `scope` | 스코프 분석 강제 |
| `harness-review --dual` | `code`（자동） + Codex 병렬 | Claude + Codex dual review |
| `harness-review --security` | Security Review | OWASP Top 10 전용 보안 리뷰（read-only） |

## 옵션

| 옵션 | 기본값 | 설명 |
|-----------|-----------|------|
| `--dual` | 없음 | Claude Reviewer와 Codex Reviewer를 병렬 실행하고 verdict를 머지. Codex 불가 시 자동 폴백. 상세: [`${CLAUDE_SKILL_DIR}/references/dual-review.md`](${CLAUDE_SKILL_DIR}/references/dual-review.md) |
| `--security` | 없음 | OWASP Top 10 기반의 보안 전용 리뷰를 실행. read-only（Write/Edit/Bash 쓰기 불가）. 상세: [`${CLAUDE_SKILL_DIR}/references/security-profile.md`](${CLAUDE_SKILL_DIR}/references/security-profile.md) |
| `--no-commit` | 없음 | APPROVE 시의 자동 커밋을 무효화 |

## 리뷰 타입 자동 판단

| 직전 활동 | 리뷰 타입 | 관점 |
|--------------------|--------------|------|
| `harness-work` 후 | **Code Review** | Security, Performance, Quality, Accessibility, AI Residuals |
| `harness-plan` 후 | **Plan Review** | Clarity, Feasibility, Dependencies, Acceptance |
| 작업 추가 후 | **Scope Review** | Scope-creep, Priority, Feasibility, Impact |

## Code Review 플로우

### Step 1:변경 차분을 수집

```bash
# BASE_REF가 harness-work에서 전달된 경우는 그것을 사용, 없으면 HEAD~1로 폴백
CHANGED_FILES="$(git diff --name-only --diff-filter=ACMR "${BASE_REF:-HEAD~1}")"
git diff ${BASE_REF:-HEAD~1} --stat
git diff ${BASE_REF:-HEAD~1} -- ${CHANGED_FILES}
```

### Step 1.5: AI Residuals를 정적 스캔

LLM의 인상만으로 판단하지 말고, 재실행할 수 있는 형태로 잔해를 검출합니다. `scripts/review-ai-residuals.sh`는 stable한 JSON을 반환하므로, 그 결과를 리뷰 근거로 사용합니다.

```bash
# 차분 기반
AI_RESIDUALS_JSON="$(bash scripts/review-ai-residuals.sh --base-ref "${BASE_REF:-HEAD~1}")"

# 대상 파일을 명시하고 싶은 경우
bash scripts/review-ai-residuals.sh path/to/file.ts path/to/config.sh
```

### Step 2: 5관점으로 리뷰

| 관점 | 체크 내용 |
|------|------------|
| **Security** | SQL인젝션, XSS, 기밀 정보 노출, 입력 검증 |
| **Performance** | N+1 쿼리, 불필요한 렌더링, 메모리 누수 |
| **Quality** |명명, 단일 책임, 테스트 커버리지, 에러 핸들링 |
| **Accessibility** | ARIA 속성, 키보드 나비, 컬러 대비 |
| **AI Residuals** | `mockData`, `dummy`, `fake`, `localhost`, `TODO`, `FIXME`, `it.skip`, `describe.skip`, `test.skip`, 하드코딩된 기밀 정보/환경 의존 URL, 명확한 임시 구현 코멘트 |

### Step 2.2: AI Residuals의 severity 판단표

`AI Residuals`는 우선 `scripts/review-ai-residuals.sh`의 JSON을 확인하고, 그 후에 diff 문맥에서「정말 출하 리스크인지」를 최종 판단합니다.

| 중요도 | 대표 예 | 판단의 고려 |
|--------|--------|-------------|
| **major** | `localhost` / `127.0.0.1` / `0.0.0.0`의 연결 대상, `it.skip` / `describe.skip` / `test.skip`, 하드코딩된 기밀 정보스러운 값, dev/staging 고정 URL |본 장애, 잘못된 설정, 검증 누락에 연결되기 쉬운. 1건이라도 `REQUEST_CHANGES` |
| **minor** | `mockData`, `dummy`, `fakeData`, `TODO`, `FIXME` | 잔해의 가능성은 높지만, 즉 사고로 이어지지는 않음.수정 권장인 verdict는 변하지 않음 |
| **recommendation** | `temporary implementation`, `replace later`, `placeholder implementation` 같은 임시 구현 코멘트 | 코멘트만으로 즉 버그 단정할 수 없지만, 추적·명확화를 촉진하고 싶다 |

### Step 2.5:임계값 기준에 의한 verdict 판단

각 지적을 이하의 중요도로 분류하고 **이 기준만으로** verdict를 결정합니다.

| 중요도 | 정의 | verdict 영향 |
|--------|------|-----------------|
| **critical** | 보안 취약성, 데이터 손실 리스크, 본 장애의 가능성 | 1건이라도 → REQUEST_CHANGES |
| **major** |기존 기능，파괴，스펙과의모순，테스트 불통과 | 1건이라도 → REQUEST_CHANGES |
| **minor** |명명 개선，댓글 부족，스타일 불 통일 | verdict에 영향하지 않음 |
| **recommendation** | 베스트 프랙티스 제안, 향후 개선안 | verdict에 영향하지 않음 |

> **중요**: minor / recommendation만 있는 경우는 **반드시 APPROVE를 반환할 것**.
> "있으면 좋은 개선"은 REQUEST_CHANGES의 이유가 되지 않음.
> `AI Residuals`도 마찬가지. `major`에 들어가는 것은「출하 사고나잘못된 설정에 연결되기 쉬운」만으로, 단순한 잔해 후보는 `minor` 또는 `recommendation`에 머무릅니다.

### Step 3: 리뷰 결과 출력

```json
{
  "schema_version": "review-result.v1",
  "verdict": "APPROVE | REQUEST_CHANGES",
  "reviewer_profile": "static | runtime | browser",
  "calibration": {
    "label": "false_positive | false_negative | missed_bug | overstrict_rule",
    "source": "manual | post-review | retrospective",
    "notes": "관찰 메모",
    "prompt_hint": "few-shot에 사용할 요점",
    "few_shot_ready": true
  },
  "critical_issues": [],
  "major_issues": [],
  "observations": [
    {
      "severity": "critical | major | minor | recommendation",
      "category": "Security | Performance | Quality | Accessibility | AI Residuals",
      "location": "파일명:행번호",
      "issue": "문제의 설명",
      "suggestion": "수정안"
    }
  ],
  "recommendations": ["필수는 아닌 개선 제안"]
}
```

browser review의 경우 `scripts/generate-browser-review-artifact.sh`가 `browser_mode`와 route / required artifacts를 결정하고, 그 후에 `scripts/write-review-result.sh`로 `.claude/state/review-result.json`에 정규화하여 저장합니다.
이 파일은 commit guard와 후속 플로우의 공통 입력이 됩니다.
`calibration`이 붙은 리뷰 결과는 `scripts/record-review-calibration.sh`로
`.claude/state/review-calibration.jsonl`에 기록하고, `scripts/build-review-few-shot-bank.sh`로 few-shot bank를 업데이트합니다.

### Step 3.5: --dual 플래그시의 Codex 병렬 리뷰

`--dual` 플래그가 지정된 경우, Step 3의 Claude 리뷰와 병렬하여 Codex 리뷰를 실행하고, 결과를 머지합니다.

1. Codex의 사용 가능 여부를 확인（`scripts/codex-companion.sh setup --json`）
2. 사용 가능하다면 `scripts/codex-companion.sh review --base "${BASE_REF:-HEAD~1}"`를 실행
3. 양쪽의 verdict를 Verdict 머지 규칙으로 통합
4. 최종 리뷰 결과에 `dual_review` 필드를 부착

자세한 절차·출력 스키마·폴백 사양은 [`${CLAUDE_SKILL_DIR}/references/dual-review.md`](${CLAUDE_SKILL_DIR}/references/dual-review.md)를 참조.

### Step 3.6: --security 플래그시의 보안 전용 리뷰

`--security` 플래그가 지정된 경우, 일반적인 5관점 리뷰를 **스킵**하고, 보안 전용 플로우를 실행합니다.

**Read-only 제약**: 이 플로우 중은 Write / Edit / 쓰기 계열 Bash를 전적 실행하지하지 않습니다.

1. 보안 프로필을 읽음:
   ```
   Read: ${CLAUDE_SKILL_DIR}/references/security-profile.md
   ```
2. OWASP Top 10 전 카테고리를변경 차분·관련 파일에 대해 확인
3. 인증·인가 플로우, 기밀 정보 다루기, 의존 패키지 취약성을 체크
4. `reviewer_profile: "security"`를 설정하여 결과를 출력（Step 3의 JSON 스키마에 준수）
5. Security 모드의 verdict 판단 기준（security-profile.md끝 참조）을 적용

일반적인 Code Review와 `--security`의 사용 구분:

| | 일반적인 Code Review | `--security` |
|---|---|---|
| 관점 | Security, Performance, Quality, Accessibility, AI Residuals | Security만（OWASP Top 10 전체 항목） |
| 심층 | 보안은 개요 체크 | 인증·인가·암호화·의존 관계까지 포괄 |
| 도구 제약 | 없음 | Read / Grep / Glob / 읽기 Bash만 |
| 용도 | PR 머지 전 종합 확인 | 보안 집중 감사·릴리스 전 추가 확인 |

### Step 4: 커밋 판단

- **APPROVE**: 자동 커밋 실행（`--no-commit`이 아니면）
- **REQUEST_CHANGES**: critical/major의 지적 사항과 수정 방안을 제시. `harness-work`의 수정 루프로 자동 수정 후 재리뷰（최대 3회）

## Plan Review 플로우

1. Plans.md를 읽음
2. 이하의 **5관점**으로 리뷰:
   - **Clarity**: 작업 설명이 명확한지
   - **Feasibility**: 기술적으로 실현 가능한지
   - **Dependencies**: 작업 간의 의존 관계가 올바른지（Depends 컬럼과실제의 의존이 일치하고 있는지）
   - **Acceptance**: 완료 조건（DoD 컬럼）이 정의되어 있고 검증 가능한지
   - **Value**: 이 작업은 사용자 과제를 해결하는지?
     - "누의, 어떤 문제"가 명시되어 있는지
     - 대안 수단（만들지 않는 선택）검토되었는가
     - Elephant（전원이 인지하고 있는 문제) 있는지
3. DoD / Depends 컬럼의 품질 체크:
   - DoD가 빈칸인 작업 → 경고（「완료 조건이 미정의입니다」）
   - DoD가 검증 불가능（「잘 됐으면 좋겠어」「제대로 작동해」등） → 경고 + 구체화 제안
   - Depends에 존재하지 않는 작업 번호 → 에러
   - 순환 의존 → 에러
4. 개선 제안을 제시

## Scope Review 플로우

1. 추가된 작업/기능을 목록화
2. 이하의 관점에서 분석:
   - **Scope-creep**: 당초 스코프으로부터의 이탈
   - **Priority**: 우선순위가 올바른지
   - **Feasibility**: 현재 리소스로 실현 가능한지
   - **Impact**:기존 기능의 영향
3. 리스크와 권장 액션을 제시

## 이상 검출

| 상황 | 액션 |
|------|----------|
| 보안 취약성 | 즉시 REQUEST_CHANGES |
| 테스트 변조 의심 | 경고 + 수정 요구 |
| force push 시도 | 거부 + 대안 제시 |

## Codex Environment

Codex CLI 환경（`CODEX_CLI=1`）에서는 일부 도구가 사용 불가로 인해, 이하의 폴백를 사용합니다.

| 일반 환경 | Codex 폴백 |
|---------|-------------------|
| `TaskList`로 작업 목록 취득 | Plans.md를 `Read`하여 WIP/TODO 작업을 확인 |
| `TaskUpdate`로 상태 업데이트 | Plans.md의 마커를 `Edit`로 직접 업데이트（예: `cc:WIP` → `cc:완료`） |
| 리뷰 결과를 Task에 기록 | 리뷰 결과를 stdout에 출력 |

### 검출 방법

```bash
if [ "${CODEX_CLI:-}" = "1" ]; then
  # Codex 환경: Plans.md 기반의 폴백
fi
```

### Codex 환경에서의 리뷰 출력

Task tool 미지원의 때문, 리뷰 결과는 표준 출력에 마크다운 형식으로 출력합니다.
Lead 에이전트 또는 사용자가 결과를 읽고, 다음 액션을 판단합니다.

## 관련 스킬

- `harness-work` — 리뷰 후 수정 구현
- `harness-plan` — 계획 작성·수정
- `harness-release` — 리뷰 통과 후 릴리스
