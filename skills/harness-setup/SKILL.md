---
name: harness-setup
description: "Harness v3 통합 세팅 스킬. 프로젝트 초기화·도구 설정·2에이전트 구성·메모리 설정·공개 skill mirror 동기화를 담당. 다음으로起動: 세팅, 초기화, 새 프로젝트, CI세팅, codex CLI세팅, harness-mem, 에이전트 설정, symlink, mirror, harness-setup. 구현·리뷰·릴리스·기획에는 사용하지 않음."
description-en: "Unified setup skill for Harness v3. Project init, tool setup, 2-agent config, memory setup, and public skill mirror sync. Use when user mentions: setup, initialization, new project, CI setup, codex CLI setup, harness-mem, agent setup, symlinks, mirrors, harness-setup. Do NOT load for: implementation, code review, release, or planning."
description-ja: "Harness v3 통합 세팅 스킬. 프로젝트 초기화·도구 설정·2에이전트 구성·메모리 설정·공개 skill mirror 동기화를 담당. 다음으로起動: 세팅, 초기화, 새 프로젝트, CI세팅, codex CLI세팅, harness-mem, 에이전트 설정, symlink, mirror, harness-setup. 구현·리뷰·릴리스·기획에는 사용하지 않음."
allowed-tools: ["Read", "Write", "Edit", "Grep", "Glob", "Bash"]
argument-hint: "[init|ci|codex|harness-mem|mirrors|agents|localize]"
effort: medium
---

# Harness Setup (v3)

Harness v3의 통합 세팅 스킬입니다.
다음 기존 스킬을 통합합니다:

- `setup` — 통합 세팅 허브
- `harness-init` — 프로젝트 초기화
- `harness-update` — Harness 업데이트
- `maintenance` — 파일 정리·클린업

## Quick Reference

| 서브명령어 | 동작 |
|------------|------|
| `harness-setup init` | 새 프로젝트 초기화(CLAUDE.md + Plans.md + hooks)|
| `harness-setup ci` | CI/CD 파이프라인 설정 |
| `harness-setup codex` | Codex CLI 설치·설정 |
| `harness-setup harness-mem` | harness-mem 통합·메모리 설정 |
| `harness-setup mirrors` | skills-v3/ → 공개 mirror bundle 업데이트 |
| `harness-setup agents` | agents-v3/ 에이전트 설정 |
| `harness-setup localize` | CLAUDE.md 규칙의 로컬라이즈 |

## 서브명령어 상세

### init — 프로젝트 초기화

새 프로젝트에 Harness v3를 도입합니다.

**생성 파일**:
```
project/
├── CLAUDE.md            # 프로젝트 설정
├── Plans.md             # 작업 관리(빈 템플릿)
├── .claude/
│   ├── settings.json    # Claude Code 설정
│   └── hooks.json       # 훅 설정(v3 심)
└── hooks/
    ├── pre-tool.sh      #薄い 심(→ core/src/index.ts)
    └── post-tool.sh     #薄い 심(→ core/src/index.ts)
```

**플로우**:
1. 프로젝트 종류를 감지(Node.js/Python/Go/Rust/기타)
2. 최소한의 CLAUDE.md를 생성
3. Plans.md 템플릿을 생성
4. hooks.json을 배치

### ci — CI/CD 설정

GitHub Actions 워크플로우를 설정합니다.

```yaml
# .github/workflows/ci.yml 생성 예
name: CI
on:
  push:
    branches: [main]
  pull_request:
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm ci && npm test
```

### codex — Codex CLI 설정

```bash
# 설치 확인
which codex || npm install -g @openai/codex

# 타임아웃 명령 확인(macOS)
TIMEOUT=$(command -v timeout || command -v gtimeout || echo "")
# macOS의 경우: brew install coreutils
```

**사용 패턴**(공식 플러그인 경유):
```bash
bash scripts/codex-companion.sh task --write "작업 내용"
# 또는 stdin 경유
cat /tmp/prompt.md | bash scripts/codex-companion.sh task --write
```

### harness-mem — 메모리 설정

Unified Harness Memory의 설정을 수행합니다.

```bash
# 메모리 디렉토리 생성
mkdir -p .claude/agent-memory/claude-code-harness-worker
mkdir -p .claude/agent-memory/claude-code-harness-reviewer

# MEMORY.md 템플릿 배치
cat > .claude/agent-memory/claude-code-harness-worker/MEMORY.md << 'EOF'
# Worker Agent Memory

## Project Context
[프로젝트 개요]

## Patterns
[학습 패턴]
EOF
```

### mirrors — 공개 skill bundle 동기화

Windows의 `core.symlinks=false`에서는 repository symlink가 일반 파일이 되어, `harness-*` skill이 command 목록에 나오지 않을 수 있습니다. 공개 bundle은 실 디렉토리 mirror로 동기화합니다.

```bash
./scripts/sync-v3-skill-mirrors.sh
./scripts/sync-v3-skill-mirrors.sh --check
```

업데이트 대상:

- `skills/`
- `codex/.codex/skills/`
- `opencode/skills/`

### agents — 에이전트 설정

agents-v3/의 3에이전트 구성을 설정합니다.

```
agents-v3/
├── worker.md      # 구현 담당(task-worker + codex-implementer + error-recovery)
├── reviewer.md    # 리뷰 담당(code-reviewer + plan-critic)
└── scaffolder.md  # 발사臺 담당(project-analyzer + scaffolder)
```

### localize — 규칙 로컬라이즈

`.claude/rules/`의 규칙을 현 프로젝트에 맞춥니다.

```bash
# 규칙 목록 확인
ls .claude/rules/

# 프로젝트 고유 규칙의 추가
cat >> .claude/rules/project-rules.md << 'EOF'
# Project-Specific Rules
[프로젝트 고유 규칙]
EOF
```

## Plugin 설치(v2.1.71+ Marketplace)

v2.1.71에서 Marketplace의 안정성이 크게 개선되었습니다.

### 권장 설치 방식

```bash
# @ref 형식으로 버전 고정(권장)
claude plugin install owner/repo@v3.5.0

# 최신판
claude plugin install owner/repo
```

`owner/repo@vX.X.X` 형식을 권장합니다. `@ref` 파서 수정을 통해, 태그·브랜치·커밋 해시 어느 것이나 정확히 해결됩니다.

### 업데이트

```bash
claude plugin update owner/repo
```

v2.1.71에서 update시의 merge conflict가 수정을 통해, 안정적인 업데이트가 가능해졌습니다.

### 그 외의 개선점

- MCP server 중복 배제: 동일 MCP 서버의 중복 등록을 자동 방지
- `/plugin uninstall`가 `settings.local.json`을 사용: 사용자 로컬 설정에 정확히 반영

## Maintenance — 파일 정리

정기 메인터넌스 작업:

| 작업 | 명령 |
|--------|---------|
| 오래된 로그 삭제 | `find .claude/logs -mtime +30 -delete` |
| Plans.md 압축 | 완료 작업을 아카이브 섹션으로 이동 |
| 오래된 트레이스 삭제 | `tail -1000 .claude/state/agent-trace.jsonl > /tmp/trace && mv /tmp/trace .claude/state/agent-trace.jsonl` |

## 관련 스킬

- `harness-plan` — 세팅 후 프로젝트 계획을 작성
- `harness-work` — 세팅 후 작업을 실행
- `harness-review` — 세팅 설정을 리뷰
