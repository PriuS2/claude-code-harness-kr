---
name: harness-work
description: "Harness v3 統合実行スキル。Plans.md タスクを1件から全並列チーム実行まで担当。다음으로起動: 実装して、実行して、harness-work、全部やって、breezing、チーム実行、parallel。プランニング・レビュー・リリース・セットアップ에는使わない。"
description-en: "Unified execution skill for Harness v3. Implements Plans.md tasks from single task to full parallel team runs. Use when user mentions: implement, execute, harness-work, do everything, build features, run tasks, breezing, team run, parallel. Do NOT load for: plang, code review, release, or setup."
description-ja: "Harness v3 統合実行スキル。Plans.md タスクを1件から全並列チーム実行まで担当。다음으로起動: 実装して、実行して、harness-work、全部やって、breezing、チーム実行、parallel。プランニング・レビュー・リリース・セットアップ에는使わない。"
allowed-tools: ["Read", "Write", "Edit", "Grep", "Glob", "Bash", "Task"]
argument-hint: "[all] [task-number|range] [--codex] [--parallel N] [--no-commit] [--resume id] [--breezing] [--auto-mode]"
effort: high
---

# Harness Work (v3)

Harness v3 の統合実行スキル。
이하의旧スキルを統合:

- `work` — Plans.md タスクの実装（スコープ自動判断）
- `impl` — 機能実装（タスクベース）
- `breezing` — チームフル自動実行
- `parallel-workflows` — 並列ワークフロー最適化
- `ci` — CI 失敗時の복구

## Quick Reference

| ユーザー入力 | モード | 動作 |
|------------|--------|------|
| `harness-work` | **auto** | タスク数で自動判定（下記参照） |
| `harness-work all` | **auto** | 全未완료タスクを自動モード실행 |
| `harness-work 3` | solo | タスク3だけ即実行 |
| `harness-work --parallel 5` | parallel | 5ワーカーで並列実行（強制） |
| `harness-work --codex` | codex | Codex CLI に委託（明示時のみ） |
| `harness-work --breezing` | breezing | チーム実行を強制 |

## Execution Mode Auto Selection（フラグ없음時の自動判定）

明示적인モードフラグ（`--parallel`, `--breezing`, `--codex`）がない경우、
대象タスク数に応じて最適なモードを自動選択する:

| 대象タスク数 | 自動選択モード | 理由 |
|-------------|---------------|------|
| **1 件** | Solo | オーバーヘッド最小。直接実装が最速 |
| **2〜3 件** | Parallel（Task tool） | Worker 分離のメリットが出시작る임계값 |
| **4 件이상** | Breezing | Lead 調整 + Worker 並列 + Reviewer 独立の三者分離が効果的 |

### ルール

1. **明示フラグは常にオートモードを上書き**する
   - `--parallel N` → Parallel モード（タスク数に関係なく）
   - `--breezing` → Breezing モード（タスク数に関係なく）
   - `--codex` → Codex モード（タスク数に関係なく）
2. **`--codex` は明示時のみ発動**。Codex CLI が未インストールの환경があるため、自動選択しない
3. `--codex` は他モードと組み合わせ可能: `--codex --breezing` → Codex + Breezing

## オプション

| オプション | 説明 | デフォルト |
|----------|------|----------|
| `all` | 全未완료タスクを대象 | - |
| `N` or `N-M` | タスク番号/範囲指定 | - |
| `--parallel N` | 並列ワーカー数 | auto |
| `--sequential` | 直列実行強制 | - |
| `--codex` | Codex CLI で実装委託（明示時のみ、自動選択しない） | false |
| `--no-commit` | 自動コミット抑制 | false |
| `--resume <id\|latest>` | 이전セッション再개 | - |
| `--breezing` | Lead/Worker/Reviewer のチーム実行 | false |
| `--no-tdd` | TDD フェーズスキップ | false |
| `--no-simplify` | Auto-Refinement スキップ | false |
| `--auto-mode` | Auto Mode rollout を明示。親セッションの permission mode が互換な경우のみ採用を検討 | false |

