# Slide Generator - Nano Banana Pro 슬라이드 이미지 생성

Nano Banana Pro（Google DeepMind）를 사용하여 프로젝트 소개 슬라이드 이미지를 자동 생성합니다.

---

## 개요

`/generate-slide` 의 Step 3에서 실행되는 이미지 생성 로직입니다.
3개의 디자인 패턴 각각으로 2매씩 생성하고,品質チェック後にベスト1枚を選出합니다。

## 사전 조건

- `GOOGLE_AI_API_KEY` 환경 변수가 설정됨
- Google AI Studio 에서 Nano Banana Pro（Gemini 3 Pro Image Preview）가有効化됨

---

## API 仕様

> **共通仕様**: `generate-video/references/image-generator.md` 와同一の Nano Banana Pro API 를 사용합니다。

### 엔드포인트

```
POST https://generativelanguage.googleapis.com/v1beta/models/gemini-3-pro-image-preview:generateContent
```

### 인증

```bash
x-goog-api-key: ${GOOGLE_AI_API_KEY}
```

### 요청 형식

```json
{
  "contents": [{
    "parts": [
      {"text": "<slide prompt here>"}
    ]
  }],
  "generationConfig": {
    "responseModalities": ["TEXT", "IMAGE"],
    "imageConfig": {
      "aspectRatio": "16:9",
      "imageSize": "2K"
    }
  }
}
```

### 응답 형식

```json
{
  "candidates": [{
    "content": {
      "parts": [
        {"text": "Description of the generated slide..."},
        {
          "inline_data": {
            "mime_type": "image/png",
            "data": "iVBORw0KGgoAAAANS..."
          }
        }
      ]
    }
  }]
}
```

---

## 기본 설정

| 설정 | 값 | 설명 |
|------|-----|------|
| 모델 | `gemini-3-pro-image-preview` | 프로品質（권장）|
| 애스팩트 비율 | `16:9` | 프레젠테이션 표준 |
| 해상도 | `2K` | 2048px, 표준 품질 |
| responseModalities | `["TEXT", "IMAGE"]` | 텍스트 설명 + 이미지 |

### 애스팩트 비율 옵션

| 비율 | 용도 |
|------|------|
| `16:9` | 프레젠테이션·스크린（권장）|
| `4:3` | 전통형 프레젠테이션 |
| `1:1` | SNS 게시, 아이콘용 |

---

## 3つの 디자인 패턴

### Pattern A: Minimalist

**콘셉트**: 여백과 타이포그래피 주도. 세련된 인상.

**프롬프트 템플릿**:

```
Create a minimalist project introduction slide for "{project_name}".

Project description: {project_description}
Key features: {features}

Design style:
- Clean whitespace-dominant layout
- Typography-driven hierarchy with bold project name
- Subtle accent color: {accent_color}
- {tone} aesthetic
- No cluttered elements, elegant simplicity
- Professional presentation quality, 2K resolution

Important: This is a single slide image, not a deck. Focus on clear visual hierarchy with the project name prominent and key value proposition visible.
```

**시각 이미지**:
```
+------------------------------------------+
|                                          |
|                                          |
|        PROJECT NAME                      |
|        _______________                   |
|                                          |
|        One-line description              |
|                                          |
|        * Feature 1                       |
|        * Feature 2                       |
|        * Feature 3                       |
|                                          |
+------------------------------------------+
```

### Pattern B: Infographic

**콘셉트**: 데이터나 플로우의 시각화. 정보량이 많지만 정리됨.

**프롬프트 템플릿**:

```
Create an infographic-style project introduction slide for "{project_name}".

Project description: {project_description}
Key features: {features}
Tech stack: {tech_stack}

Design style:
- Data visualization and structured layout
- Icons and visual elements for each feature
- Flow or architecture diagram elements
- Metrics and key numbers highlighted
- {tone} color palette with {accent_color} accents
- Professional infographic quality, 2K resolution

Important: This is a single slide image. Organize information visually with icons, sections, and clear data hierarchy. Make the project's value immediately understandable through visual structure.
```

**시각 이미지**:
```
+------------------------------------------+
|  PROJECT NAME          [icon] [icon]     |
|  ================                        |
|                                          |
|  [Feature 1]    [Feature 2]    [Feat 3]  |
|  +----------+   +----------+   +------+  |
|  | icon     |   | icon     |   | icon |  |
|  | detail   |   | detail   |   | det  |  |
|  +----------+   +----------+   +------+  |
|                                          |
|  Tech: [TS] [Node] [React]    v1.0      |
+------------------------------------------+
```

