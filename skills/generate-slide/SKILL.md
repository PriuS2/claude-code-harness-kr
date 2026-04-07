---
name: generate-slide
description: "Nano Banana Proでプロジェクト紹介スライドを自動生成。スライド、1枚紹介、ビジュアル紹介で起動。動画生成やデッキ作成では起動しない。"
description-ja: "Nano Banana Proでプロジェクト紹介スライドを自動生成。スライド、1枚紹介、ビジュアル紹介で起動。動画生成やデッキ作成では起動しない。"
description-en: "Generate project intro slides with Nano Banana Pro. Use when user mentions slide, project slide, 1-page summary, or visual introduction."
allowed-tools: ["Read", "Write", "Edit", "Bash", "Glob", "Grep", "AskUserQuestion"]
argument-hint: "[project-path|description]"
---

# Generate Slide Skill

프로젝트의 내용을 소개·설명하는 1매 슬라이드 이미지를 Nano Banana Pro（Gemini 3 Pro Image Preview）API로 자동 생성합니다.

---

## 개요

3 패턴 x 각 2매 후보 = 총 6매 생성 → 패턴별로 품질 체크 → NG시면 리트리 → 각 패턴의 최상 1매, 총 3매를 출력.

## 사전 조건

- `GOOGLE_AI_API_KEY` 환경 변수가 설정됨
- Google AI Studio 에서 Nano Banana Pro（Gemini 3 Pro Image Preview）가有効化됨

## 기능 상세

| 기능 | 상세 |
|------|------|
| **슬라이드 이미지 생성** | See [references/slide-generator.md](${CLAUDE_SKILL_DIR}/references/slide-generator.md) |
| **품질 판정** | See [references/slide-quality-check.md](${CLAUDE_SKILL_DIR}/references/slide-quality-check.md) |

---

## 실행 플로우

```
/generate-slide
    |
    +--[Step 1] 정보 수집
    |   +-- 사용자 지정 텍스트 or 코드베이스 자동 분석（README, package.json 등）
    |   +-- 프로젝트명·개요·주요 기능·기술 스택 추출
    |
    +--[Step 2] 사양 확인（AskUserQuestion）
    |   +-- 사이즈·애스팩트 비율（기본값: 16:9 / 2K）
    |   +-- 톤（텍, 캐주얼, 코포레이트 등）
    |   +-- 강조したい 포인트（모호한 경우에만 질문）
    |
    +--[Step 3] 3 패턴 x 2매 생성（Nano Banana Pro API x 6회）
    |   +-- Pattern A: Minimalist（2매）
    |   +-- Pattern B: Infographic（2매）
    |   +-- Pattern C: Hero Visual（2매）
    |
    +--[Step 4] 패턴별로 품질 체크
    |   +-- 각 패턴의 2매를 Claude 가 Read로 읽기
    |   +-- 5단계 스코어링 → 높은 쪽을 채용 후보
    |   +-- 양쪽 다 스코어 2 이하 → 프롬프트 개선하여 리트리（최대 3회）
    |   +-- 리트리 상한 도달 → 사용자에게 보고,続行 or スキップを選択
    |
    +--[Step 5] 최상 3매를 출력
        +-- 각 패턴의 베스트 1매를 selected/에 복사
        +-- 결과 목록（パス + スコア + 評価コメント）を사용자에게 제시
```

---

## 디자인 패턴

| 패턴 | 콘셉트 | 특징 |
|---------|-----------|------|
| **Minimalist** | 여백과タイポグラフィ主体 | clean, whitespace, typography-driven, elegant |
| **Infographic** | 데이터/플로우 시각화 | data visualization, metrics, flow diagram, structured |
| **Hero Visual** | 큰 비주얼 + 캐치프레이즈 | bold visual, impactful, hero image, catchy headline |

---

## 출력처

```
out/slides/
+-- minimalist_1.png       # Pattern A 후보1
+-- minimalist_2.png       # Pattern A 후보2
+-- infographic_1.png      # Pattern B 후보1
+-- infographic_2.png      # Pattern B 후보2
+-- hero_1.png             # Pattern C 후보1
+-- hero_2.png             # Pattern C 후보2
+-- selected/
|   +-- minimalist.png     # Pattern A 최상
|   +-- infographic.png    # Pattern B 최상
|   +-- hero.png           # Pattern C 최상
+-- quality-report.md      # 품질 체크 결과 보고서
```

---

## 실행 절차

### Step 1: 정보 수집

프로젝트 정보를以下の優先順位で収集:

1. **사용자 지정 텍스트**: 引数でプロジェクト説明が渡された場合はそれを使用
2. **코드베이스 자동 분석**: 引数がない場合、以下を自動分析
   - `README.md` — 프로젝트 개요
   - `package.json` / `Cargo.toml` / `pyproject.toml` — 프로젝트명·설명·의존 관계
   - `CLAUDE.md` — 프로젝트 구성·목적
   - `Plans.md` — 진행 중인 태스크（존재하는 경우）

추출하는 정보:

| 항목 | 예 |
|------|-----|
| 프로젝트명 | Claude Code Harness |
| 개요（1-2문）| Claude Code를 Plan-Work-Review로 자율 운용하는 플러그인 |
| 주요 기능（3-5개）| 스킬 관리, 품질 체크, 병렬 실행 |
| 기술 스택 | TypeScript, Node.js, Claude Code Plugin |
| 컬러（있는 경우）| 브랜드 컬러 or 추정 |

### Step 2: 사양 확인

AskUserQuestion で以下を確認（デフォルト値があるため、曖昧な場合のみ質問）:

```
질문 1: 슬라이드의 사이즈·애스팩트 비율은?
  - 16:9 / 2K（권장）
  - 4:3 / 2K
  - 1:1 / 2K
  - 커스텀

질문 2: 톤은?
  - 텍（다크 테마, 코드감）
  - 캐주얼（밝은, 프렌들리）
  - 코포레이트（포멀, 신뢰감）
  - 크리에이티브（대담, 아트寄り）
```

### Step 3: 이미지 생성

`slide-generator.md` 의 절차에 따라, 3 패턴 x 2매 = 6매를 생성.

각 패턴의 생성은 독립적이기 때문에, 가능한 병렬로 curl을 실행:

```bash
# 並列実行例（3 패턴 x 2매）
for pattern in minimalist infographic hero; do
  for i in 1 2; do
    # slide-generator.md 의 curl 패턴을 실행
    # → out/slides/${pattern}_${i}.png に保存
  done
done
```

### Step 4: 품질 체크

`slide-quality-check.md` 의 기준에 따라, 각 패턴의 2매를 평가:

1. 각 이미지를 Read로 읽기
2. 5단계 스코어링（정보 전달력, 레이아웃, 텍스트 가독성, 프로페셔널감, 브랜드 통합성）
3. 패턴 내에서 스코어가 높은 쪽을 채용 후보
4. 양쪽 다 스코어 2 이하 → 프롬프트 개선하여 재생성（최대 3회）

### Step 5: 결과 출력

```bash
# 최상 이미지를 selected/에 복사
mkdir -p out/slides/selected
cp out/slides/minimalist_best.png out/slides/selected/minimalist.png
cp out/slides/infographic_best.png out/slides/selected/infographic.png
cp out/slides/hero_best.png out/slides/selected/hero.png
```

품질 보고서（`out/slides/quality-report.md`）를 생성:

```markdown
# Slide Quality Report

## 생성 정보
- 프로젝트: {project_name}
- 생성 일시: {datetime}
- 애스팩트 비율: {aspect_ratio}
- 톤: {tone}

## 결과 요약

| 패턴 | 후보1 | 후보2 | 채용 | 스코어 |
|---------|-------|-------|------|--------|
| Minimalist | 3/5 | 4/5 | 후보2 | 4/5 |
| Infographic | 4/5 | 3/5 | 후보1 | 4/5 |
| Hero Visual | 5/5 | 4/5 | 후보1 | 5/5 |

## 상세 평가
...
```

---

## 에러 핸들링

### GOOGLE_AI_API_KEY 미설정

```
GOOGLE_AI_API_KEY가 설정되어 있지 않습니다.

설정 방법:
1. Google AI Studio에서 API 키를取得: https://ai.google.dev/aistudio
2. export GOOGLE_AI_API_KEY="your-api-key"
```

### 전 패턴에서 리트리 상한 도달

AskUserQuestion で選択肢を提示:

```
패턴 {pattern}의 이미지가 3회의 리트리에서도 기준을 충족하지 못했습니다.

선택지:
1. 가장 고스코어 이미지를 채용하여続行
2. 이 패턴을 スキップ
3. 프롬프트를 수동으로 지정하여 재생성
```

---

## 관련 스킬

- `generate-video` — 프로덕트 데모 영상 생성（이미지 생성 엔진 공유）
- `notebookLM` — 문서·슬라이드 생성（별 접근）