> **Token Optimization (v2.1.69+)**: git 操作を따르지 않는경량 태스크では
> plugin settings の `includeGitInstructions: false` を유効にして
> プロンプトトークンを절감できる。

## スコープダイアログ（引数없음時）

```
harness-work
어디までやりますか?
1) 次のタスク: Plans.md の次の未완료タスク → Solo 실행
2) 全部（推奨）: 残りのタスクを전체완료 → タスク数で自動モード選択
3) 番号指定: タスク番号を入力（例: 3, 5-7）→ 件数で自動モード選択
```

引数ありなら即実行（대話スキップ）:
- `harness-work all` → 全タスク、自動モード選択
- `harness-work 3-6` → 4件なので Breezing 自動選択

## Effort レベル制御（v2.1.68+, v2.1.72 단순화）

Claude Code v2.1.68 で Opus 4.6 は **medium effort** (`◐`) がデフォルト。
v2.1.72 で `max` レベルが廃止され、3段階 `low(○)/medium(◐)/high(●)` に단순화。
`/effort auto` でデフォルトにリセット可能。
복잡한タスク에는 `ultrathink` キーワードで high effort (`●`) を유効化する。

### 多要素スコアリング

タスク착수時に이하의スコアを합산し、**임계값 3 이상**で ultrathink を注入:

| 要素 | 条件 | スコア |
|------|------|--------|
| ファイル数 | 変갱대象 4 ファイル이상 | +1 |
| ディレクトリ | core/, guardrails/, security/ を포함하는 | +1 |
| キーワード | architecture, security, design, migration を포함하는 | +1 |
| 失敗履歴 | agent memory に同タスクの실패 기록あり | +2 |
| 明示指定 | PM テンプレートに ultrathink 記載あり | +3（自動採用） |

### 注入方法

スコア ≥ 3 の경우、Worker spawn prompt の머리말に `ultrathink` を追加。
breezing モードでも同じロジックが適用される（harness-work が一本化して管理）。

## 実行モード詳細

### Solo モード（1 件時の自動選択）

1. Plans.md を読み込み、대象タスクを특정
   - **Plans.md が존재しない경우**: `harness-plan create --ci` を自動呼び出し → Plans.md を생성して続行
   - ヘッダーに DoD / Depends カラムがない경우: `Plans.md が旧포맷です。harness-plan create で재생성하세요。` → **停止**
   - **会話に未記載タスクが있는 경우**: 直前の会話コンテキストから要件を抽出し、Plans.md に `cc:TODO` で自動追記
     - 抽出ロジック: ユーザー発言からアクション動詞（「〜を追加」「〜を수정」「〜を実装」）を検出
     - 追記時は v2 フォーマット（Task / 내容 / DoD / Depends / Status）에 준거
     - 追記後、ユーザーに「Plans.md に이하를追記했습니다」と表示（5 秒타임아웃付きプロンプト、デフォルト: 続行）
1.5. **タスク背景確認**（30 秒）:
   - タスクの「내容」と「DoD」から **目的**（이タスクが解く課題）を 1 行で推論表示
   - `git grep` / `Glob` で **影響範囲**（変갱が及ぶファイル/モジュール）を推論表示
   - 推論に自信が있는 경우: 그まま実装に進む（フロー遅延없음）
   - 推論に自信がない경우: ユーザーに 1 問だけ確認（「이理解で合っていますか？」）
2. タスクを `cc:WIP` に갱新
3. **TDD フェーズ**（`[skip:tdd]` 없음 & テストFW존재時）:
   a. テストファイルを선に作成（Red）
   b. 失敗を確認
