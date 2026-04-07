# Image Generator - Nano Banana Pro 画像自動生成

Nano Banana Pro（Google DeepMind）を使用して、動画シーン用の高品質画像を自動生成します。

---

## 개요

`/generate-video` 의 씬 生成 페이즈에서、素材画像が必要と判定された場合に自動実行されます。
2枚生成 → Claude が品質判定 → NG なら再生成、という品質保証ループを実装しています。

## 사전 조건

- `GOOGLE_AI_API_KEY` 환경 변수가 설정됨
- Google AI Studio で Nano Banana Pro（Gemini 3 Pro Image Preview）가有効化됨

---

## API 仕様

> **공식 문서**: [Nano Banana image generation | Gemini API](https://ai.google.dev/gemini-api/docs/image-generation)

### 엔드포인트

```
POST https://generativelanguage.googleapis.com/v1beta/models/gemini-3-pro-image-preview:generateContent
```

### 모델 선택

| 모델 | 용도 | 최대 해상도 |
|--------|------|-----------|
| `gemini-3-pro-image-preview` | 프로品质（권장） | 4K |
| `gemini-2.5-flash-image` |高速·저렴| 1024px |

### 인증

```bash
# x-goog-api-key ヘッダー（Gemini API 標準方式）
x-goog-api-key: ${GOOGLE_AI_API_KEY}
```

> **주의**: Gemini API は `x-goog-api-key` ヘッダーを使用します。Query parameter 方式 (`?key=...`) 도利用可能하지만、ヘッダー方式을 권장합니다。

### 요청 형식

```json
{
  "contents": [{
    "parts": [
      {"text": "A modern SaaS dashboard interface with clean design, showing analytics charts and user metrics, professional UI mockup, light theme"}
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

> **注**: `responseModalities` で `["TEXT", "IMAGE"]` または `["IMAGE"]` を指定できます。本フローでは品質判定用にテキスト説明も取得するため、両方を指定しています。

### 응답 형식

```json
{
  "candidates": [{
    "content": {
      "parts": [
        {"text": "Here is the generated image of a modern SaaS dashboard..."},
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

> **注**: REST API は snake_case を使用します（`inline_data`, `mime_type`）。SDK は camelCase（`inlineData`, `mimeType`）を使用します。

---

## 해상도 옵션

| 설정 | 해상도 | 용도 | 비용目安 |
|------|--------|------|-----------|
| `1K` | 1024×1024 | 프리뷰, 테스트 | ~$0.02/枚 |
| `2K` | 2048×2048 | 표준 품질 | ~$0.06/枚 |
| `4K` | 4096×4096 | 고품질, 프로페셔널 | ~$0.12/枚 |

###アスペクト比

| 비율 | 용도 |
|------|------|
| `16:9` | 영상 씬（권장） |
| `1:1` | 아이콘, 로고 |
| `9:16` | 세로형 영상 |
| `4:3` | 프레젠테이션 자료 |

---

## プロンプト設計 가이드라인

### 基本 구조

```
[主題] + [スタイル] + [品質指定] + [制約]
```

### 씬 타입별 プロンプト テンプレート

#### 인트로/타이틀 씬

```
Professional product logo and title card for "{product_name}",
modern minimalist design, clean typography,
{brand_color} accent color, dark background,
cinematic quality, 4K render
```

#### UI デモ 씬（보조 이미지）

```
Modern web application interface showing {feature_description},
clean UI design, light theme, subtle shadows,
professional SaaS aesthetic, mockup style,
no text labels, focus on visual hierarchy
```

#### CTA 씬

```
Call-to-action banner for {product_name},
action-oriented design, prominent button,
{brand_color} gradient, professional marketing style,
clear visual hierarchy, engaging composition
```

#### 아키텍처/개념도

```
Technical architecture diagram showing {concept},
isometric illustration style, modern tech aesthetic,
clear visual flow, connected components,
professional documentation quality, clean lines
```

### プロンプト品質向上のコツ

| 추가 요소 | 효과 |
|---------|------|
| `professional quality` | 전체的品质向上 |
| `clean design` | 불필요한 요소 감소 |
| `modern aesthetic` | 현대적 디자인 |
| `cinematic lighting` | 드라마틱한 조명 |
| `4K render` | 고해상도 |
| `no text` | 텍스트 없음（後で追加する場合）|

### 피해야 할 プロンプト

| NG 패턴 | 이유 |
|------------|------|
| 모호한 지시 | 「いい感じの画像」→ 結果 불안정 |
| 과도하게 복잡 | 요소가 많으면品質低下 |
| 텍스트 지정 | AI 生成テキストは品質不安定 |
| 저작권물 | ブランドロゴ等は生成不可 |

---

##実行 플로우

```
씬 生成 페이즈
    │
    ├── [Step 1] 素材必要判定
    │   └─ 씬 타입,既存素材の有無を確認
    │       ├── 素材あり → スキップ
    │       └── 素材なし → Step 2 へ
    │
    ├── [Step 2] プロンプト生成
    │   ├─ 씬 정보からプロンプト構築
    │   ├─  브랜드情報（색, 스타일）を反映
    │   └─ テンプレートを適用
    │
    ├── [Step 3] 画像生成（2枚並列）
    │   └─ Nano Banana Pro API 呼び出し（2回並列実行）
    │       generateContent × 2（同時リクエストで遅延削減）
    │
    ├── [Step 4] 品質判定
    │   └─ → image-quality-check.md 参照
    │
    ├── [Step 5] 結果処理
    │   ├── 成功 → 画像保存、씬에組み込み
    │   └── 失敗 → Step 6 へ
    │
    └── [Step 6] 再生成ループ（最大3回）
        ├─ プロンプト改善（Claude が提案）
        └─ Step 3 に戻る
```

---

## Bash 実行例

### curl 로의 API 호출

```bash
# 環境変数確認（キーが設定されているか確認）
test -n "$GOOGLE_AI_API_KEY" && echo "GOOGLE_AI_API_KEY is set" || echo "GOOGLE_AI_API_KEY is not set"

# 画像生成リクエスト
curl -X POST \
  "https://generativelanguage.googleapis.com/v1beta/models/gemini-3-pro-image-preview:generateContent" \
  -H "x-goog-api-key: ${GOOGLE_AI_API_KEY}" \
  -H "Content-Type: application/json" \
  -d '{
    "contents": [{
      "parts": [
        {"text": "Modern SaaS dashboard interface, clean design, light theme, professional UI"}
      ]
    }],
    "generationConfig": {
      "responseModalities": ["TEXT", "IMAGE"],
      "imageConfig": {
        "aspectRatio": "16:9",
        "imageSize": "2K"
      }
    }
  }' \
  -o response.json

# Base64 デコードして保存（parts配列から画像データを抽出）
cat response.json | jq -r '.candidates[0].content.parts[] | select(.inline_data) | .inline_data.data' | head -1 | base64 -d > out/assets/generated/image_1.png
```

> **주의**: 1회의リクエストで1枚の画像が生成されます。2枚必要な場合は2回リクエストを実行してください。

###画像 保存先

```
out/
└── assets/
    └── generated/
        ├── intro_1.png
        ├── intro_2.png
        ├── cta_1.png
        └── cta_2.png
```

---

## 再生成ループ制御

### 最大試行回数

```
max_attempts = 3
```

### 再生成時のプロンプト改善

각 시도마다 Claude がプロンプトを改善:

| 시도 | 개선 전략 |
|------|---------|
| 1회 |初期プロンプトで生成|
| 2회 |品質指摘を反映してプロンプト調整|
| 3회 |より具体的な指示を追加、スタイル変更|

###改善 プロンプト生成

```
前回の画像が以下の理由で不採用でした:
- {rejection_reason}

改善案:
1. {improvement_1}
2. {improvement_2}

新しいプロンプト:
{improved_prompt}
```

### 3回失敗時のフォールバック

```
⚠️ 画像生成が3回失敗しました

씬: {scene_name}
最後のエラー: {last_error}

선택지:
1. 「続行」→ プレースホルダー画像で進める
2. 「スキップ」→ このシーンを画像なしで生成
3. 「手動」→  使用자가 이미지를 제공
```

---

## エラーハンドリング

### API エラー

| 에러 코드 | 원인 | 대응 |
|-------------|------|------|
| `400` | 不正なプロンプト | プロンプト内容を確認 |
| `401` | 認証失敗 | API 키를 확인 |
| `429` | レート制限 | 60초 대기して再試行 |
| `500` | 서버 에러 | 30초 대기して再試行 |

### コンテンツポリシー違反

```
⚠️ コンテンツポリシー違反

プロンプトが Google のポリシーに違反しています。
以下を削除/変更してください:
- {violation_reason}

自動修正を試みますか？ (y/n)
```

### 環境変数未設定

```
⚠️ GOOGLE_AI_API_KEY が設定されていません

設定方法:
1. Google AI Studio でAPIキーを取得
   https://ai.google.dev/aistudio

2. 環境変数に設定
   export GOOGLE_AI_API_KEY="your-api-key"

3. または .env.local に追加
   GOOGLE_AI_API_KEY=your-api-key
```

---

## コスト見積もり

### 씬당 비용

```
基本: 2枚 × $0.12 = $0.24
最大（3回再生成）: 6枚 × $0.12 = $0.72
```

### 영상당 비용目安

| 영상 타입 | 씬 수 | 画像生成 씬 | 비용目安 |
|-----------|---------|---------------|-----------|
| 90초 티저 | 5 | 2-3 | $0.48-$0.72 |
| 3분 데모 | 8 | 3-4 | $0.72-$0.96 |
| 5분 아키텍처 | 12 | 4-6 | $0.96-$1.44 |

---

##関連 문서

- [image-quality-check.md](./image-quality-check.md) - 品質判定 로직
- [generator.md](./generator.md) - 並列 씬 生成 엔진
- [planner.md](./planner.md) - 시나리오 플래너
