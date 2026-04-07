---
name: generate-video
description: "Auto-generate product demo videos. A picture worth thousand words, embodied. Use when user mentions '/generate-video', video generation, product demos, or visual documentation. Do NOT load for: embedding video players, live demos, video playback features. Requires Remotion setup."
description-en: "Auto-generate product demo videos. A picture worth thousand words, embodied. Use when user mentions '/generate-video', video generation, product demos, or visual documentation. Do NOT load for: embedding video players, live demos, video playback features. Requires Remotion setup."
description-ja: "プロダクトデモ動画を自動生成。百聞は一見にしかず、を体現。'/generate-video'、動画生成、プロダクトデモ、ビジュアルドキュメントの作成時に起動。埋め込み動画プレイヤー、ライブデモ、動画再生機能では起動しない。Remotion セットアップが必要。"
allowed-tools: ["Read", "Write", "Edit", "Grep", "Glob", "Bash", "Task", "AskUserQuestion", "WebFetch"]
disable-model-invocation: true
argument-hint: "[demo|arch|release]"
context: fork
---

# Generate Video Skill

프로젝트 설명 영상을 자동 생성하는 스킬 그룹입니다.

---

## 개요

`/generate-video` 명령어의 내부에서 사용되는 스킬입니다.
코드베이스 분석 → 시나리오 제안 → 병렬 생성의 흐름을 실행합니다.

## 기능 상세

| 기능 | 상세 |
|------|------|
| **베스트 프랙티스** | See [references/best-practices.md](${CLAUDE_SKILL_DIR}/references/best-practices.md) |
| **코드베이스 분석** | See [references/analyzer.md](${CLAUDE_SKILL_DIR}/references/analyzer.md) |
| **시나리오 플래닝** | See [references/planner.md](${CLAUDE_SKILL_DIR}/references/planner.md) |
| **병렬 씬 생성** | See [references/generator.md](${CLAUDE_SKILL_DIR}/references/generator.md) |
| **시각 효과 라이브러리** | See [references/visual-effects.md](${CLAUDE_SKILL_DIR}/references/visual-effects.md) |
| **AI 이미지 생성** | See [references/image-generator.md](${CLAUDE_SKILL_DIR}/references/image-generator.md) |
| **이미지 품질 판정** | See [references/image-quality-check.md](${CLAUDE_SKILL_DIR}/references/image-quality-check.md) |

## 사전 조건

- Remotion이セットアップ済み（`/remotion-setup`）
- Node.js 18+
- （선택）`GOOGLE_AI_API_KEY` - AI 이미지 생성용

## `/generate-video` 흐름

```
/generate-video
    │
    ├─[Step 1] 분석（analyzer.md）
    │   ├─ 프레임워크 감지
    │   ├─ 주요 기능 감지
    │   ├─ UI 컴포넌트 감지
    │   └─ 프로젝트 자산 분석（Plans.md, CHANGELOG等）
    │
    ├─[Step 2] 시나리오 제안（planner.md）
    │   ├─ 영상 타입 자동 판정
    │   ├─ 씬 구성 제안
    │   └─ 사용자 확인
    │
    ├─[Step 2.5]素材生成（image-generator.md）← NEW
    │   ├─素材必要判定（인트로、CTA等）
    │   ├─ Nano Banana Pro で2枚生成
    │   ├─ Claude が品質判定（image-quality-check.md）
    │   └─ OK → 採用 / NG → 再生成（最大3回）
    │
    └─[Step 3] 병렬 생성（generator.md）
        ├─ 씬 병렬 생성（Task tool）
        ├─ 통합 + 트랜지션
        └─ 최종 렌더링
```

## 실행 절차

1. 사용자가 `/generate-video` 를 실행
2. Remotionセットアップ確認
3. `analyzer.md` 로 코드베이스 분석
4. `planner.md` 로 시나리오 제안 + 사용자 확인
5. `generator.md` 로 병렬 생성
6. 완료 보고

## 영상 타입（펀널별）

| 타입 | 펀널 | 길이目安 | 자동 판정 조건 | 구성 핵심 |
|--------|----------|----------|--------------|----------|
| **LP/광고 티저** | 인지~관심 | 30-90초 | 신규 프로젝트 | 고통→결과→CTA |
| **인트로 데모** | 관심→검토 | 2-3분 | UI 변경 감지 | 1 유즈케이스 완료 |
| **릴리스 노트** | 검토→확신 | 1-3분 | CHANGELOG 업데이트 | Before/After 중시 |
| **아키텍처 해설** | 확신→결정 | 5-30분 | 대규모 구조 변경 | 실제 운영+증거 |
| **온보딩** | 지속·활용 | 30초-수분 | 최초セットアップ | Aha 경험으로的最短パス |