4. `scripts/generate-sprint-contract.sh <task-id>` で `sprint-contract.json` を생성
5. Reviewer 観点の追記を `scripts/enrich-sprint-contract.sh` で加え、`scripts/ensure-sprint-contract-ready.sh` で approved を確認
6. コードを実装（Green）（Read/Write/Edit/Bash）
7. `/simplify` で Auto-Refinement（`--no-simplify` で생략可）
8. **自動レビューステージ**（「レビューループ」参照）:
   - Codex exec 優선でレビュー実行 → フォールバックで내部 Reviewer agent
   - `sprint-contract.json` の `reviewer_profile` が `runtime` の경우は `scripts/run-contract-review-checks.sh` 실행
   - REQUEST_CHANGES の경우: 指摘を元に수정→再レビュー（最大 3 回）
   - APPROVE で次ステップへ。self-check だけでは완료を確定しない
9. `scripts/write-review-result.sh` で review artifact を正規化して保存
10. `git commit` で自動コミット（`--no-commit` で생략可）
11. タスクを `cc:완료` に갱新（commit hash 付여）
   - `git log --oneline -1` で直近の commit hash（短縮形 7 文字）を取得
   - Plans.md の Status を `cc:완료 [a1b2c3d]` 形式で갱新
   - commit がない경우（`--no-commit` 時）は hash 없음で `cc:완료` のみ
12. **リッチ완료報告**（「완료報告フォーマット」参照）
13. **失敗時の自動再計画**（テスト/CI 失敗時のみ）:
    - テスト実行結果を確認
    - 失敗した경우: 수정タスク案を state に保存し、承認コマンド経由で Plans.md に追加（「失敗タスクの自動再チケット化」参照）
    - 成功した경우: 次タスクへ進む

### Parallel モード（2〜3 件時の自動選択 / `--parallel N` で強制）

`[P]` マーク付きタスクを N ワーカーで並列実行。
`--parallel N` で明示指定した경우は、タスク数に関係なく이モードを使用。
同一ファイルへの書き込みが競合する경우は git worktree で分離。

### Codex モード（`--codex` 明示時のみ）

公式プラグイン `codex-plugin-cc` の companion 経由で Codex CLI にタスクを委託する。

```bash
# タスク委託（書き込み可能）
bash scripts/codex-companion.sh task --write "タスク내容"

# stdin 経由（큰プロンプト향け）
CODEX_PROMPT=$(mktemp /tmp/codex-prompt-XXXXXX.md)
# タスク내容を書き出し
cat "$CODEX_PROMPT" | bash scripts/codex-companion.sh task --write
rm -f "$CODEX_PROMPT"

# 이전スレッドの続行
bash scripts/codex-companion.sh task --resume-last --write "계속をやって"
```

companion は App Server Protocol 経由で Codex と通信し、
Job 管理・thread resume・構造化出力を提供する。
結果を検証し、品質基準を満たさない경우は自力で수정。

### Breezing モード（4 件이상で自動選択 / `--breezing` で強制）

Lead / Worker / Reviewer の役割分離でチーム実行する。
Codex では `spawn_agent`, `wait`, `send_input`, `resume_agent`, `close_agent`
を使った native subagent orchestration を前提にし、
오래된 TeamCreate / TaskCreate ベースの説明を採らない。

**権限ポリシー**:
- 現行の shipped default は `bypassPermissions`
- `--auto-mode` は互換な親セッション향けの opt-in rollout フラグとして扱う
- `permissions.defaultMode` や agent frontmatter の `permissionMode` 에는未文書化の `autoMode` 値を書かない

> **CC v2.1.69+**: nested teammates はプラットフォーム側で금지されるため、
> Worker/Reviewer プロンプト에는冗長な nested 防止文言を追加しない。

```
Lead (this agent)
├── Worker (task-worker agent) — 実装担当
└── Reviewer (code-reviewer agent) — レビュー担当
```

