---
name: session-control
description: "Apply /work --resume/--fork flags by updating session state files."
allowed-tools: ["Read", "Bash", "Write", "Edit"]
---

# Session Control

## 입력

workflow 변수:
- `resume_session_id` (string)
- `resume_latest` (boolean)
- `fork_session_id` (string)
- `fork_reason` (string)

## 실행

### 1) 인수의 결정
- resume:
  - `resume_latest == true` → `--resume latest`
  - 그 외で `resume_session_id`가 있으면 `--resume <id>`
- fork:
  - `fork_session_id`가 있으면 `--fork <id>`, 없으면 `--fork current`
  - `fork_reason`가 있으면 `--reason "<text>"`

### 2) 스크립트 실행
```bash
./scripts/session-control.sh --resume <id|latest>
./scripts/session-control.sh --fork <id|current> --reason "<text>"
```

## 기대される 결과
- `.claude/state/session.json`이 업데이트됨
- `.claude/state/session.events.jsonl`에 `session.resume` 또는 `session.fork`가追記됨
- 에러時は stderr에 이유가 출력됨
