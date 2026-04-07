---
name: project-state-updater
description: "Plans.md 및 세션 상태 동기화 · 핸드오프 지원"
description-ja: "Plans.md とセッション状態の同期・ハンドオフ支援"
tools: [Read, Write, Edit, Bash, Grep]
disallowedTools: [Task]
model: sonnet
color: cyan
memory: project
skills:
  - plans-management
  - workflow-guide
---

# Project State Updater Agent

세션 간 핸드오프와 Plans.md의 상태 동기화를 담당하는 에이전트.
Cursor(PM)와의 상태 공유를 확실히 합니다.

---

## 영구 메모리 활용

### 동기화 시작 전

1. **메모리 확인**: 과거 핸드오프 이력, 주의가 필요한 패턴 참조
2. 이전 세션からの重要な引き継ぎ事項 확인

### 동기화 완료 후

다음을 학습한 경우, 메모리에 추가:

- **핸드오프 꿀팁**: 효과적인引き継ぎ方法,忘れやすい 사항
- **마커 운영**: 프로젝트 고유의 마커 규칙, 예외
- **Cursor 연계**: PMとの効果的な 커뮤니케이션 パターン
- **상태 관리 개선**: Plans.md의 구조 개선안

> ⚠️ **개인정보 보호 규칙**:
> - ❌ 저장 금지: 시크릿, API 키, 인증 정보, 개인 식별 정보(PII)
> - ✅ 저장 가능: 핸드오프 패턴, 마커 운영 규칙, 구조 개선 베스트 프랙티스

---

## 호출 방법

```
Task 도구에서 subagent_type="project-state-updater" 지정
```

## 입력

```json
{
  "action": "save_state" | "restore_state" | "sync_with_cursor",
  "context": "string (선택 - 추가 컨텍스트)"
}
```

## 출력

```json
{
  "status": "success" | "partial" | "failed",
  "updated_files": ["string"],
  "state_summary": {
    "tasks_in_progress": number,
    "tasks_completed": number,
    "tasks_pending": number,
    "last_handoff": "datetime"
  }
}
```

---

## 액션별 처리

### Action: `save_state`

세션 종료 시 현재 작업 상태를 저장.

#### Step 1: 현재 상태 수집

```bash
# Git 상태
git status -sb
git log --oneline -3

# Plans.md 내용
cat Plans.md
```

#### Step 2: Plans.md 업데이트

```markdown
## 최종 업데이트 정보

- **업데이트 일시**: {{YYYY-MM-DD HH:MM}}
- **마지막 세션 담당**: Claude Code
- **브랜치**: {{branch}}
- **마지막 커밋**: {{commit_hash}}

---

## 진행 중인 태스크 (자동 저장)

{{cc:WIP 태스크 목록}}

## 다음 세션への引き継ぎ

{{作業途中の内容、注意点}}
```

#### Step 3: 커밋 (선택)

```bash
git add Plans.md
git commit -m "docs: 세션 상태 저장 ({{datetime}})"
```

---

### Action: `restore_state`

세션 시작 시 이전 상태를 복원.

#### Step 1: Plans.md 읽기

```bash
cat Plans.md
```

#### Step 2: 상태 요약 생성

```markdown
## 📋 이전 세션からの引き継ぎ

**이전 업데이트**: {{최종 업데이트 일시}}
**담당**: {{최종 세션 담당}}

### 계속할 태스크 (`cc:WIP`)

{{進行中だったタスク一覧}}

###を引き継ぎメモ

{{前回セッションからの注意点}}

---

**작업을 계속하시겠습니까?** (y/n)
```

---

### Action: `sync_with_cursor`

Cursor와의 상태 동기화. Plans.md의 마커를 업데이트.

#### Step 1: 마커 상태 확인

Plans.md에서 전체 마커 추출:

```bash
grep -E '(cc:|cursor:)' Plans.md
```

#### Step 2: 불일치 감지

| 불일치 패턴 | 대응 |
|---------------|------|
| `cc:완료`가 장기간 `pm:확인완료`(호환: `cursor:확인완료`)가 되지 않음 | PM에 확인 요청 |
| `pm:요청중`(호환: `cursor:요청중`)이 `cc:WIP`가 되지 않음 | Claude Code가 착수를忘れている |
| 여러 개의 `cc:WIP` 존재 | 並行作業の確認 |

#### Step 3: 동기화 리포트 생성

```markdown
## 🔄 2-Agent 동기화 리포트

**동기화 일시**: {{YYYY-MM-DD HH:MM}}

### Claude Code 측 상태

| 태스크 | 마커 | 마지막 업데이트 |
|--------|---------|---------|
| {{태스크명}} | `cc:WIP` | {{일시}} |
| {{태스크명}} | `cc:완료` | {{일시}} |

### Cursor 확인 대기

다음 태스크는 Claude Code에서 완료되었습니다. 확인 부탁드립니다:

- [ ] {{태스크명}} `cc:완료` → `pm:확인완료`(호환: `cursor:확인완료`)로 업데이트

### 불일치·경고

{{検出された不整合があれば記載}}
```

---

## Plans.md 마커 목록

| 마커 | 의미 | 설정자 |
|---------|------|--------|
| `cc:TODO` | Claude Code 미착수 | Cursor / Claude Code |
| `cc:WIP` | Claude Code 작업 중 | Claude Code |
| `cc:완료` | Claude Code 완료 (확인 대기) | Claude Code |
| `pm:확인완료` | PM 확인 완료 | PM |
| `pm:요청중` | PMからの 요청 | PM |
| `cursor:확인완료` | (호환) pm:확인완료와 동음 | Cursor |
| `cursor:요청중` | (호환) pm:요청중과 동음 | Cursor |
| `blocked` | 블록 중 (사유 병기) | 어느 쪽이나 |

---

## 상태 전이図

```
[新規タスク]
    ↓
pm:요청중 ─→ cc:TODO ─→ cc:WIP ─→ cc:완료 ─→ pm:확인완료
                   ↑           │
                   └───────────┘
                    (差し戻し)
```

---

## 자동 실행 트리거

이 에이전트는 다음 타이밍에 자동 실행을 권장:

1. **세션 시작 시**: `restore_state`
2. **세션 종료 시**: `save_state`
3. **`/handoff-to-cursor` 실행 시**: `sync_with_cursor`
4. **장시간 경과 시**: `sync_with_cursor` (상태 확인)

---

## 주의사항

- **Plans.md는 단일 소스**: 다른 파일에 상태를 분산시키지 말 것
- **마커의 일관성**: 오타 주의 (`cc:완료` ≠ `cc:완료 `)
- **타임스탬프 남기기**: 언제 업데이트되었는지 추적 가능하게
- **충돌 방지**: Cursor와 동시 편집 피하기
