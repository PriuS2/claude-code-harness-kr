---
name: state-transition
description: "Execute session state transitions using session-state.sh"
allowed-tools: [Read, Bash]
---

# State Transition

세션 상태의 전이를 실행합니다.

## 입력

workflow 변수:
- `target_state` (string): 전이先の 상태
- `event_name` (string): 트리거 이벤트
- `event_data` (string, optional): 이벤트 부가 데이터 (JSON)

## 유효한 상태

| 상태 | 설명 |
|------|------|
| `idle` | 세션 미시작 |
| `initialized` | SessionStart 완료 |
| `planning` | Plan/Work의 준비 |
| `executing` | /work 실행중 |
| `reviewing` | review 실행중 |
| `verifying` | build/test 실행중 |
| `escalated` | 인간 확인 대기 |
| `completed` | 성과물 확정 |
| `failed` | 회복 불가능 |
| `stopped` | Stop 훅到达 |

## 대표적인 전이

| From | Event | To |
|------|-------|----|
| idle | session.start | initialized |
| initialized | plan.ready | planning |
| planning | work.start | executing |
| executing | work.task_complete | reviewing |
| reviewing | verify.start | verifying |
| verifying | verify.passed | completed |
| verifying | verify.failed | escalated |
| * | session.stop | stopped |
| stopped | session.resume | initialized |

## 실행

```bash
./scripts/session-state.sh --state <state> --event <event> [--data <json>]
```

### 예: 실행 상태로의 전이

```bash
./scripts/session-state.sh --state executing --event work.start
```

### 예: 에스컬레이션（데이터 포함）

```bash
./scripts/session-state.sh --state escalated --event escalation.requested \
  --data '{"reason":"Build failed 3 times","retry_count":3}'
```

## 기대되는 결과

- `.claude/state/session.json`의 `state`, `updated_at`, `last_event_id`, `event_seq`가 업데이트됨
- `.claude/state/session.events.jsonl`에 이벤트가追記됨
-不正な遷移は stderr にエラー出力 + 非ゼロ終了

## 에러 핸들링

전이가 실패한 경우（不正な遷移等）:
1. 현재 상태와 허용된 전이를 stderr에 출력
2. 非ゼロ終了コードを返す
3. 호출원（workflow）에서 에스컬레이션 처리를 수행
