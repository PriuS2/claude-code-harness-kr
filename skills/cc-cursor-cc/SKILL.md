---
name: cc-cursor-cc
description: "Cursor PM으로 아이디어를 검증하고 Plans.md를 업데이트하여バトンタッチ。Cursor ↔ Claude Code 2-Agent 워크플로우 지원。Use when user mentions Cursor PM handoff, 2-agent plan validation, CC-Cursor round trip, or brainstorm review. Do NOT load for: implementation work, single-agent tasks, or direct coding."
description-en: "Validates brainstormed ideas with Cursor PM, updates Plans.md, then handoff back. Cursor ↔ Claude Code 2-Agent workflow support."
description-ja: "Cursor PMでアイデアを検証し Plans.md を更新してバトンタッチ。Cursor ↔ Claude Code 2-Agent ワークフロー対応。"
allowed-tools: ["Read", "Write", "Edit", "Bash"]
user-invocable: false
---

# CC-Cursor-CC Skill (Plan Validation Round Trip)

**Cursor (PM)**로 브레인스토밍 내용을 전송하여 실현 가능성 검증을 지원하는 스킬.

## Prerequisites

이 스킬은 **2-agent 작업**을 전제로 합니다.

| Role | Agent | 설명 |
|------|-------|------|
| **PM** | Cursor | plans 검증, Plans.md 업데이트 |
| **Impl** | Claude Code | 브레인스토밍, 구현 |

## 실행 흐름

### Step 1: 브레인스토밍 컨텍스트 추출

최근 대화에서 다음을 추출:
1. **목표** (feature/purpose)
2. **기술 선택**
3. **결정 사항**
4. **미결정 항목**
5. **우려 사항**

### Step 2: Plans.md에 임시 작업 추가

```markdown
## 🟠 Under Validation: {{Project}} `pm:awaiting-validation`

### Provisional Tasks (To Validate)
- [ ] {{task1}} `awaiting-validation`
- [ ] {{task2}} `awaiting-validation`

### Undecided Items
- {{item1}} → **Requesting PM decision**
```

### Step 3: Cursor용 검증 요청 생성

복사하여 Cursor에 붙여넣을 텍스트 생성:

```markdown
## 📋 Plan Validation Request

**Goal**: {{summary}}

**Provisional tasks**:
1. {{task1}}
2. {{task2}}

### ✅ Requesting Cursor (PM) to:
1. Validate feasibility
2. Break down tasks
3. Decide undecided items
4. Update Plans.md (awaiting → cc:TODO)
```

### Step 4: 다음 행동 안내

1. 요청을 **Cursor**에 복사 & 붙여넣기
2. Cursor에서 `/plan-with-cc` 실행
3. Cursor가 Plans.md 업데이트
4. Cursor가 `/handoff-to-claude` 실행
5. **Claude Code**에 다시 복사 & 붙여넣기

## 전체 흐름

```
Claude Code (Brainstorm)
    ↓ /cc-cursor-cc
Cursor (PM validates & breaks down)
    ↓ /handoff-to-claude
Claude Code (/work implements)
```
