---
name: harness-release
description: "Harness v3 통합 릴리스 스킬. CHANGELOG·버전 밤프·태그·GitHub Release·mirror 동기화·검증을 자동화. 다음으로 실행: 릴리스, 버전 밤프, 태그 생성, 공개, /harness-release. 구현·코드 리뷰·기획·세팅에는 사용하지 않음."
description-en: "Unified release skill for Harness v3. CHANGELOG, version bump, tag, GitHub Release, mirror sync, and validation automation. Use when user mentions: release, version bump, create tag, publish, /harness-release. Do NOT load for: implementation, code review, planning, or setup."
description-ja: "Harness v3 통합 릴리스 스킬. CHANGELOG・버전 밤프・태그・GitHub Release・mirror 동기화・검증을 자동화. 다음으로 실행: 릴리스, 버전 밤프, 태그 생성, 공개, /harness-release. 구현・코드 리뷰・기획・세팅에는 사용하지 않음."
allowed-tools: ["Read", "Write", "Edit", "Bash"]
argument-hint: "[patch|minor|major|--dry-run|--announce|--complete]"
context: fork
effort: high
---

# Harness Release (v3)

Harness v3의 통합 릴리스 스킬입니다.
다음 기존 스킬을 통합합니다:

- `release-har` -- 범용 릴리스 자동화
- `x-release-harness` -- Harness 전용 릴리스 자동화
- `handoff` -- PM에게 핸드오프·완료 보고

## Quick Reference

```bash
/release          # 대화형（버전 종류를 확인）
/release patch    # 패치 버전 밤프（버그 수정）
/release minor    # 마이너 버전 밤프（신규 기능）
/release major    # 메이저 버전 밤프（파괴적 변경）
/release --dry-run   # 미리보기만（실행하지 않음）
/release --announce  # X (Twitter) 고지도 실행
/release --complete  # 릴리스 완료 표시（태그 후의 마무리）
```

## Release-only policy

- 일반 PR: `VERSION` / `.claude-plugin/plugin.json` / versioned `CHANGELOG.md` entry는 건드리지 않음
- 일반 PR의 변경 이력: `CHANGELOG.md`의 `[Unreleased]`에 추가
- `/release` 실행 시만 version bump, versioned CHANGELOG entry, tag / GitHub Release를 한꺼번에 업데이트
- `/release --dry-run`에서도 본번 실행과 같은 preflight를 통과하고, 공개 전의 위험 신호를 먼저 차단한다

## 브랜치 정책

- **단독 개발**: main에 직접 push 허용（CI가품질 게이트로 기능）
- **공동 개발**: PR 경유 머지가 필수
- force push（`--force` / `--force-with-lease`）는 항상 금지

## 버전 판단 기준（SemVer）

`.claude/rules/versioning.md`에 따른 판단 플로우차트:

```
기존 동작이 망가지는가?
├─ Yes → major
└─ No → 사용자가 새로운 것을 할 수 있게 되는가?
    ├─ Yes → minor
    └─ No → patch
```

| 변경 종류 | 버전 | 예 |
|-----------|----------|-----|
| 스킬 정의의 문언 수정·追記 | **patch** | 템플릿 미수정 |
| hooks/scripts의 버그 수정 | **patch** | 에스케이프修正 |
| 새 스킬/플래그/에이전트 추가 | **minor** | `--dual`, 새 스킬 |
| CC 새 버전 호환 대응 | **minor** | CC v2.1.90 대응 |
| 파괴적 변경（기존 스킬 폐지, 포맷 비호환） | **major** | Plans.md v1 삭제 |

**배치 릴리스 권장**:같은 날 복수 변경이 있는 경우 1개의 minor로 정리.같은 날 2회 이상의 minor 밤프 금지.

## NPM 배포에 대해

이 프로젝트는 Claude Code 플러그인이며, npm 패키지로서는 배포하지 않습니다.
루트에 `package.json`는 존재하지 않음（`core/package.json`은 내부 TypeScript 빌드용）.
버전 관리 대상은 다음 2파일만:

- `VERSION` -- 정본
- `.claude-plugin/plugin.json` -- 플러그인 マニフェスト

## 배포 면과 Mirror 동기화

`skills-v3/`가 SSOT（Single Source of Truth）입니다. 다음 3개의 배포 면이 mirror로 동기화됩니다:

| 배포 면 |パス | 대상 사용자 |
|--------|------|------------|
| Claude | `skills/harness-release/` | Claude Code 사용자 |
| Codex | `codex/.codex/skills/harness-release/` | Codex CLI 사용자 |
| OpenCode | `opencode/skills/harness-release/` | OpenCode 사용자 |

**중요**: `skills-v3/`를 편집하면, 릴리스전에 반드시 mirror를 동기화하세요:

```bash
./scripts/sync-v3-skill-mirrors.sh
```

검증만（쓰기 변경 없음）:

```bash
./scripts/sync-v3-skill-mirrors.sh --check
```

## 일본어 대응（i18n）

스킬의 description 필드를 일본어·영어 전환할 수 있습니다. 릴리스 전에 로케일 설정이 의도대로인지 확인하세요:

```bash
# 일본어로 설정（description-ja → description）
./scripts/i18n/set-locale.sh ja

# 영어로 설정（description-en → description）
./scripts/i18n/set-locale.sh en
```

현재 기본값: description은 일본어（`description-ja`와 동일）.`description-en`은 영어 백업으로 항상 유지.

## 실행 플로우

### Phase 0: Pre-flight 체크（필수）

```bash
# 1. 필수 도구 확인
command -v gh &>/dev/null || echo "gh 없음: GitHub Release는 스킵"
command -v jq &>/dev/null || echo "jq 없음: plugin.json 업데이트에 필요"

# 2. vendor-neutral preflight（본 실행 / dry-run 공통）
bash scripts/release-preflight.sh

# 3. 플러그인 구조 검증
bash tests/validate-plugin.sh

# 4.整合성 체크
bash scripts/ci/check-consistency.sh

# 5. mirror 동기화 상태 확인
bash scripts/sync-v3-skill-mirrors.sh --check
```

`scripts/release-preflight.sh`는 다음을 검증합니다:

- working tree가 clean인지
- `CHANGELOG.md`에 `[Unreleased]`가 있는지
- `.env.example`와 `.env`의 차이（managed secrets 전제에서는 warning  끝)
- `healthcheck` / `preflight` 명령（있으면 실행）
- `agents/` / `core/` / `hooks/` / `scripts/`의 shipped surface에 debug / mock / placeholder 잔해가 있는지
- CI 상태（취득 가능한 경우）

환경 변수로 리포지토리마다 조정 가능:

- `HARNESS_RELEASE_PROJECT_ROOT`
- `HARNESS_RELEASE_HEALTHCHECK_CMD`
- `HARNESS_RELEASE_CI_STATUS_CMD`

상세: [docs/release-preflight.md](${CLAUDE_SKILL_DIR}/../../docs/release-preflight.md)

### Phase 1: 현재 버전 취득

```bash
CURRENT=$(cat VERSION 2>/dev/null)
echo "현재 버전: $CURRENT"
```

### Phase 2: 새 버전 산출

`scripts/sync-version.sh`는 patch 밤프만 대응. minor / major는 수동으로 VERSION을書재작성:

```bash
# patch 밤프（x.y.Z → x.y.(Z+1)）
./scripts/sync-version.sh bump

# minor 밤프（수동: x.Y.z → x.(Y+1).0）
CURRENT=$(cat VERSION)
MAJOR=$(echo "$CURRENT" | cut -d. -f1)
MINOR=$(echo "$CURRENT" | cut -d. -f2)
NEW_VERSION="$MAJOR.$((MINOR + 1)).0"
echo "$NEW_VERSION" > VERSION
./scripts/sync-version.sh sync

# major 밤프（수동: X.y.z → (X+1).0.0）
CURRENT=$(cat VERSION)
MAJOR=$(echo "$CURRENT" | cut -d. -f1)
NEW_VERSION="$((MAJOR + 1)).0.0"
echo "$NEW_VERSION" > VERSION
./scripts/sync-version.sh sync
```

`sync-version.sh sync`는 `VERSION`의 값을 `.claude-plugin/plugin.json`에 반영합니다.

### Phase 3: CHANGELOG 업데이트

릴리스 entry는, 일반 PR에서 쌓은 `[Unreleased]`의 변경을 versioned section으로 확정합니다.

**상세 Before/After 포맷**（일본어）で記述します.
각 기능을 번호 있는 섹션에 나누고,「지금까지」와「앞으로」를 구체적 예쁘로 설명합니다.