**Phase A: Pre-delegate（準備）**:
1. Plans.md を読み込み、대象タスクを특정
2. 依存グラフを解析し、実行順序を決定（Depends カラム）
3. 各タスクの effort スコアリング（ultrathink 注入判定）
4. `scripts/generate-sprint-contract.sh` で `sprint-contract.json` を생성
5. `scripts/enrich-sprint-contract.sh` で Reviewer 観点を加え、`scripts/ensure-sprint-contract-ready.sh` で未承認なら停止

**Phase B: Delegate（Worker spawn → レビュー → cherry-pick）**:

各タスクについて이하를**逐次**実行する（依存順）:

> **API 注記**: 以下は Claude Code の API 構文で記述。
> Codex 환경では `Agent(...)` → `spawn_agent(...)`, `SendMessage(...)` → `send_input(...)` に読み替え。
> 詳細は `team-composition.md` の API マッピング表を参照。

```
for task in execution_order:
    # B-1. sprint-contract を생성
    contract_path = bash("scripts/generate-sprint-contract.sh {task.number}")
    contract_path = bash("scripts/enrich-sprint-contract.sh {contract_path} --check \"DoD を reviewer 観点で確認\" --approve")
    bash("scripts/ensure-sprint-contract-ready.sh {contract_path}")

    # B-2. Worker spawn（フォアグラウンド、worktree 分離）
    # Agent tool の戻り値に agentId が含まれる — 수정ループで SendMessage に使用
    Plans.md: task.status = "cc:WIP"  # 착수時に갱新（未착수タスクは cc:TODO のまま）

    worker_result = Agent(
        subagent_type="claude-code-harness:worker",
        prompt="タスク: {task.내容}\nDoD: {task.DoD}\ncontract_path: {contract_path}\nmode: breezing",
        isolation="worktree",
        run_in_background=false  # フォアグラウンド실행 → Worker 완료まで待機
    )
    worker_id = worker_result.agentId  # SendMessage 用に유지
    # worker_result 에는 {commit, worktreePath, files_changed, summary} が含まれる

    # B-3. Lead がレビュー実行（Codex exec 優선）
    diff_text = git("-C", worker_result.worktreePath, "show", worker_result.commit)
    verdict = codex_exec_review(diff_text) or reviewer_agent_review(diff_text)
    profile = jq(contract_path, ".review.reviewer_profile")
    review_input = "review-output.json"
    if profile == "runtime":
        review_input = bash("cd {worker_result.worktreePath} && scripts/run-contract-review-checks.sh {contract_path}")
        runtime_verdict = jq(review_input, ".verdict")
        if runtime_verdict == "REQUEST_CHANGES":
            verdict = "REQUEST_CHANGES"
        elif runtime_verdict == "DOWNGRADE_TO_STATIC":
            pass  # runtime 検証コマンド없음 → static verdict を그まま使う
    if profile == "browser":
        # browser artifact は PENDING_BROWSER scaffold を생성。
        # 実際の browser 実行は reviewer agent が後続で担当する。
        # review-result 에는 static review の verdict を書く（PENDING_BROWSER ではなく）。
        browser_artifact = bash("scripts/generate-browser-review-artifact.sh {contract_path}")
        # browser artifact は参照用に保存するが、review-result の verdict は static のまま
    # review_input が DOWNGRADE_TO_STATIC の경우は static review 結果を使う
    if review_input != "review-output.json" and jq(review_input, ".verdict") == "DOWNGRADE_TO_STATIC":
        review_input = "review-output.json"  # static review の結果にフォールバック
    bash("scripts/write-review-result.sh {review_input} {latest_commit}")

    # B-4. 수정ループ（REQUEST_CHANGES 時、最大 3 回）
    # Worker はフォアグラウンドで완료済みだが、SendMessage で再개可能
    # （CC: SendMessage(to: agentId) / Codex: resume_agent(agent_id) + send_input）
    review_count = 0
    latest_commit = worker_result.commit
    while verdict == "REQUEST_CHANGES" and review_count < 3:
        SendMessage(to=worker_id, message="指摘내容: {issues}\n수정して amend してください")
        # Worker が수정 → amend → 갱新された commit hash を返す
        updated_result = wait_for_response(worker_id)
        latest_commit = updated_result.commit
        diff_text = git("-C", worker_result.worktreePath, "show", latest_commit)
        verdict = codex_exec_review(diff_text) or reviewer_agent_review(diff_text)
        review_count++

    # B-5. APPROVE → main に cherry-pick
    if verdict == "APPROVE":
        git cherry-pick --no-commit {latest_commit}  # worktree → main
        git commit -m "{task.내容}"
        Plans.md: task.status = "cc:완료 [{hash}]"
    else:
        → ユーザーにエスカレーション

    # B-6. Progress feed
    print("📊 Progress: Task {completed}/{total} 완료 — {task.내容}")
```

