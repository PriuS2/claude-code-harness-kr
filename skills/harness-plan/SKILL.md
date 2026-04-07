---
name: harness-plan
description: "Harness v3 통합 기획 스킬. 작업 계획·Plans.md관리·진행 동기화를 담당. 다음 구문으로起動: 계획 만들기, 작업 추가, Plans.md업데이트, 완료 표시, 진행 확인, harness-plan, harness-sync. 구현·리뷰·릴리스에는 사용하지 않음."
description-en: "Unified planning skill for Harness v3. Handles task planning, Plans.md management, and progress sync. Use when user mentions: create a plan, add tasks, update Plans.md, mark complete, check progress, sync status, where am I, harness-plan, harness-sync. Do NOT load for: implementation, code review, or release tasks."
description-ja: "Harness v3 統合プランニングスキル。タスク計画・Plans.md管理・進捗同期を担当。以下のフレーズで起動: 計画を作る、タスクを追加、Plans.md更新、完了マーク、進捗確認、harness-plan、harness-sync。実装・レビュー・リリースには使わない。"
allowed-tools: ["Read", "Write", "Edit", "Bash", "Grep", "Glob", "WebSearch", "Task"]
argument-hint: "[create|add|update|sync|sync --no-retro|--ci]"
effort: medium
---

# Harness Plan (v3)

Harness v3의 통합 기획 스킬입니다.
다음 3개의 기존 스킬을 통합합니다:

- `planning` (plan-with-agent) — 아이디어 → Plans.md 반영
- `plans-management` — 작업 상태 관리·마커 업데이트
- `sync-status` — Plans.md와 구현의 동기화 확인

## Quick Reference

| 사용자 입력 | 서브명령어 | 동작 |
|------------|------------|------|
| "계획 만들어줘" / "create a plan" | `create` | 대화형ヒアリング → Plans.md 생성 |
| "작업 추가해줘" / "add a task" | `add` | Plans.md에 새 작업 추가 |
| "완료로 표시해줘" / "mark complete" | `update` | 작업 마커를 cc:완료로 변경 |
| "지금 어디까지?" / "check progress" | `sync` | 구현과 Plans.md를 대조·동기화 |
| `harness-sync` | `sync` | 진행 확인（독립 sync surface와 동일） |
| `harness-plan create` | `create` | 계획 작성 |

## 서브명령어 상세

### create — 계획 작성

See [references/create.md](${CLAUDE_SKILL_DIR}/references/create.md)

아이디어·요구를ヒアリング하고 실행 가능한 Plans.md를 생성합니다.

**플로우**:
1. 대화 컨텍스트 확인（이전 논의에서 추출 또는 신규ヒアリング）
2. 무엇을 만들지 물어보기（최대 3문）
3. 기술 조사（WebSearch）
4. 기능 목록 추출
5. 우선순위 매트릭스（Required / Recommended / Optional）
6. TDD 채택 판단（테스트 설계）
7. Plans.md 생성（`cc:TODO` 마커 포함）
8. 다음 액션 안내

**CI 모드** (`--ci`):
ヒアリング 없음. 기존 Plans.md를 그대로 사용하여 작업 분해만 수행.

### add — 작업 추가

Plans.md에 새 작업을 추가합니다.

```
harness-plan add 작업명: 상세설명 [--phase 단계번호]
```

작업은 `cc:TODO` 마커로 추가됩니다.

### update — 마커 업데이트

작업의 상태 마커를 변경합니다.

```
harness-plan update [작업명|작업번호] [WIP|완료|blocked]
```

마커 대응표:

| 명령어 | 마커 |
|---------|---------|
| `WIP` | `cc:WIP` |
| `완료` / `done` | `cc:완료` |
| `blocked` | `blocked` |
| `TODO` | `cc:TODO` |

### sync — 진행 동기화

구현 상황과 Plans.md를 대조하고 차이를 검출·업데이트합니다.

See [references/sync.md](${CLAUDE_SKILL_DIR}/references/sync.md)

**플로우**:
1. Plans.md의 현황 취득
2. Plans.md 포맷 검출（v1: 3컬럼 / v2: 5컬럼）
3. git status / git log에서 구현 상황 취득
4. 에이전트 트레이스 확인（`.claude/state/agent-trace.jsonl`）
5. Plans.md와 구현의 차이 검출
6. 미갱신 마커의 자동 수정 제안
7. 다음 액션 제시

**레트로스펙티브**（기본 ON）:
`cc:완료` 작업이 1건 이상 있으면 자동으로振り返りを 실행합니다.
견적 정밀도, 블록 원인 패턴, 스코프 변동을 분석하고 학습을 기록합니다.
`sync --no-retro`로 명시적으로 건너뛰기 가능.

### team mode / issue bridge

Plans.md는 정본을 유지하며, GitHub Issue 연동은 opt-in인 team mode에서만 사용합니다.

- solo 개발에서는 bridge를 사용하지 않음
- team mode에서는 tracking issue를 1개 만들고, 그 하위에 작업별 sub-issue payload를 dry-run으로 생성
- `scripts/plans-issue-bridge.sh`는 실제로 GitHub를 업데이트하지 않고 항상 dry-run payload를 반환
- Plans.md에 대한 변경은 이 bridge에서는 수행하지 않음

참조:

- `docs/plans/team-mode.md`

## Plans.md 포맷 규정

### 포맷

```markdown
# [프로젝트명] Plans.md

작성일: YYYY-MM-DD

---

## Phase N: 단계명

| Task | 내용 | DoD | Depends | Status |
|------|------|-----|---------|--------|
| N.1  | 설명 | 테스트 통과 | - | cc:TODO |
| N.2  | 설명 | lint 에러 0 | N.1 | cc:WIP |
| N.3  | 설명 | 마이그레이션 실행 가능 | N.1, N.2 | cc:완료 |
```

**DoD（Definition of Done）**: 검증 가능한 완료 조건을 1행으로 기술. "잘 됐으면 좋겠어""제대로 작동해"는 금지. Yes/No로 판정 가능한 형태로 작성.

**Depends**: 작업 간의 의존 관계. `-`（의존 없음）、작업 번호（`N.1`）、쉼표 구분（`N.1, N.2`）、단계 의존（`Phase N`）。

### optional briefs / manifest

`harness-plan create`는 필요한 경우에만 brief를 붙입니다.

- UI를 포함한 작업에는 `design brief`
- API를 포함한 작업에는 `contract brief`
- brief는 "무엇을 만드는지"를 짧게 고정하는 보조 자료로, Plans.md를 대체하지 않음
- skill frontmatter 목록은 `scripts/generate-skill-manifest.sh`로 machine-readable JSON 생성 가능

참조:

- `docs/plans/briefs-manifest.md`

### 마커 목록

| 마커 | 의미 |
|---------|-------|
| `pm:依頼中` | PM에게 의뢰됨 |
| `cc:TODO` | 미착수 |
| `cc:WIP` | 작업중 |
| `cc:완료` | Worker 작업 완료 |
| `pm:確認済` | PM 리뷰 완료 |
| `blocked` | 블록중（사유 반드시 기재） |

## 관련 스킬

- `harness-sync` — 구현과 Plans.md를 동기화
- `harness-work` — 계획한 작업을 구현
- `harness-review` — 구현의 리뷰
- `harness-setup` — 프로젝트 초기화
