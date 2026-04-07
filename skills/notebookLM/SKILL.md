---
name: notebookLM
description: "NotebookLM用YAMLやスライドを生成。ドキュメント職人の腕の見せ所。Use when user mentions NotebookLM, YAML, slides, or presentations. Do NOT load for: implementation work, code fixes, reviews, or deployments."
description-en: "Generate NotebookLM YAML and slides. Document craftsman shows skill. Use when user mentions NotebookLM, YAML, slides, or presentations. Do NOT load for: implementation work, code fixes, reviews, or deployments."
description-ja: "NotebookLM用YAMLやスライドを生成。ドキュメント職人の腕の見せ所。Use when user mentions NotebookLM, YAML, slides, or presentations. Do NOT load for: implementation work, code fixes, reviews, or deployments."
allowed-tools: ["Read", "Write", "Edit"]
argument-hint: "[yaml|slides]"
---

# NotebookLM Skill

문서 생성을 담당하는 스킬 그룹입니다.

## 기능 상세

| 기능 | 상세 |
|------|------|
| **NotebookLM YAML** | See [references/notebooklm-yaml.md](${CLAUDE_SKILL_DIR}/references/notebooklm-yaml.md) |
| **슬라이드 YAML** | See [references/notebooklm-slides.md](${CLAUDE_SKILL_DIR}/references/notebooklm-slides.md) |

## 실행 절차

1. 사용자의 요청을 분류
2. 위의「기능 상세」에서 적절한 참조 파일을 읽음
3. 그 내용에 따라 생성

---

## PDF 페이지 범위 읽기（Claude Code 2.1.49+）

대형 PDF를 효율적으로 다루기 위한 기능입니다.

### 페이지 범위 지정으로 읽기

```javascript
// 페이지 범위 지정으로 읽기
Read({ file_path: "docs/spec.pdf", pages: "1-10" })

// 목차만 확인
Read({ file_path: "docs/manual.pdf", pages: "1-3" })

// 특정 섹션만
Read({ file_path: "docs/api-reference.pdf", pages: "25-45" })
```

### 유즈케이스별 권장 접근

| 케이스 | 권장 읽기 방법 | 이유 |
|--------|----------------|------|
| **100페이지 초과의 PDF** | 목차(1-3) → 관련 장만 | 토큰 소비를 최소화 |
| **사양서 리뷰** | 섹션 단위로 범위 지정 | 필요한 부분만 정독 |
| **API 문서** | 엔드포인트 목록(목차)에서 시작 | 전체 구조를 파악한 후 상세로 |
| **학술 논문** | 초록 + 결론 → 本文 | 요점을 먼저 파악 |
| **기술 매뉴얼** | 목차 + 트러블슈팅 장 | 실용적인 부분을 우선 |

### NotebookLM YAML 생성시의 활용 예

```markdown
대형 PDF（300페이지 기술 사양서）에서 YAML을 생성하는 경우：

1. **목차를 읽음**（1-5페이지）
   Read({ file_path: "spec.pdf", pages: "1-5" })
   → 챕터 구성을 파악

2. **각 챕터의冒頭を読む**（각 챕터의 처음 2페이지）
   Read({ file_path: "spec.pdf", pages: "10-11" })  // 제1장
   Read({ file_path: "spec.pdf", pages: "45-46" })  // 제2장
   → 각 챕터의 개요를 파악

3. **중요 섹션을 정독**
   Read({ file_path: "spec.pdf", pages: "78-95" })  // API 레퍼런스
   →詳細な 내용을抽出

이 방법으로, 300페이지 모두를 읽지 않고도 효율적으로 YAML을 생성할 수 있습니다.
```

### 베스트 프랙티스

| 원칙 | 설명 |
|------|------|
| **단계적 읽기** | 목차 → 개요 → 상세의 순서로 읽기 |
| **관련 페이지만** | 태스크에 필요한 페이지만 지정 |
| **토큰 절약** | 전체 페이지 읽기는 최후의 수단 |
| **구조 이해 우선** | 목차로 전체상을 파악한 후 상세로 |

###従来の方法との比較

| 방법 | 토큰 소비 | 처리 시간 | 정밀도 |
|------|------------|---------|------|
| **전체 페이지 읽기**（300페이지）| ~150,000 | 김 | 높음 |
| **페이지 범위 지정**（필요한 30페이지）| ~15,000 | 짧음 | 높음 |

→ **90%의 토큰 절약과 처리 시간 단축이 가능**