### Sprint Contract

`sprint-contract` は「이タスクを무엇で合格にするか」を機械でも人でも同じ意味で読める形にする작은契約ファイルです。
既定の保存선は `.claude/state/contracts/<task-id>.sprint-contract.json` です。

```bash
scripts/generate-sprint-contract.sh 32.1.1
```

생성物에는次を含めます。

- `checks`: DoD を分解した確認項目
- `non_goals`: 今回やらない것
- `runtime_validation`: test, lint, typecheck 등の検証コマンド
- `browser_validation`: browser reviewer が残すべき UI フロー検証項目
- `browser_mode`: `scripted` または `exploratory`
- `route`: browser reviewer が `playwright` / `agent-browser` / `chrome-devtools` の어느を使うか
- `risk_flags`: `needs-spike`, `security-sensitive`, `ux-regression` 등
- `reviewer_profile`: `static`, `runtime`, `browser`

**Phase C: Post-delegate（統合・報告）**:
1. 全タスクの commit log を集計
2. **リッチ완료報告**（「완료報告フォーマット」の Breezing テンプレート）を出力
3. Plans.md の最종確認（全タスク cc:완료 になっているか）

## CI 失敗時の대応

CI が失敗した경우:

1. ログを確認してエラーを특정
2. 수정を実施
3. 同一原因で 3 回失敗したら自動수정ループを停止
4. 失敗ログ・試みた수정・残る論点をまとめてエスカレーション

## 失敗タスクの自動再チケット化

タスク완료後にテスト/CI が失敗した경우、수정タスク案を自動생성し、承認後に Plans.md へ反映する:

### トリガー条件

| 条件 | アクション |
|------|----------|
| `cc:완료` 後にテスト失敗 | 수정タスク案を state に保存し、承認を待つ |
| CI 失敗（3回未満） | 수정を実施し、失敗カウントをインクリメント |
| CI 失敗（3回目） | 수정タスク案を제시 + エスカレーション |

### 수정タスクの自動생성

1. 失敗原因を分類（syntax_error / import_error / type_error / assertion_error / timeout / runtime_error）
2. `.claude/state/pending-fix-proposals.jsonl` に수정タスク案を保存:
   - 番号: 元タスク番号 + `.fix` サフィックス（例: `26.1.fix`）
   - 내容: `fix: [元タスク名] - [失敗原因カテゴリ]`
   - DoD: テスト/CI が通る것
   - Depends: 元タスク番号
3. ユーザーが `approve fix <task_id>` を送ると Plans.md に `cc:TODO` で追加
4. `reject fix <task_id>` で提案を破棄。pending が1件だけのときは `yes` / `no` でも応答可能

## レビューループ

実装완료後（ステップ 5 の後）に自動実行される品質検証ステージ。
**全モード共通**（Solo / Parallel / Breezing）で統一的に適用される。
Parallel モードでは各 Worker が step 10（외部レビュー受付）として同じループ실행する。

### レビュー実行の우선순위

```
1. Codex exec（優선）
   ↓ codex コマンドが존재しない or 타임아웃（120s）
2. 내部 Reviewer agent（フォールバック）
```

