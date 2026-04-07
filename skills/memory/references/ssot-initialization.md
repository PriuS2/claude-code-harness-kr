---
name: init-memory-ssot
description: "프로젝트의 SSOT 메모리（decisions/patterns）와任意のsession-log을 초기화합니다.初回セットアップ어나, .claude/memory가 未整備のプロジェクトで使用します."
allowed-tools: ["Read", "Write"]
---

# Init Memory SSOT

`.claude/memory/`以下の **SSOT** 를 초기화합니다.

- `decisions.md`（중요한 의사결정의SSOT）
- `patterns.md`（재利用可能な解法のSSOT）
- `session-log.md`（세션 로그. 로컬運用推奨）

상세方针: `docs/MEMORY_POLICY.md`

---

## 실행 절차

### Step 1: 기존 파일의 확인

- `.claude/memory/decisions.md`
- `.claude/memory/patterns.md`
- `.claude/memory/session-log.md`

존재하는 것은**上書き하지 않음**.

### Step 2: 템플릿에서 초기화（존재하지 않는 경우만）

템플릿:

- `templates/memory/decisions.md.template`
- `templates/memory/patterns.md.template`
- `templates/memory/session-log.md.template`

`{{DATE}}`는当日（例: `2025-12-13`）で置換して生成します.

### Step 3: 완료 보고

- 작성한 파일 목록
- Git방침（`decisions/patterns`는 공유推奨, `session-log/.claude/state`는 로컬推奨）
