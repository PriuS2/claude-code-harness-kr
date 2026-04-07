---
name: cc-update-review
description: "CC アプデ統合の品質ガードレール。Feature Table 追加時に「書だけではない」を検出し、実装案を強制出力。Use when reviewing CC update integration PRs. Do NOT load for: implementation work, standard reviews, setup."
description-en: "Quality guardrail for CC update integration. Detects doc-only Feature Table additions and requires implementation proposals. Internal use only."
description-ja: "CC アプデ統合の品質ガードレール。Feature Table 追加時に「書だけではない」を検出し、実装案を強制出力。内部専用。"
user-invocable: false
allowed-tools: ["Read", "Grep", "Glob"]
---

# CC Update Review 가드레일

Claude Code 업데이트 통합 시 "Feature Table에 쓴 것만"을 방지하는 품질 가드레일.
Feature Table 추가가 구현을 수반하는지를 자동 분류하고, 부족분이 있으면 구현안을 강제 출력.

## Quick Reference

이 스킬이 트리거되는 상황:

- **CC 업데이트 통합 PR** 검토 시
- **Feature Table** (`CLAUDE.md` / `docs/CLAUDE-feature-table.md`)에 새 행이 추가된 diff를検出した時
- `/harness-review`가 CC 통합 PR로 판단한 경우의 내부 호출

트리거 **하지 않는** 상황:

- 일반적인 구현 작업 (`/work`)
- Feature Table 외의 변경만 있는 경우
- 설정・초기화 작업

## 3 카테고리 분류

Feature Table에 추가된 각 항목을 다음 3 카테고리로 분류.

### (A) 구현 있음

**정의**: Feature Table 추가에 대응하는 hooks / scripts / agents / skills / core 의 구현 변경이同一 PR에 포함.