### APPROVE / REQUEST_CHANGES の判定基準

レビュアー에는이하의임계값基準を渡し、**이 기준のみで**で verdict を判定させる。
基準외の改善提案は `recommendations` として返すが、verdict 에는영향 없음。

| 重要度 | 定義 | verdict への影響 |
|--------|------|-----------------|
| **critical** | セキュリティ脆弱性、データ損失リスク、本番障害の可能性 | 1 件でも → REQUEST_CHANGES |
| **major** | 既存機能の破壊、仕様와의명확한矛盾、テスト不通過 | 1 件でも → REQUEST_CHANGES |
| **minor** | 命名改善、コメント不足、スタイル不統一 | verdict に영향 없음 |
| **recommendation** | ベストプラクティス提案、将来の改善案 | verdict に영향 없음 |

> **重要**: minor / recommendation のみの경우は **必ず APPROVE** を返す것。
> 「あったほうが良い改善」は REQUEST_CHANGES の理由にならない。

### Codex exec レビュー（公式プラグイン経由）

タスク시작 시の HEAD を `BASE_REF` として유지し、그 ref 와의差分をレビュー대象にする。
公式プラグイン `codex-plugin-cc` の companion review を使用する。

```bash
# タスク시작 시に base ref を記録（Step 2 の cc:WIP 갱新前に実行）
BASE_REF=$(git rev-parse HEAD)

# ... 実装완료後 ...

# 公式プラグインの構造化レビュー실행
bash scripts/codex-companion.sh review --base "${BASE_REF}"
REVIEW_EXIT=$?
```

**verdict マッピング**（公式プラグイン → Harness 形式）:

公式プラグインは `review-output.schema.json` 準拠の構造化出力を返す。
Harness の verdict 形式への変換ルール:

| 公式 plugin | Harness | verdict 影響 |
|---|---|---|
| `approve` | `APPROVE` | - |
| `needs-attention` | `REQUEST_CHANGES` | - |
| `findings[].severity: critical` | `critical_issues[]` | 1件でも → REQUEST_CHANGES |
| `findings[].severity: high` | `major_issues[]` | 1件でも → REQUEST_CHANGES |
| `findings[].severity: medium/low` | `recommendations[]` | verdict に영향 없음 |

AI Residuals スキャンは引き계속 `scripts/review-ai-residuals.sh` 실행し、
companion review の結果と合わせて最종 verdict を判定する。

```bash
# AI Residuals スキャン（companion review と並行実行可能）
AI_RESIDUALS_JSON="$(bash scripts/review-ai-residuals.sh --base-ref "${BASE_REF}" 2>/dev/null || echo '{"tool":"review-ai-residuals","scan_mode":"diff","base_ref":null,"files_scanned":[],"summary":{"verdict":"APPROVE","major":0,"minor":0,"recommendation":0,"total":0},"observations":[]}')"
```

### 내部 Reviewer agent フォールバック

Codex exec が使えない경우（`command -v codex` が失敗、または exit code ≠ 0）:

```
Agent tool: subagent_type="reviewer"
prompt: "이하의変갱をレビューしてください。判定基準: critical/major → REQUEST_CHANGES、minor/recommendation のみ → APPROVE。diff: {git diff ${BASE_REF}}"
```

Reviewer agent は Read-only（Write/Edit/Bash 무効）で安全にレビュー실행する。

### 수정ループ（REQUEST_CHANGES 時）

```
review_count = 0
MAX_REVIEWS = 3

while verdict == "REQUEST_CHANGES" and review_count < MAX_REVIEWS:
    1. レビュー指摘を解析（critical / major のみ대象）
    2. 各指摘に대して수정を実装
    3. 재レビュー실행（同じ判定基準・同じ우선순위）
    review_count++

if review_count >= MAX_REVIEWS and verdict != "APPROVE":
    → ユーザーにエスカレーション
    → 「3 回수정しましたが이하의 critical/major 指摘が残っています」+ 指摘一覧を表示
    → ユーザー判断を待つ（続行 / 中断）
```