> 상세: [references/best-practices.md](${CLAUDE_SKILL_DIR}/references/best-practices.md)

## 씬 템플릿

### 90초 티저（LP/광고용）

| 시간 | 씬 | 내용 |
|------|--------|------|
| 0-5초 | Hook | 고통 or 원하는 결과 |
| 5-15초 | Problem+Promise | 대상 사용자와 약속 |
| 15-55초 | Workflow | 상징 워크플로우 |
| 55-70초 | Differentiator | 차별화의 근거 |
| 70-90초 | CTA | 다음 수단 |

### 3분 Intro 데모（검토용）

| 시간 | 씬 | 내용 |
|------|--------|------|
| 0-10초 | Hook | 결론+고통 |
| 10-30초 | UseCase | 유즈케이스 선언 |
| 30-140초 | Demo | 실제 화면에서 완료 |
| 140-170초 | Objection | 흔한 불안감 1개 해소 |
| 170-180초 | CTA | 행동 유도 |

### 공통 씬

| 씬 | 권장 시간 | 내용 |
|--------|----------|------|
| 인트로 | 3-5초 | 로고 + 태그라인 |
| 기능 데모 | 10-30초 | Playwright 캡처 |
| 아키텍처 다이어그램 | 10-20초 | Mermaid → 애니메이션 |
| CTA | 3-5초 | URL + 연락처 |

> 상세 템플릿: [${CLAUDE_SKILL_DIR}/references/best-practices.md](${CLAUDE_SKILL_DIR}/references/best-practices.md#템플릿)

## 음성 동기화 규칙（중요）

내레이션이 포함된 영상에서는以下を厳守:

| 규칙 | 값 |
|--------|-----|
| 음성 시작 | 씬 시작 + 30f（1초 대기） |
| 씬 길이 | 30f + 음성 길이 + 20f 여백 |
| 트랜지션 | 15f（이웃 씬과 오버랩） |
| 씬 시작 계산 | 이전 씬 시작 + 이전 씬 길이 - 15f |

**사전 확인**: `ffprobe` 로 음성 길이를 확인してから 씬 설계

> 상세: [${CLAUDE_SKILL_DIR}/references/generator.md](${CLAUDE_SKILL_DIR}/references/generator.md#음성-동기화-규칙-중요)

## BGM 지원

| 항목 | 권장값 |
|------|--------|
| 내레이션 있음 | bgmVolume: 0.20 - 0.30 |
| 내레이션 없음 | bgmVolume: 0.50 - 0.80 |
| 파일 배치 | `public/BGM/` |

> 상세: [${CLAUDE_SKILL_DIR}/references/generator.md](${CLAUDE_SKILL_DIR}/references/generator.md#bgm-지원)

## 자막 지원

| 규칙 | 값 |
|--------|-----|
| 자막 시작 | 음성 시작과 동일 |
| 자막 duration | 음성 길이 + 10f |
| 폰트 | Base64 임베딩 권장 |

> 상세: [${CLAUDE_SKILL_DIR}/references/generator.md](${CLAUDE_SKILL_DIR}/references/generator.md#자막-지원)

## 시각 효과 라이브러리

임팩트 있는 영상용 이펙트 모음:

| 이펙트 | 용도 |
|-----------|------|
| GlitchText | Hook, 타이틀 |
| Particles | 배경, CTA 수렴 |
| ScanLine |解析 중 연출 |
| ProgressBar | 병렬 처리 표시 |
| 3D Parallax | 카드 표시 |

> 상세: [references/visual-effects.md](${CLAUDE_SKILL_DIR}/references/visual-effects.md)

## Notes

- Remotion未セットアップの場合は `/remotion-setup` を案内
- 병렬 생성수는 씬 수에 따라 자동 조정（max 5）
- 생성된 영상은 `out/` 디렉토리에 출력
- AI 생성 이미지는 `out/assets/generated/` 에 저장
- `GOOGLE_AI_API_KEY` 未設定時は画像生成をスキップ（既存素材 or プレースホルダー使用）