### Pattern C: Hero Visual

**콘셉트**: 큰 비주얼과 캐치프레이즈로 임팩트 중시.

**프롬프트 템플릿**:

```
Create a hero-style project introduction slide for "{project_name}".

Project description: {project_description}
Key value: {key_value_proposition}

Design style:
- Bold, impactful hero image as background
- Large catchy headline text
- Dramatic visual composition
- {tone} mood with cinematic lighting
- Strong visual metaphor representing the project's purpose
- Professional marketing quality, 2K resolution

Important: This is a single slide image. Prioritize visual impact and emotional resonance. The project name and core value should be immediately visible with a compelling visual backdrop.
```

**시각 이미지**:
```
+------------------------------------------+
|                                          |
|    ==============================        |
|    ||  PROJECT NAME            ||        |
|    ||                          ||        |
|    ||  "Catchy tagline here"   ||        |
|    ||                          ||        |
|    ==============================        |
|                                          |
|         [ Bold Visual BG ]               |
|                                          |
+------------------------------------------+
```

---

## 프롬프트 구성

### 기본 구조

```
[프로젝트 개요] + [디자인 스타일] + [품질 지정] + [제약]
```

### 톤별修飾語

| 톤 |修飾語 |
|--------|--------|
| 텍 | `dark theme, code-inspired, terminal aesthetic, neon accents` |
| 캐주얼 | `bright colors, friendly, playful, approachable` |
| 코포레이트 | `formal, trustworthy, blue tones, clean lines, business` |
| 크리에이티브 | `bold, artistic, gradient, unconventional layout` |

### 품질 향상 키워드

| 키워드 | 효과 |
|-----------|------|
| `professional presentation quality` | 프레젠테이션 품질 |
| `clean design` | 불필요 요소 감소 |
| `2K resolution` | 고해상도 |
| `clear visual hierarchy` | 시각적 계층 |
| `modern aesthetic` | 현대적 디자인 |

### 피해야 할 프롬프트

| NG 패턴 | 이유 |
|------------|------|
| 모호한 지시 | 「いい感じのスライド」→ 結果 불안정 |
| 과도하게 복잡 | 요소가 많으면品質低下 |
| 장문 텍스트 지정 | AI 生成テキストは品質不安定. 키워드 정도에留める |
| 저작권물 | 브랜드 로고 등은 생성 불가 |

---

## Bash 実行例

### 환경 변수 확인

```bash
test -n "$GOOGLE_AI_API_KEY" && echo "GOOGLE_AI_API_KEY is set" || { echo "GOOGLE_AI_API_KEY is not set"; exit 1; }
```

### 출력 디렉토리 생성

```bash
mkdir -p out/slides/selected
```

### curl 로의 이미지 생성

```bash
PROMPT='Create a minimalist project introduction slide for "My Project". Clean whitespace-dominant layout, typography-driven, professional presentation quality, 2K resolution.'

curl -s -X POST \
  "https://generativelanguage.googleapis.com/v1beta/models/gemini-3-pro-image-preview:generateContent" \
  -H "x-goog-api-key: ${GOOGLE_AI_API_KEY}" \
  -H "Content-Type: application/json" \
  -d "{
    \"contents\": [{
      \"parts\": [
        {\"text\": \"${PROMPT}\"}
      ]
    }],
    \"generationConfig\": {
      \"responseModalities\": [\"TEXT\", \"IMAGE\"],
      \"imageConfig\": {
        \"aspectRatio\": \"16:9\",
        \"imageSize\": \"2K\"
      }
    }
  }" \
  -o /tmp/slide_response.json

# Base64 디코드して PNG 保存
cat /tmp/slide_response.json | jq -r '.candidates[0].content.parts[] | select(.inline_data) | .inline_data.data' | head -1 | base64 -d > out/slides/minimalist_1.png
```

> **주의**: 1回のリクエストで1枚の画像が生成됩니다. 2枚必要な場合は2回リクエストを実行하세요.

### 병렬 생성（6매 일괄）