### Breezing モードでの適用

Breezing モードでは **Lead** がレビューループ실행する（上記 Phase B 参照）:

1. Worker が worktree 내で実装・commit → Lead に結果返却
2. Lead が Codex exec でレビュー（優선）/ Reviewer agent（フォールバック）
3. REQUEST_CHANGES → Lead が SendMessage で Worker に수정 지시 → Worker が amend
4. 수정後、再レビュー（最大 3 回）
5. APPROVE → Lead が main に cherry-pick → Plans.md を `cc:완료 [{hash}]` に갱新

## 완료報告フォーマット

タスク완료時（`cc:완료` + commit 後）に自動出力される視覚的サマリ。
非専門家にも変갱내容と影響が伝わる것を目的とする。

### テンプレート

```
┌─────────────────────────────────────────────┐
│  ✓ Task {N} 완료: {タスク名}                    │
├─────────────────────────────────────────────┤
│                                              │
│  ■ 무엇をしたか                                 │
│    • {変갱내容 1}                              │
│    • {変갱내容 2}                              │
│                                              │
│  ■ 무엇が変わるか                                │
│    Before: {旧動作}                            │
│    After:  {新動作}                            │
│                                              │
│  ■ 変갱ファイル ({N} files)                    │
│    {ファイルパス 1}                             │
│    {ファイルパス 2}                             │
│                                              │
│  ■ 残りの課題                                  │
│    • Task {X} ({status}): {내容}  ← Plans.md  │
│    • Task {Y} ({status}): {내容}  ← Plans.md  │
│    （Plans.md に {M} 件の未완료タスクあり）       │
│                                              │
│  commit: {hash} | review: {APPROVE}           │
└─────────────────────────────────────────────┘
```

### 생성ルール

1. **무엇をしたか**: `git diff --stat HEAD~1` と commit message から自動抽出。技術用語は最小限にし、動詞で시작る
2. **무엇が変わるか**: タスクの「내容」と「DoD」から Before/After を推論。ユーザー体験の変化を重視
3. **変갱ファイル**: `git diff --name-only HEAD~1` から取得。5 ファイル超は생략して件数表示
4. **残りの課題**: Plans.md の `cc:TODO` / `cc:WIP` タスク목록表示。Plans.md に記載済みかどうかを明示
5. **review**: レビュー結果（APPROVE / REQUEST_CHANGES → APPROVE）を表示

### Parallel モードでの報告

- **1 タスク**（`--parallel` 強制時）: Solo テンプレートを使用
- **복수タスク**: Breezing 集約テンプレートを使用（下記参照）

### Breezing モードでの報告

全タスク완료後にまとめて出力。各タスクは簡略版（무엇をしたか + commit hash のみ）で一覧し、
最後に全体サマリ（合計変갱ファイル数 + 残り課題）を出力する:

```
┌─────────────────────────────────────────────┐
│  ✓ Breezing 완료: {N}/{M} タスク             │
├─────────────────────────────────────────────┤
│                                              │
│  1. ✓ {タスク名 1}            [{hash1}]      │
│  2. ✓ {タスク名 2}            [{hash2}]      │
│  3. ✓ {タスク名 3}            [{hash3}]      │
│                                              │
│  ■ 全体の変갱                                 │
│    {N} files changed, {A} insertions(+),     │
│    {D} deletions(-)                          │
│                                              │
│  ■ 残りの課題                                  │
│    Plans.md に {K} 件の未완료タスクあり         │
│    • Task {X}: {내容}                         │
│                                              │
└─────────────────────────────────────────────┘
```

## 関連スキル

- `harness-plan` — 実行するタスクを計画する
- `harness-sync` — 実装と Plans.md を同期する
- `harness-review` — 実装のレビュー
- `harness-release` — バージョンバンプ・リリース
