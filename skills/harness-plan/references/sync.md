# sync 서브명령어 — 진행 동기화 플로우

구현 상황과 Plans.md를 대조하고 차이를 검출·업데이트합니다.

## Step 0: Plans.md 검증

Plans.md의 존재와 포맷을 확인합니다. 문제 있으면 즉시 안내하고 정지합니다.

| 상태 | 안내 |
|------|------|
| Plans.md가 존재하지 않음 | `Plans.md를 찾을 수 없습니다. /harness-plan create로 생성해 주세요.` → **정지** |
| 헤더에 DoD / Depends 컬럼 없음（v1 형식） | `Plans.md가旧포맷（3컬럼）입니다. /harness-plan create로 v2（5컬럼）に再生成してください.既存タスクは自動的に引き継がれます。` → **정지** |
| v2 형식（5컬럼） | 그대로 Step 1로 진행 |

## Step 1: 현황 수집（병렬）

```bash
# Plans.md의 상태
cat Plans.md

# Git 변경 상태
git status
git diff --stat HEAD~3

#直近コミット履歴
git log --oneline -10

# 에이전트 트레이스（直近の編集ファイル）
tail -20 .claude/state/agent-trace.jsonl 2>/dev/null | jq -r '.files[].path' | sort -u
```

## Step 1.5: Agent Trace 분석

Agent Trace에서直近の編集履歴を取得し、Plans.mdのタスクと照合します:

```bash
#直近の編集ファイル一覧
RECENT_FILES=$(tail -20 .claude/state/agent-trace.jsonl 2>/dev/null | \
  jq -r '.files[].path' | sort -u)

# プロジェクト情報
PROJECT=$(tail -1 .claude/state/agent-trace.jsonl 2>/dev/null | \
  jq -r '.metadata.project')
```

**照合 포인트**:

| 체크 항목 | 검출 방법 |
|------------|----------|
| Plans.md에 없는 파일 편집 | Agent Trace vs 작업 기술 |
| 작업 기술과 다른 파일 | 예상 파일 vs 실제 편집 |
| 장시간 편집 없는 작업 | Agent Trace 시계열 vs WIP 기간 |

## Step 2: 차이 검출

| 체크 항목 | 검출 방법 |
|------------|----------|
| 완료인데 `cc:WIP` | 커밋履歴 vs 마커 |
| 착수인데 `cc:TODO` | 변경 파일 vs 마커 |
| `cc:완료`인데 미커밋 | git status vs 마커 |

### Artifact Hash 하위 호환

`cc:완료 [a1b2c3d]` 형식（commit hash 포함）과 `cc:완료`（hash 없음）의両方を認識합니다.

**매칭 규칙**:
- `cc:완료` → hash 없음 완료로 취급
- `cc:완료 [xxxxxxx]` → hash 포함 완료로 취급. 7문자의 단축 hash를保持
- hash 포함の場合、`git log --oneline`と照合してコミットの存在を確認可能

> **하위 호환**: hash 없음 형식도 계속有効. 기존 Plans.md를破壊하지 않음.

## Step 3: Plans.md 업데이트 제안

차이가 검출된 경우, 제안하여 실행합니다:

```
Plans.md 업데이트 필요

| Task | 현재 | 변경후 | 이유 |
|------|------|--------|------|
| XX   | cc:WIP | cc:완료 | 커밋됨 |
| YY   | cc:TODO | cc:WIP | 파일 편집됨 |

업데이트しますか? (yes / no)
```

## Step 4: 진행 요약 출력

```markdown
## 진행 요약

**프로젝트**: {{project_name}}

| 상태 | 건수 |
|----------|------|
| 미착수 (cc:TODO) | {{count}} |
| 작업중 (cc:WIP) | {{count}} |
| 완료 (cc:완료) | {{count}} |
| PM확인済 (pm:確認済) | {{count}} |

**진행율**: {{percent}}%

###直近の編集ファイル (Agent Trace)
- {{file1}}
- {{file2}}
```

## Step 5: 다음 액션 제안

```
다음에 할 것

**우선 1**: {{작업}}
- 이유: {{依頼中 / アンブロック待ち}}

**권장**: harness-work, harness-review
```

## 이상 검출

| 상황 | 경고 |
|------|------|
| 복수의 `cc:WIP` | 복수 작업 동시 진행중 |
| `pm:依頼中` 미처리 | PM의依頼を先に処理する |
| 큰 괴리 | 작업 관리가跟不上 |
| WIP가 3일 이상 변화없음 | 블록されていないか確認 |

## Step 6: 레트로스펙티브（기본 ON）

`sync` 실행時、`cc:완료` 작업이 1건 이상 있으면自動的に振り返りを実行합니다.
`--no-retro`로 명시적으로 건너뛰기 가능.

### Step R1: 완료 작업 수집

```bash
# Plans.md에서 cc:완료 / pm:확인済의 작업을 추출
grep -E 'cc:완료|pm:確認済' Plans.md

#直近の完了コミット履歴
git log --oneline --since="7 days ago"

# 변경 규모
git diff --stat HEAD~10
```

### Step R2:振り返り 4 항목

| 항목 | 분석 방법 |
|------|---------|
| **견적 정밀도** | Plans.md의 작업 기술에서 예상 파일 수를 추론 → `git diff --stat`의 실제 변경 파일 수와 비교 |
| **블록 원인** | `blocked` 마커가 붙은 작업의 이유 패턴을 집계（기술적/외부 의존/스펙 불명확） |
| **품질 마커 적중률** | `[feature:security]` 등을 붙인 작업에서 실제로 관련 문제가 발생했는지 |
| **스코프 변동** | Plans.md의 첫 커밋時의 작업 수 vs 현재 작업 수（추가/삭제 건수） |

### Step R3:振り返り 요약 출력

```markdown
##振り返りサマリー

**기간**: {{start_date}} 〜 {{end_date}}

| 지표 | 값 |
|------|-----|
| 완료 작업 | {{count}} 건 |
| 블록 발생 | {{blocked_count}} 건 |
| 스코프 변동 | +{{added}} / -{{removed}} 건 |
| 견적 정밀도 | 예상 {{est}} 파일 → 실제 {{actual}} 파일 |

### 배운 것
- {{1-2 행의 배움}}

###次に活かすこと
- {{1-2 행의 개선 액션}}
```

### Step R4: harness-memへの記録

振り返り 결과를 harness-mem에 기록하고, 次回の `create`時に参照できるようにします.
記録先: `.claude/agent-memory/`配下の該当エージェントメモリ.
