---
name: migrate-workflow-files
description: "既存プロジェクトのAGENTS.md/CLAUDE.md/Plans.mdを、既存内容を精査して対話で引き継ぎ項目を確定しつつ、新フォーマットへ移行(バックアップ付き·Plansは作業保持マージ)."
allowed-tools: ["Read", "Write", "Edit", "Bash"]
---

# Migrate Workflow Files (Interactive Merge)

## 목적

기존 프로젝트에서運用中の以下を、**기존 내용을 존중하면서新フォーマットへアップデート**합니다.

- `AGENTS.md`
- `CLAUDE.md`
- `Plans.md`

포인트:

- **대화 형식으로引き継ぎ情報を確定**（勝手に捨てない / 勝手に上書きしない）
- 변경前に **반드시 백업** 을 남김
- `Plans.md`는 `merge-plans`의方针으로 **작업을 유지하면서 구조를 업데이트**

---

## 전제（重要）

이 스킬은「初回適用時の安全」과「의도한 동작（新フォーマット）」의両立のため,
**사용자 합의→백업→생성→차분 확인**의 순서로 진행합니다.

---

## 입력（이 스킬 내에서 자동 검출로OK）

- `project_name`: `basename $(pwd)`로 추정
- `date`: `YYYY-MM-DD`
- 기존 파일의 유무:
  - `AGENTS.md`
  - `CLAUDE.md`
  - `Plans.md`
- 新フォーマットの参照テンプレ:
  - `templates/AGENTS.md.template`
  - `templates/CLAUDE.md.template`
  - `templates/Plans.md.template`

---

## 실행 플로우

### Step 0: 검출와 합의 취득（필수）

1. `Read`로 기존 `AGENTS.md` / `CLAUDE.md` / `Plans.md`의 존재를 확인.
2. 존재하는 경우 사용자에게 확인:
   - **마이그레이션（新フォーマットへアップデート）してよいか**
   - 重要: 마이그레이션은 ** 내용의 재정리を含む**（= 정도의 배치替え나 표현 변경이 일어날 수 있음）

사용자가 NO의 경우:

- 이 스킬은 중지（아무것도 쓰지 않음）
- 대안으로「`.claude/settings.json`의 안전 머지만」等の安全工作を提案

### Step 1: 기존 내용의 심사（요약）

각 파일을 `Read`하고, 다음을 추출하여短く 요약して提示します:

- **AGENTS.md**: 역할 분담, 핸드오프 절차, 금지 사항, 환경/전제
- **CLAUDE.md**: 중요한 제약（금지 사항/권한/브랜치 운영）, 테스트 절차, 커밋 규정, 운영 규칙
- **Plans.md**: 작업 구조, 마커 운영, 현재 WIP/依頼中 작업

### Step 2:引き継ぎ 항목의 확정（대화）

요약을 바탕으로, 사용자에게 **유지/조정**하고 싶은 항목을 질문합니다（최대5〜10문으로 충분）:

- 반드시 남겨야 할 제약（예: 본환경 배포 금지,特定ディレクトリ 금지, 보안 요건）
- 역할 분담（Solo/2-agent）의 전제
- 브랜치 운영（main/staging 등）
- 테스트/빌드의 대표 명령
- Plans의 마커 운영（기존 규칙이 있으면整合）

### Step 3: 백업 생성（필수）

백업은 프로젝트内の `.claude-code-harness/backups/`에まとめます（git에入れしたくない 경우가 많음）.

예:

- `.claude-code-harness/backups/2025-12-13/AGENTS.md`
- `.claude-code-harness/backups/2025-12-13/CLAUDE.md`
- `.claude-code-harness/backups/2025-12-13/Plans.md`

`Bash`로 `mkdir -p`와 `cp`를使用해도 좋습니다.

### Step 4: 新フォーマットの生成（머지）

#### 4-1. Plans.md（작업 유지 머지）

`merge-plans`의方针으로 실행:

- 기존 🔴🟡🟢📦 작업을 유지
- 마커 凡例·최종更新情報はテンプレ側に更新
- 분석不能なら 백업을 남기고 템플릿 채택

#### 4-2. AGENTS.md / CLAUDE.md（テンプレ +引き継ぎブロック）

템플릿으로 뼈대를 만들고, Step 2에서 확정한 항목을 **新フォーマットの適切な場所に再配置**합니다.

최소한의方针:

- 기존"중요 규칙"은削らず、**「프로젝트 고유 규칙（마이그레이션）」**의 섹션으로 남김
- 역할 분담/플로우는 템플릿的形式に書き直す（의미는 유지）

### Step 5: 차분 확인과 완료

- `git diff`（또는 파일 차분）로 변경점을短く 요약
- 중요 포인트（권한/금지 사항/작업 상태）가 의도대로인지 최종 확인
- 문제가 있으면 즉시 수정

---

## 성과물（완료 조건）

- 기존 내용을 반영한 **新フォーマット版**의 `AGENTS.md` / `CLAUDE.md` / `Plans.md`
- `.claude-code-harness/backups/`에 백업이 남아 있음
- Plans의 작업은 사라지지 않음（유지）