```markdown
## [X.Y.Z] - YYYY-MM-DD

### 테마: [변경 전체를 한마디로]

**[사용자에게 가치를 1〜2문으로]**

---

#### 1. [기능명]

**지금까지**: [이전 동작를 구체적으로. 사용자가「매우 공감됨」이라고 느끼는 과제 묘사]

**앞으로**: [새 동작를 구체적으로. 무엇이 해결되는지]

```
[실제 출력 예나 명령 예]
```

#### 2. [다음 기능명]

**지금까지**: ...

**앞으로**: ...
```

**CC 버전 통합시의 패턴**: 일반적인「지금까지 / 앞으로」가 아니라「CC의 업뎃 → Harness에서의 활용」형을 사용합니다.
상세는 `.claude/rules/github-release.md`의「CC 버전 통합시의 CHANGELOG 패턴」을 참조.

**작성 방법의 규칙**:

| 규칙 | 설명 |
|--------|-------|
| 언어 | **일본어** |
| 각 기능을 독립 섹션에 | `#### N. 기능명`으로 번호 있음 |
| 「지금까지」는 과제 묘사 | 사용자가 체험하던 불편을 구체적으로 작성 |
| 「앞으로」는 해결을 표시 | 무엇이 어떻게 달라지는지 + 구체적 예（코드/출력） |
| 구체적 예를 반드시 포함 | 명령 예, 출력 예, Plans.md의 스니펫 등 |
| 기술적 상세는 최소한으로 | 파일명이나 스텝 번호는「앞으로」의 보충으로서 |
| 길어도 OK | 각 기능 3〜10행. 읽기 쉽기가 최우선 |

`[Unreleased]` 섹션은 비우지 않고, 다음 릴리스를 위해 남겨둡니다:

```markdown
## [Unreleased]

## [X.Y.Z] - YYYY-MM-DD
...
```

### Phase 4: 버전 파일 업데이트

```bash
# VERSION은 Phase 2에서 업데이트済み
# plugin.json을 동기화
./scripts/sync-version.sh sync

# 동기화 확인
./scripts/sync-version.sh check
```

### Phase 5: Mirror 동기화

```bash
# skills-v3 → skills, codex, opencode로의 mirror 동기화
./scripts/sync-v3-skill-mirrors.sh

# 동기화 확인
./scripts/sync-v3-skill-mirrors.sh --check
```

### Phase 6: 커밋 & 태그

```bash
NEW_VERSION=$(cat VERSION)

# 스테이징（대상 파일을 명시적으로 지정）
git add VERSION .claude-plugin/plugin.json CHANGELOG.md
git add skills/ codex/.codex/skills/ opencode/skills/

git commit -m "chore: release v$NEW_VERSION"
git tag -a "v$NEW_VERSION" -m "Release v$NEW_VERSION"
```

### Phase 7: 푸시

```bash
git push origin main --tags
```

**주의**: `.github/workflows/release.yml`이 태그 푸시를 감지하고, GitHub Release가 이미 존재하면 CHANGELOG에서 자동 생성하는 세이프티넷이 작동한다. 수동으로 GitHub Release를 먼저 생성하면, 워크플로우는 자동 스킵한다.

### Phase 8: GitHub Release 생성

```bash
NEW_VERSION=$(cat VERSION)

gh release create "v$NEW_VERSION" \
  --title "v$NEW_VERSION - 제목" \
  --notes "$(cat <<'EOF'
## What's Changed

**[변경 개요（영어）]**

### Before / After

| Before | After |
|--------|-------|
| Previous state | New state |

---

## Added

- **Feature**: Description

## Changed

- **Change**: Description

## Fixed

- **Fix**: Description

---

Generated with [Claude Code](https://claude.com/claude-code)
EOF
)"
```

GitHub Release Notes의 규칙:
- 언어: **영어**（공개 리포지토리 때문)
- 필수: `## What's Changed`, 굵은 개요, Before / After 테이블, 푸터
-상세 포맷: `.claude/rules/github-release.md`를 참조

릴리스 노트 검증:

```bash
./scripts/validate-release-notes.sh "v$NEW_VERSION"
```

### Phase 9: 릴리스 완료 표시

```bash
git commit --allow-empty -m "chore: mark v$NEW_VERSION release complete"
git push origin main
```

이 빈 커밋은「릴리스 작업이 모두 완료됨」을 명확히 표시하는 마커입니다.

### Phase 10: 고지（`--announce` 지정 시만）