```bash
mkdir -p out/slides/selected

generate_slide() {
  local pattern=$1
  local index=$2
  local prompt=$3
  local aspect_ratio=${4:-"16:9"}
  local image_size=${5:-"2K"}

  curl -s -X POST \
    "https://generativelanguage.googleapis.com/v1beta/models/gemini-3-pro-image-preview:generateContent" \
    -H "x-goog-api-key: ${GOOGLE_AI_API_KEY}" \
    -H "Content-Type: application/json" \
    -d "{
      \"contents\": [{
        \"parts\": [
          {\"text\": \"${prompt}\"}
        ]
      }],
      \"generationConfig\": {
        \"responseModalities\": [\"TEXT\", \"IMAGE\"],
        \"imageConfig\": {
          \"aspectRatio\": \"${aspect_ratio}\",
          \"imageSize\": \"${image_size}\"
        }
      }
    }" \
    -o "/tmp/slide_${pattern}_${index}.json"

  # Base64 디코드
  cat "/tmp/slide_${pattern}_${index}.json" \
    | jq -r '.candidates[0].content.parts[] | select(.inline_data) | .inline_data.data' \
    | head -1 \
    | base64 -d > "out/slides/${pattern}_${index}.png"
}

# 병렬 실행（백그라운드 잡）
generate_slide "minimalist" "1" "$MINIMALIST_PROMPT" &
generate_slide "minimalist" "2" "$MINIMALIST_PROMPT" &
generate_slide "infographic" "1" "$INFOGRAPHIC_PROMPT" &
generate_slide "infographic" "2" "$INFOGRAPHIC_PROMPT" &
generate_slide "hero" "1" "$HERO_PROMPT" &
generate_slide "hero" "2" "$HERO_PROMPT" &
wait

echo "6매의 생성이 완료되었습니다"
```

---

## 재생성시의 프롬프트 개선 전략

### 시도ごとの 개선

| 시도 | 개선 전략 |
|------|---------|
| 1회 |初期プロンプトで生成|
| 2회 |品質指摘を反映してプロンプト調整（구체적修飾語を追加）|
| 3회 |스타일을大幅変更、より具体的な構図指示を追加|

### 문제 카테고리별 개선

| 문제 | 개선 프롬프트 추가 |
|------|-------------------|
| 텍스트가 읽히지 않음 | `no text elements, text-free design` 을 추가 |
| 레이아웃이 무너짐 | `balanced composition, grid-based layout` 을 추가 |
| 정보량이 부족 | 구체적 기능명·수치를 프롬프트에 명기 |
| 프로페셔널감이 낮음 | `corporate quality, polished, refined` 을 추가 |
| 색이 맞지 않음 | 구체적 HEX 컬러코드를 지정 |

---

## 에러 핸들링

### API 에러

| 에러 코드 | 원인 | 대응 |
|-------------|------|------|
| `400` | 부적절한 프롬프트 | 프롬프트 내용을 확인·수정 |
| `401` | 인증 실패 | API 키를 확인 |
| `429` | 레이트 리밋 | 60초 대기하여再試行 |
| `500` | 서버 에러 | 30초 대기하여再試行 |

### jq 파스 에러

응답에 이미지 데이터가 포함되지 않은 경우:

```bash
# 응답 확인
cat /tmp/slide_response.json | jq '.candidates[0].content.parts | length'

# 에러 메시지 확인
cat /tmp/slide_response.json | jq '.error'
```

---

## 비용 추정

### 1회 실행당

```
基本: 6枚 x ~$0.06 = ~$0.36（2K 해상도）
最大（全パターンリトライ3回）: 18枚 x ~$0.06 = ~$1.08
```

### 해상도별 비용

| 해상도 | 1매당 | 6매（기본） | 18매（최대） |
|--------|----------|-------------|-------------|
| `1K` | ~$0.02 | ~$0.12 | ~$0.36 |
| `2K` | ~$0.06 | ~$0.36 | ~$1.08 |
| `4K` | ~$0.12 | ~$0.72 | ~$2.16 |

---

## 관련 문서

- [slide-quality-check.md](./slide-quality-check.md) — 품질 판정 로직
- [generate-video/references/image-generator.md](../../generate-video/references/image-generator.md) — API 공통 사양（상세）
- [generate-video/references/image-quality-check.md](../../generate-video/references/image-quality-check.md) — 영상用品질 판정（참고）