**판정 조건**:
- Feature Table의 행에서 언급된 기능과 관련된 파일이 변경됨
- hooks.json, 스킬 SKILL.md, 에이전트 .md, scripts/*.sh, core/src/*.ts 중 어느 것이라도 diff가 있음

**예**:

| Feature Table 추가 | 대응하는 구현 변경 | 판정 |
|-------------------|----------------|------|
| `PostCompact 훅` | `hooks/post-compact-handler.sh` 신규 작성 | A |
| `MCP Elicitation 대응` | `hooks.json`에 Elicitation 이벤트 추가 + `elicitation-handler.sh` 작성 | A |
| `Worker maxTurns 제한` | `agents-v3/worker.md`에 maxTurns 필드 추가 | A |

**결과**: OK. 추가 액션 불필요.

---

### (B) 쓴 것만

**정의**: Feature Table에만 행이 추가되고, Harness 측 구현 변경이 전혀 포함되지 않음. 또한 CC 자동 상속(카테고리 C)에도 해당하지 않음.

**판정 조건**:
- Feature Table에 새 행이 있음
-同一 PR 내에서 hooks / scripts / agents / skills / core 관련 변경이 없음
- Harness가 고유한 부가 가치를 제공해야 하는 기능임 (설정, 워크플로우 통합, 가드레일 등)

**예**:

| Feature Table 추가 | 대응하는 구현 변경 | 판정 |
|-------------------|----------------|------|
| `PreCompact 훅` | 없음 (Feature Table만) | B |
| `Agent Teams` | 없음 (Feature Table만) | B |
| `Desktop Scheduled Tasks` | 없음 (Feature Table만) | B |

**결과**: NG. PR을 블록하고, 구현안 제시를 요구. 출력 포맷은 후술.

---

### (C) CC 자동 상속

**정의**: Claude Code 본체의 성능 개선・버그 수정・내부 최적화 등으로 Harness 측 변경이 불필요한 항목.

**판정 조건**:
- CC 본체의 수정이며, Harness가 래핑・확장할 여지가 없음
- 성능 개선, 메모리 누수 수정, UI 개선 등
- Harness의 워크플로우에 영향을 주지 않는 내부 변경

**예**:

| Feature Table 추가 | 이유 | 판정 |
|-------------------|------|------|
| `Streaming API memory leak fix` | CC 내부의 메모리 누수 수정. Harness 측 대응 불필요 | C |
| `Compaction image retention` | CC가 컴팩션 시 이미지를 유지. Harness 변경 불필요 | C |
| `Parallel tool call fix` | CC 내부의 병렬 실행 수정. 자동으로 혜택을 받음 | C |

**결과**: OK. 단, Feature Table의 칼럼에 "CC 자동 상속"을 명기할 것.

## CC 업데이트 PR 체크리스트

PR 검토 시 아래를 순서대로 확인:

```
## CC 업데이트 통합 체크리스트

### 1. Feature Table 차분 추출
- [ ] `CLAUDE.md` 또는 `docs/CLAUDE-feature-table.md`의 diff에서 추가 행을 열거

### 2. 각 항목의 분류
- [ ] 추가된 각 행에 대해 A / B / C 판정
- [ ] 카테고리 B의 항목이 0건인지 확인

### 3. 카테고리별 확인
- [ ] (A) 구현 있음: 대응하는 구현 파일이 올바르게 링크되어 있는지
- [ ] (B) 쓴 것만: 구현안이 제시되어 있는지 (0건이 아니면 PR 블록)
- [ ] (C) CC 자동 상속: Feature Table에 "CC 자동 상속" 명기가 있는지

### 4. CHANGELOG 확인
- [ ] 카테고리 A의 항목이 CHANGELOG에 "지금까지 / 향후" 형식으로 기재되어 있는지
- [ ] 카테고리 C의 항목이 CHANGELOG에서 CC 자동 상속으로 기재되어 있는지

### 분류 결과

| # | Feature Table 항목 | 카테고리 | 대응 파일 / 비고 |
|---|-------------------|---------|-------------------|
| 1 | (항목명) | A / B / C | (파일 경로 또는 비고) |
| 2 | (항목명) | A / B / C | (파일 경로 또는 비고) |
```

## 카테고리 B 检测 시 출력 포맷

카테고리 B가 1건 이상 检测された場合, 아래 포맷으로 구현안 출력.
**이 포맷의 출력은 필수이며, 생략은 허용되지 않음.**

```
## 카테고리 B 检测: 구현안

### B-{번호}. {Feature Table 항목명}

**현황**: Feature Table에 기재のみ. Harness 측 구현 없음.

**Harness만이 제공하는 부가 가치**:
{이 기능을 Harness가 어떻게 활용해야 하는지의 구체적 설명}

**구현안**:

| 대상 파일 | 변경 내용 |
|------------|---------|
| `{파일 경로}` | {구체적인 변경 내용} |
| `{파일 경로}` | {구체적인 변경 내용} |

**사용자 경험의 개선**:
- 지금까지: {현재 사용자 경험}
- 향후: {구현 후 사용자 경험}

**구현 우선순위**: {높음 / 중간 / 낮음}
**추정 공수**: {소 / 중 / 대}
```

### 출력 예

```
## 카테고리 B 检测: 구현안

### B-1. Desktop Scheduled Tasks

**현황**: Feature Table에 기재のみ. Harness 측 구현 없음.

**Harness만이 제공하는 부가 가치**:
Scheduled Tasks를 Harness의 워크플로우와 통합하여 정기적인 품질 체크 ·
상태 동기화 · 메모리 정리를 자동화할 수 있다.

**구현안**:

| 대상 파일 | 변경 내용 |
|------------|---------|
| `skills/harness-work/references/scheduled-tasks.md` | 스케줄 태스크의 템플릿과 가이드 |
| `scripts/setup-scheduled-tasks.sh` | 초기 설정 스크립트 |
| `hooks/hooks.json` | Cron 트리거 등록 |

**사용자 경험의 개선**:
- 지금까지: 사용자가 수동으로 정기 작업을 실행할 필요가 있었다
- 향후: Harness가 자동으로 정기 품질 체크를 실행하고 결과를 통지한다

**구현 우선순위**: 중간
**추정 공수**: 중간
```

## 「부가 가치」열 권장

Feature Table에 다음 칼럼 추가를 권장:

| Feature | Skill | Purpose | 부가 가치 |
|---------|-------|---------|-----------|
| PostCompact 훅 | hooks | 컨텍스트 재주입 | A: 구현 있음 |
| Streaming leak fix | all | 메모리 누수 수정 | C: CC 자동 상속 |

이 열により 각 항목의 분류が一目で確認でき, 카테고리 B의 잔류를防止할 수 있다.

## 관련 스킬

- `harness-review` - 코드レビュー（CC 통합 PR 판정 시 이 스킬을 내부 호출）
- `harness-work` - 구현 작업（카테고리 B의 구현안에 따른 작업 시）
- `memory` - SSOT 관리（분류 기준 결정 기록）