`/x-announce` 스킬을 호출하여 X (Twitter)에의 고지 스레드를 생성합니다:

```
Skill: x-announce
Args: v$NEW_VERSION
```

게시 텍스트 5개 + Gemini이미지 5개를 1회 출력합니다.

## `--dry-run` 모드

`--dry-run`은 다음을 실행하고, 실제 변경은 하지 않습니다:

1. Pre-flight 체크（Phase 0）을 **전부 실행**
2. 버전 산출를 표시（기록 안함）
3. CHANGELOG의 드래프트를 표시（기록 안함）
4. GitHub Release Notes의 드래프트를 표시（생성 안함）
5. mirror 동기화의 차분을 표시（기록 안함）

스킵됨: VERSION/plugin.json 쓰기, git commit/tag/push, GitHub Release 생성, 고지

## `--complete` 모드

태그 작성 후에「릴리스 완료」의 표시만 수행:

```bash
/release --complete
```

Phase 9만 실행합니다. GitHub Release의 생성 누락이 없는こ.과を確認한 뒤, 완료 커밋을 찍습니다.

## 데그레 체크리스트

릴리스전에 다음의 데그레를 확인합니다:

| 체크 항목 | 확인 방법 | 비고 |
|------------|---------|------|
| 플러그인 구조 | `tests/validate-plugin.sh` | plugin.json, 스킬, 훅, 스크립트 검증 |
|整合성 | `scripts/ci/check-consistency.sh` | 템플릿, 버전, mirror, CHANGELOG |
| Mirror 동기화 | `scripts/sync-v3-skill-mirrors.sh --check` | skills-v3와 3 배포 면의 일치 |
| Preflight | `scripts/release-preflight.sh` | working tree, CHANGELOG, CI, 잔해 |
| 릴리스 노트 | `scripts/validate-release-notes.sh vX.Y.Z` | GitHub Release의 포맷 검증 |
| VERSION 동기화 | `scripts/sync-version.sh check` | VERSION와 plugin.json의 일치 |
| 가드레일 | `core/src/guardrails/rules.ts`의 R01-R13 | TypeScript 규칙의 건전성 |
| 태그 연속성 | `git tag --sort=-version:refname \| head -5` |누락이 없다면 |
| 로케일 | description와 description-ja의 일치 | `set-locale.sh`로 전환 가능 |

## CI 세이프티넷

`.github/workflows/release.yml`이 태그 푸시 시에 자동으로 실행된다:

1. `v*` 태그의 푸시를 감지
2. 동일 이름의 GitHub Release가 이미 존재하는지 확인
3. 존재하지 않으면 CHANGELOG에서 자동 생성（세이프티넷）
4. 존재하면 아무것도 하지 않음

수동으로 GitHub Release를 먼저 생성하고서 푸시하는 것이 추천 플로우입니다.
세이프티넷은「Release 생성 놓침」만을 구합니다.

## PM 핸드오프

릴리스후에 PM에게의 완료 보고:

```markdown
## 릴리스 완료 보고

**버전**: v{{NEW_VERSION}}
**릴리스일**: {{DATE}}

### 실시 내용
{{CHANGELOG의 내용}}

### GitHub Release
{{URL}}

### 다음 액션
- PM에 의한 릴리스 노트 확인
- 본환경에 대한 배포（해당하는 경우）
```

## 금지 사항

- 태그 삭제·되돌리기（공개된 버전은 불변）
- 같은 날 2회 이상의 minor 밤프
- patch 수준의 변경에서의 minor 밤프
- `--force` / `--force-with-lease`에 의한 force push
- 릴리스 커밋에 VERSION / plugin.json / CHANGELOG 이외의 구현 변경을 혼합

## 관련 스킬

- `harness-review` -- 릴리스전에 코드 리뷰를 실시
- `harness-work` -- 릴리스 후의 다음 작업을 구현
- `harness-plan` -- 다음 버전의 계획을 작성
- `x-announce` -- X (Twitter)에의 릴리스 고지 스레드 생성
- `harness-setup` -- mirror 동기화나 플러그인 설정의 세팅

## 관련 규칙

- `.claude/rules/versioning.md` -- SemVer 판단 기준과 배치 릴리스 추천
- `.claude/rules/github-release.md` -- GitHub Release Notes 포맷（영어）
- `.claude/rules/cc-update-policy.md` -- CC 업뎃追従시의 Feature Table 品質 기준
