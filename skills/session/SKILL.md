---
name: session
description: "세션 관리의 종합 창구. 초기화·기억·상태를 도맡아 관리합니다. Claude Code 세션 관리, /session 명령을 사용할 때 호출하세요. 앱 사용자 세션, 로그인 상태, 인증 기능에는 사용하지 마세요."
description-en: "Unified session management window. Handles initialization, memory, state all-in-one. Use when managing Claude Code sessions, /session command. Do NOT load for: app user sessions, login state, authentication features."
description-ja: "セッション管理の総合窓口。初期化・記憶・状態を一手に引き受けます。Use when managing Claude Code sessions, /session command. Do NOT load for: app user sessions, login state, authentication features."
allowed-tools: ["Read", "Bash", "Write", "Edit", "Glob"]
argument-hint: "[list|inbox|broadcast \"message\"]"
---

# Session Skill (Unified)

세션 관련 기능을 하나의 스킬로 통합합니다.

## Usage

```bash
/session              # Show available options
/session list         # Show active sessions
/session inbox        # Check incoming messages
/session broadcast "message"  # Send message to all sessions
```

## Subcommands

### `/session list` - List Active Sessions

Shows all active Claude Code sessions in the current project.

```
📋 Active Sessions

| Session ID | Status | Last Activity |
|------------|--------|---------------|
| abc123     | active | 2 min ago     |
| def456     | idle   | 15 min ago    |
```

### `/session inbox` - Check Inbox

Checks for incoming messages from other sessions.

```
📬 Session Inbox

| From | Time | Message |
|------|------|---------|
| abc123 | 5m ago | "Ready for review" |
| def456 | 10m ago | "API implementation done" |
```

### `/session broadcast "message"` - Broadcast Message

Sends a message to all active sessions.

```bash
/session broadcast "Review complete, ready for merge"
```

---

## Capabilities

| Feature | Description | Reference |
|---------|-------------|-----------|
| **Initialization** | Start new session, load context | See [../session-init/SKILL.md](../session-init/SKILL.md) |
| **Memory** | Persist learnings across sessions | See [../session-memory/SKILL.md](../session-memory/SKILL.md) |
| **State Control** | Resume/fork session based on flags | See [references/session-control.md](${CLAUDE_SKILL_DIR}/references/session-control.md) |
| **Communication** | Cross-session messaging | See [../session-state/SKILL.md](../session-state/SKILL.md) |

---

## 메모리 최적화（CC 2.1.49+）

Claude Code 2.1.49以降,セッション再開時のメモリ使用量が **68%削減**되었습니다.

### 장시간 세션 관리의 베스트 프랙티스

| 워크로드 | 권장 전략 |
|------------|---------|
| **일반 구현** | 1-2시간ごとに `--resume`로再開 |
| **대규모 리팩터** | 기능 단위로 세션 분할 → 各セッションで `--resume` |
| **병렬 작업** | `/work all`로 병렬 실행, 장시간이면途中에서 `--resume` |
| **메모리 경고時** | 即座に `--resume`로再開（이전보다高速） |

### 세션명의 자동 생성（CC 2.1.41+）

`/rename`을 인수 없이 실행하면, 대화 컨텍스트からセッション名を自動生成합니다.
장시간 세션이나 `--resume`을多用するワークフロー에서 세션의 식별이 용이해집니다.

### 효율적인 워크플로우 예

```bash
# 구현 단계 1
claude "인증 기능 구현"
# → 1시간後

# 세션 다시 시작（메모리 효율적）
claude --resume "비밀번호 재설정 기능 추가"
# → 1시간後

# 다시 시작
claude --resume "테스트 추가"
```

### 메모리 관리의 권장 사항

| 권장 사항 | 이유 |
|---------|------|
| **적극적인 세션 다시 시작** | 68% 메모리削減で再開コストが低い |
| **정기적인 다시 시작** | 컨텍스트를 정리하고 집중력을 유지 |
| **기능 단위의 분할** | 대규모 작업을 작게 나누어 다시 시작 |
| **Plans.md 활용** | 다시 시작時の引継ぎがスムーズ |

> 💡  메모리 효율이 크게 개선되었으므로, 세션 다시 시작을 적극 활용하세요.

---

## When to Use

- Session initialization (`/harness-init`)
- Session resume/fork (`/work --resume`, `/work --fork`)
- Memory persistence (automatic)
- Cross-session communication (`/session broadcast`)

## Execution Flow

### 1. Session Initialization

```
/harness-init
    ↓
├── Load project context
├── Initialize session.json
├── Load previous session memory (if exists)
└── Display session status
```

### 2. Session Control (from /work)

```
/work --resume
    ↓
├── Check session.json exists
├── Load session state
└── Continue from last checkpoint

/work --fork
    ↓
├── Create new session branch
├── Copy relevant context
└── Start fresh with context
```

### 3. Memory Persistence

```
Session end
    ↓
├── Extract learnings (gotchas, patterns)
├── Update .claude/memory/*.md
└── Prepare handoff summary
```

### 4. Cross-Session Communication

```
/session broadcast "message"
    ↓
├── Find active sessions
├── Write to session.events.jsonl
└── Notify all sessions
```

## Files Managed

| File | Purpose |
|------|---------|
| `.claude/state/session.json` | Current session state |
| `.claude/state/session.events.jsonl` | Event log for cross-session communication |
| `.claude/memory/*.md` | Persistent memory files |

## Migration Note

This skill consolidates:
- `session-init` → Session initialization
- `session-memory` → Memory persistence
- `session-control` → Resume/fork control
- `session-state` → State management & communication

The individual skills are deprecated but still work for backward compatibility.
