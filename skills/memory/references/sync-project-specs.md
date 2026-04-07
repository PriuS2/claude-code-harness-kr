# Sync Project Specs Reference

**作業完了後に「Plans.md ちゃんと更新されているかなぁ」と不安消除時に実行します.**

## When to Use

| Situation | Command to Use |
|-----------|----------------|
| "How far along? What's next?" | `/sync-status` (먼저 이것부터) |
| "Worked on it but forgot if I updated Plans.md" | **이 명령** |
| "Started from old template, format might be outdated" | **이 명령** |

> Tip: Usually `/sync-status` 가 충분합니다. 이 명령은 "just in case" 또는 "format migration"용으로 사용하세요.

---

## Purpose

프로젝트 스펙/문서（예: `Plans.md`, `AGENTS.md`, `.claude/rules/*`）를 최신 claude-code-harness 작업（**PM ↔ Impl**, `pm:*` 마커, handoff 명령）と정렬합니다.

## VibeCoder Phrases

- "**Worked on it but unsure if Plans.md is updated**" → 이 명령
- "**Want to align old format files to latest**" → 마커와 설명을 통일
- "**Keep manual changes, fix only needed parts**" → 기존 텍스트를 유지하고 필요한 부분만 차이 적용

---

## Sync Targets (Existing Files Only)

- `Plans.md`
- `AGENTS.md`
- `CLAUDE.md` (작업 설명이 있는 경우만)
- `.claude/rules/workflow.md`
- `.claude/rules/plans-management.md`

---

## Sync Content (Minimal Diff Policy)

### 1. 마커 정규화

- **표준**: `pm:依頼中`, `pm:確認済`
- **호환**: `cursor:依頼中`, `cursor:確認済` (동의어로 취급)

### 2. 상태 전이 문서

```
pm:依頼中 → cc:WIP → cc:완료 → pm:確認済
```

### 3. Handoff 경로 추가

- PM→Impl: `/handoff-to-impl-claude` (PM Claude용)
- Impl→PM: `/handoff-to-pm-claude`
- Cursor 워크플로우: `/handoff-to-claude`, `/handoff-to-cursor`

### 4. 알림 파일 설명

- `.claude/state/pm-notification.md` (호환: `.claude/state/cursor-notification.md`)

---

## Execution Steps

### Step 1: 현재 상태 수집（필수）

- 대상 파일 존재 확인 및 관련 섹션 추출
- `Plans.md` 마커 발생 수 집계（pm/cursor/cc）

### Step 2: 변경 정책 선언（필수）

사용자에게 알립니다:
- 기존 텍스트는 원칙적으로 유지（파괴적 재작성 없음）
- 추가/교체는 "작업에 필요한 최소한으로" 제한
- 변경을 차분으로 표시하고, 필요하면 조정

### Step 3: 동기화（차분 적용）

- **Plans.md**: 마커凡例에 `pm:*`를 추가하고, `cursor:*`를 호환으로 표시
- **AGENTS.md**: 역할을 PM/Impl로 업데이트
- **rules/*.md**: `cursor:*`를 `pm:*` 표준으로 변경 + 호환 표기 추가
- **CLAUDE.md**: 작업 섹션이 있으면 PM↔Impl 경로를 추가

### Step 4: 완료（필수）

- `/sync-status`를 실행하여 마커 확인
- 필요하면 `/remember`를 사용하여 "프로젝트 고유 작업"을 잠금

---

## 병렬 실행

파일 읽기는 병렬화할 수 있습니다:

| Process | 병렬 |
|---------|---------|
| Plans.md 읽기 | ✅ 독립 |
| AGENTS.md 읽기 | ✅ 독립 |
| CLAUDE.md 읽기 | ✅ 독립 |
| rules/*.md 읽기 | ✅ 독립 |

업데이트는 일관성을 위해 순차 실행됩니다.
