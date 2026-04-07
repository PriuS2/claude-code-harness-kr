---
name: memory
description: "SSOT와 메모리를 관리하고, 도구 간 메모리 검색을 제공. decisions.md와 patterns.md의 수호자입니다. 사용자가 메모리, SSOT, decisions.md, patterns.md, 머징, 마이그레이션, SSOT 승격, 메모리 동기화, 학습 저장, 메모리 검색, harness-mem, 과거 결정, 또는 이를 기록해달라고 말할 때 사용합니다. 구현 작업, 리뷰, 일시적 노트, 또는 세션 내 로깅에는 사용하지 않습니다."
description-en: "Manage SSOT, memory, and cross-tool memory search. Guardian of decisions.md and patterns.md. Use when user mentions memory, SSOT, decisions.md, patterns.md, merging, migration, SSOT promotion, sync memory, save learnings, memory search, harness-mem, past decisions, or record this. Do NOT load for: implementation work, reviews, ad-hoc notes, or in-session logging."
description-ja: "SSOTと記憶を管理し、ツール横断の記憶検索を提供。decisions.mdとpatterns.mdの守護者です。Use when user mentions memory, SSOT, decisions.md, patterns.md, merging, migration, SSOT promotion, sync memory, save learnings, memory search, harness-mem, past decisions, or record this. Do NOT load for: implementation work, reviews, ad-hoc notes, or in-session logging."
allowed-tools: ["Read", "Write", "Edit", "Bash", "mcp__harness__harness_mem_*"]
argument-hint: "[ssot|sync|migrate|search|record]"
context: fork
---

# Memory Skills

메모리와SSOT 관리를 담당하는 스킬 그룹입니다.

## 기능 상세

| 기능 | 상세 |
|------|------|
| **SSOT초기화** | See [references/ssot-initialization.md](${CLAUDE_SKILL_DIR}/references/ssot-initialization.md) |
| **Plans.md머지** | See [references/plans-merging.md](${CLAUDE_SKILL_DIR}/references/plans-merging.md) |
| **마이그레이션 처리** | See [references/workflow-migration.md](${CLAUDE_SKILL_DIR}/references/workflow-migration.md) |
| **프로젝트 스펙 동기화** | See [references/sync-project-specs.md](${CLAUDE_SKILL_DIR}/references/sync-project-specs.md) |
| **메모리→SSOT 승격** | See [references/sync-ssot-from-memory.md](${CLAUDE_SKILL_DIR}/references/sync-ssot-from-memory.md) |

## Unified Harness Memory（공통 DB）

Claude Code / Codex / OpenCode 공통의 기록·검색은 `harness_mem_*` MCP를 우선합니다.

- 검색: `harness_mem_search`, `harness_mem_timeline`, `harness_mem_get_observations`
- 주입: `harness_mem_resume_pack`
- 기록: `harness_mem_record_checkpoint`, `harness_mem_finalize_session`, `harness_mem_record_event`

## Claude Code 자동 메모리와의 관계（D22）

Harness의 SSOT 메모리（Layer 2）는 Claude Code의 자동 메모리（Layer 1）와 共存します.
자동 메모리는 범용적인 학습을暗黙的に記録하고, SSOT는 프로젝트 고유의 의사결정을明示的に管理합니다.
Layer 1의 지식이 프로젝트 전체에 중요한 경우, `/memory ssot`로 Layer 2에 승격하세요.

详细: [D22: 3층 메모리 아키텍처](../../.claude/memory/decisions.md#d22-3層メモリアーキテクチャ)

## 실행 절차

1. 사용자의 요청을 분류
2. 위의「기능 상세」에서 적절한 참조 파일을 읽음
3. 그 내용에 따라 실행

## SSOT 승격

메모리 시스템（Claude-mem / Serena）에서 중요한 학습을SSOT에永続화합니다.

- "**Save what we learned**" → [references/sync-ssot-from-memory.md](${CLAUDE_SKILL_DIR}/references/sync-ssot-from-memory.md)
- "**Promote decisions to SSOT**" → [references/sync-ssot-from-memory.md](${CLAUDE_SKILL_DIR}/references/sync-ssot-from-memory.md)
