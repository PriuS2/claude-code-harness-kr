---
name: breezing
description: "팀 실행 모드 — harness-work의 팀 협업 앨리어스. breezing, 팀 실행, 다 해줘로 트리거."
description-ja: "팀 실행 모드 — harness-work의 팀 협업 앨리어스. breezing, 팀 실행, 다 해줘로 트리거."
description-en: "Team execution mode — backward-compatible alias for harness-work with team orchestration."
allowed-tools: ["Read", "Write", "Edit", "Bash", "Grep", "Glob", "Task", "WebSearch"]
argument-hint: "[all|N-M|--codex|--parallel N|--no-commit|--no-discuss|--auto-mode]"
user-invocable: true
---

# Breezing — Team Execution Mode

> **하위 호환 앨리어스**: `harness-work`를 팀 실행 모드로 동작시킵니다.

## Quick Reference

```bash
breezing                        # 스코프를 물어보고 실행
breezing all                    # Plans.md 전 작업을 완주
breezing 3-6                    # 작업 3〜6을 완주
breezing --codex all            # Codex CLI로 전 작업 완주
breezing --parallel 2 all       # 2병렬로 전 작업 완주
breezing --no-discuss all       # 계획 토론 스킵으로 전 작업 완주
breezing --auto-mode all        # 호환한 부모 세션에서 Auto Mode rollout를 시도
```

## Options

| Option | Description | Default |
|--------|-------------|---------|
| `all` | 전 미완료 작업 대상 | - |
| `N` or `N-M` | 작업 번호/범위 지정 | - |
| `--codex` | Codex CLI로 구현 의뢰 | false |
| `--parallel N` | Implementer 병렬 수 | auto |
| `--no-commit` | 자동 커밋 억제 | false |
| `--no-discuss` | 계획 토론 스킵 | false |
| `--auto-mode` | Auto Mode rollout를 명시. 부모 세션의 permission mode가 호환인 경우에만 도입 검토 | false |

## Execution

**이 스킬은 `harness-work`에 위임합니다.** 다음 설정으로 `harness-work`를 실행하세요:

1. **인수를 그대로 `harness-work`에 전달**
2. **팀 실행 모드 강제** — Lead → Worker spawn → Reviewer spawn의 3자 분리
3. **Lead는 delegate 전념** — 코드를 직접 쓰지 않음
4. **Auto Mode는 opt-in 취급** — `--auto-mode`는 호환한 부모 세션에서의 rollout용 플래그로 받아들임

### `harness-work`와의 차이

| 특징 | `harness-work` | `breezing` (이 스킬) |
|------|-----------------|------------------------|
| 병렬 수단 | 필요 수에 따른 자동 분할 | **Lead/Worker/Reviewer의 역할 분리** |
| Lead의 역할 | 조정+구현 | **delegate (조정 전념)** |
| 리뷰 | Lead 자기 리뷰 | **독립 Reviewer** |
| 기본 스코프 | 다음 작업 | **전부** |

### Team Composition

| Role | Agent Type | Mode | 책임 |
|------|-----------|------|------|
| Lead | (self) | - | 조정·지휘·작업 배분 |
| Worker ×N | `claude-code-harness:worker` | `bypassPermissions`（현행） / Auto Mode（follow-up）* | 구현 |
| Reviewer | `claude-code-harness:reviewer` | `bypassPermissions`（현행） / Auto Mode（follow-up）* | 독립 리뷰 |

> *부모 세션 또는 frontmatter가 `bypassPermissions`인 경우그쪽이 우선. 배포 템플릿은 현재도 `bypassPermissions`을 사용하므로, Auto Mode는 follow-up의 rollout 대상이며,기본 동작이 아니다.

### Codex Mode (`--codex`)

공식 플러그인 `codex-plugin-cc` 경유로 Codex CLI에 모든 구현을 의뢰하는 모드:

```bash
# 작업 의뢰（쓰기 가능）
bash scripts/codex-companion.sh task --write "작업 내용"

# stdin 경유（큰 프롬프트용）
CODEX_PROMPT=$(mktemp /tmp/codex-prompt-XXXXXX.md)
# 작업 내용을 기록
cat "$CODEX_PROMPT" | bash scripts/codex-companion.sh task --write
rm -f "$CODEX_PROMPT"
```

## Flow Summary

```
breezing [scope] [--codex] [--parallel N] [--no-discuss] [--auto-mode]
    │
    ↓ Load harness-work with team mode
    │
Phase 0: Planning Discussion (--no-discuss 으로 스킵)
Phase A: Pre-delegate（팀 초기화）
Phase B: Delegate（Worker 구현 + Reviewer 리뷰）
Phase C: Post-delegate（통합 검증 + Plans.md 업데이트 + commit）
```

### Progress Feed（Phase B 중의 진행 알림）

Lead는 Worker의 작업 완료 시에 따라, 다음 형식으로 진행을 출력합니다:

```
📊 Progress: Task {completed}/{total} 완료 — "{작업 주제}"
```

**출력 예**:
```
📊 Progress: Task 1/5 완료 — "harness-work에 실패 재 티켓화 추가"
📊 Progress: Task 2/5 완료 — "harness-sync에 --snapshot 추가"
📊 Progress: Task 3/5 완료 — "breezing에 프로그레스 피드 추가"
```

> **설계 의도**: breezing은 장시간 실행이 되는 경우가 많습니다.
> 사용자가 터미널을 살짝 볼 때「지금 어디까지 진행되었는지」가 한눈에 보이도록 합니다.
> task-completed.sh 훅이 systemMessage으로 동등한 정보를 출력하므로, Lead의 출력과 상호 보완.

### Review Policy（전 모드 통일）

Breezing 모드에서도 리뷰는 **Codex exec 우선 → 내부 Reviewer 폴백**의 통일 정책에 따릅니다.
상세는 `harness-work`의「리뷰 루프」섹션을 참조합니다.

- Worker가 worktree 내에서 구현·commit → Lead에게 결과 반환
- Lead가 Codex exec로 리뷰（120s 타임아웃, 폴백: Reviewer agent）
- REQUEST_CHANGES → Lead가 SendMessage로 Worker에게수정 지시, Worker가 amend（최대 3회）
- APPROVE → **Lead**가 main에 cherry-pick → Plans.md를 `cc:완료 [{hash}]`로 업데이트

### 완료 보고（Phase C — Lead가 생성）

전 작업 완료 후, **Lead**가 다음 절차로 리치 완료 보고를 생성합니다:

1. `git log --oneline {base_ref}..HEAD`로 전 cherry-pick 커밋을 수집
2. `git diff --stat {base_ref}..HEAD`로 전체 변경 규모를 취득
3. Plans.md의 `cc:TODO` / `cc:WIP` 잔작업을 추출
4. `harness-work`의「완료 보고 포맷」의 Breezing 템플릿에 따라 출력

> **생성자는 Lead**. Worker나 훅이 아닙니다. Lead가 Phase C에서 git + Plans.md를 읽고 생성합니다.

### Phase 0: Planning Discussion（구조화 3문 체크）

전 작업 실행전에, 다음 3문으로 계획의 건전성을 확인합니다.
`--no-discuss` 지정시는 전체 스킵.

**Q1. 스코프 확인**:
> "{{N}}건의 작업을 실행합니다. 스코프가 적절합니까?"

너무 많을 경우 우선순위（Required > Recommended > Optional）로 선별 제안.

**Q2. 의존 관계 확인**（Plans.md에 Depends 컬럼이 있는 경우만）:
> "작업 {{X}}는 {{Y}}에 의존하고 있습니다. 실행 순서가 맞습니까?"

Depends 컬럼을 읽고, 의존 체인을 표시. 순환 의존이 있으면 에러.

**Q3. 리스크 플래그**（`[needs-spike]` 작업이 있는 경우만）:
> "작업 {{Z}}는 [needs-spike]입니다. 먼저 spike합니까?"

spike 미완료의 `[needs-spike]` 작업이 있는 경우, spike를 먼저 실행할지 확인.

3문 모두 문제없다면, Phase A로 진행（총 30초로 완료하도록 설계）.

### 의존 그래프에 따른 작업 배분

Plans.md에 Depends 컬럼이 있는 경우（v2 포맷）, 의존 그래프에 따라 작업을 실행합니다:

1. **Depends가 `-`인 작업**을 먼저 실행. 독립 작업이여러 개 있으면 병렬 spawn 가능
2. 각 Worker 완료 후, Lead가 리뷰→cherry-pick（harness-work Phase B 참조）
3. 의존 원작업이 main에 cherry-pick되면, 그 작업에 의존하던 작업을 다음 실행
4. 모든 작업이 완료될 때까지 반복

> **주의**: 각 작업의「Worker 완료→리뷰→cherry-pick」은 순차 처리.
> 병렬화 할 수 있는 것은 독립 작업（Depends가 `-`）의 Worker spawn 부분만.

## Codex Native Orchestration

Codex에서는 native subagent를 사용합니다.
대표적인 제어 면은 `spawn_agent`, `wait`, `send_input`, `resume_agent`, `close_agent`.

> **Claude Code vs Codex의 통신 API**（SSOT: `team-composition.md`의 API 매핑표）:
> - Claude Code: `SendMessage(to: agentId, message: "...")`로 Worker에게수정 지시
> - Codex: `resume_agent(agent_id)`로 Worker를 재개 → `send_input(agent_id, "...")`로 지시 전송
>
> harness-work의 유사 코드는 Claude Code 구문으로 기술. Codex 환경에서 위의 것으로 대체.

## Related Skills

- `harness-work` — 단일 작업에서 팀 실행까지（본체）
- `harness-sync` — 진행 동기화
- `harness-review` — 코드 리뷰（breezing 내에서 자동 시작）
