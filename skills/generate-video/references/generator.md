# Video Generator -並列 씬 生成 엔진

시나리오에 따라 멀티 에이전트로 병렬에 씬을 생성합니다.

---

## 개요

`/generate-video` 의 Step 3에서 실행되는 생성 엔진입니다.
planner.md 의 시나리오를 받아, 각 씬을 병렬로 생성하고 최종적으로統合します。

## 입력

planner.md 로부터의 시나리오:
- 씬 리스트（id, name, duration, template, content）
- 영상 설정（resolution, fps）

##並列 生成 아키텍처

```
시나리오（N 씬）
    │
    ├─[素材生成フェーズ] ← NEW
    │   ├── 各シーンの素材必要判定
    │   ├── Nano Banana Pro で画像生成（2枚: 2回リクエスト）
    │   ├── Claude が品質判定
    │   └── OK → 採用 / NG → 再生成（最大3回）
    │
    ├─[並列数決定]
    │   └─ min(씬 수, 5) を並列数とする
    │
    ├─[並列 生成フェーズ]
    │   ├── Agent 1: 씬 1 생성
    │   ├── Agent 2: 씬 2 생성
    │   ├── Agent 3: 씬 3 생성
    │   └── ... (max 5 並列)
    │
    ├─[統合フェーズ]
    │   ├── 씬 결합
    │   ├── トランジション追加
    │   └── 음성 동기화（선택）
    │
    └─[レンダリングフェーズ]
        └── 최종 출력（mp4/webm/gif）
```

---

##素材 生成 フェーズ（Nano Banana Pro）

씬 生成前に、必要な素材画像を自動生成します。

###素材 必要判定

| 씬 타입 |素材必要| 이유 |
|-------------|---------|------|
| intro | ✅ 필요 | 로고, 타이틀 카드 |
| cta | ✅ 필요 | 액션 배너 |
| architecture | ✅ 필요 | 개념도, 다이어그램 |
| ui-demo | ❌ 불필요 | Playwright 캡처 사용 |
| changelog | ❌ 불필요 | 텍스트 기반 |

###判定 로직

```javascript
const needsGeneratedAsset = (scene) => {
  //既存素材がある場合はスキップ
  if (scene.existingAssets?.length > 0) return false;

  // Playwright 캡처 대상はスキップ
  if (scene.template === 'ui-demo') return false;

  // テキストベース 씬はスキップ
  if (scene.template === 'changelog') return false;

  // それ以外は生成対象
  return ['intro', 'cta', 'architecture', 'feature-highlight'].includes(scene.template);
};
```

###生成 플로우

```
各シーンに対して:
    │
    ├── needsGeneratedAsset(scene) = false
    │   └─ スキップ → 次のシーンへ
    │
    └── needsGeneratedAsset(scene) = true
        │
        ├── [Step 1] プロンプト生成
        │   └─ 씬 정보 + 브랜드 정보からプロンプト構築
        │
        ├── [Step 2] 画像生成（2枚: 2回リクエスト）
        │   └─ Nano Banana Pro API 呼び出し（generateContent × 2）
        │   └─ → image-generator.md 参照
        │
        ├── [Step 3] 品質判定
        │   └─ Claude が2枚を評価・選択
        │   └─ → image-quality-check.md 参照
        │
        └── [Step 4] 結果処理
            ├── 成功 → out/assets/generated/{scene_name}.png
            └── 失敗 → 再生成（最大3回）or フォールバック
```

### 生成画像の保存先

```
out/
└── assets/
    └── generated/
        ├── intro.png
        ├── cta.png
        ├── architecture.png
        └── feature-highlight.png
```

### 씬への組み込み

生成した画像は、씬 生成エージェントに渡されます:

```
Task:
  subagent_type: "video-scene-generator"
  prompt: |
    씬 정보:
    - 이름: intro
    - 템플릿: intro
    - 생성 이미지: out/assets/generated/intro.png  ← 추가

    生成画像を背景またはメイン要素として使用してください。
```

### 상세 문서

- [image-generator.md](./image-generator.md) - API 呼び出し、プロンプト設計
- [image-quality-check.md](./image-quality-check.md) - 品質判定 로직

---

##並列 数決定 로직

| 씬 수 |並列数| 이유 |
|---------|--------|-------|
| 1-2 | 1-2 | 오버헤드가 이익을上回る |
| 3-4 | 3 | 최적의 균형 |
| 5+ | 5 | これ以上はリソース競合 |

**実装**:
```javascript
const parallelCount = Math.min(scenes.length, 5);
```

---

## Task Tool に 의한並列JSON生成

###新しい 生成 플로우（JSON-schema駆動）

```
시나리오（scenario.json）
    ↓
┌─────────────────────────────────────────────┐
│     Task並列起動（各シーン → JSON出力）      │
├─────────────────────────────────────────────┤
│ Agent 1 → scenes/intro.json                 │
│ Agent 2 → scenes/auth-demo.json             │
│ Agent 3 → scenes/dashboard.json             │
│ Agent 4 → scenes/features.json              │
│ Agent 5 → scenes/cta.json                   │
└─────────────────────────────────────────────┘
    ↓
┌─────────────────────────────────────────────┐
│         scenes/*.json → マージ               │
├─────────────────────────────────────────────┤
│ - section_id + order でソート               │
│ - 競合検出（同一scene_id = Critical error） │
│ - 欠落検出（セクションにシーンなし）         │
└─────────────────────────────────────────────┘
    ↓
video-script.json（全シーン統合）
    ↓
Remotion rendering
```

### 씬 生成エージェント起動（JSON出力）

```
各シーンに対して Task tool を起動:

Task:
  subagent_type: "video-scene-generator"
  run_in_background: true
  prompt: |
    以下のシーンのJSONを scene.schema.json に従って生成してください。

    씬 정보:
    - scene_id: {scene.id}
    - section_id: {section.id}
    - order: {scene.order} （섹션 내 순서）
    - type: {scene.type}
    - duration_ms: {scene.duration_ms}
    - content: {scene.content}

    出力先: out/video-{date}-{id}/scenes/{scene_id}.json

    필수 항목:
    - scene_id, section_id, order, type, content
    - content.duration_ms（음성 길이 + 여백 고려）
    - direction（transition, emphasis, background, timing）
    - assets（사용하는 이미지·음성 파일）

    バリデーション:
    ```bash
    node scripts/validate-scene.js out/video-{date}-{id}/scenes/{scene_id}.json
    ```

    완료 보고:
    - 파일パス
    - バリデーション結果（PASS/FAIL）
    - 警告があれば報告
```

###進捗 モニタリング

```
🎬 並列JSON生成中... (3/5 完了)

├── [Agent 1] intro.json ✅ PASS
├── [Agent 2] auth-demo.json ✅ PASS
├── [Agent 3] dashboard.json ⏳ 生成中...
├── [Agent 4] features.json 🔜 待機中
└── [Agent 5] cta.json 🔜 待機中
```

### 결과 수집（JSON）

```
TaskOutput で各エージェントの結果を収集:

결과:
  - scene_id: "intro"
    file: "out/video-20260202-001/scenes/intro.json"
    validation: "PASS"
    status: "success"

  - scene_id: "auth-demo"
    file: "out/video-20260202-001/scenes/auth-demo.json"
    validation: "PASS"
    status: "success"
    warnings: ["duration_ms が音声長より短い可能性"]
```

### JSON出力 仕様

**出力ファイル**: `out/video-{date}-{id}/scenes/{scene_id}.json`

**スキーマ**: `schemas/scene.schema.json`

**必須 필드**:
```json
{
  "scene_id": "intro",
  "section_id": "opening",
  "order": 0,
  "type": "intro",
  "content": {
    "title": "MyApp",
    "subtitle": "タスク管理を簡単に",
    "duration_ms": 5000
  },
  "direction": {
    "transition": {
      "in": "fade",
      "out": "fade",
      "duration_ms": 500
    },
    "emphasis": {
      "level": "high"
    },
    "background": {
      "type": "gradient",
      "value": "linear-gradient(135deg, #667eea 0%, #764ba2 100%)"
    }
  },
  "assets": [
    {
      "type": "image",
      "source": "assets/generated/intro.png",
      "generated": true
    }
  ]
}
```

### マージ フェーズ

全エージェントの完了後、`scripts/merge-scenes.js` を実行:

```bash
node scripts/merge-scenes.js out/video-20260202-001/
```

**処理 내용**:
1. `scenes/*.json` を読み込み
2. `section_id` + `order` でソート
3. 競合検出（同一 `scene_id` → Critical error）
4. 欠落検出（セクションにシーンなし → Critical error）
5. `video-script.json` を生成

**出力**: `out/video-20260202-001/video-script.json`

**フォーマット**:
```json
{
  "scenes": [
    { "scene_id": "intro", "section_id": "opening", "order": 0, ... },
    { "scene_id": "hook", "section_id": "opening", "order": 1, ... },
    { "scene_id": "demo", "section_id": "main", "order": 0, ... }
  ],
  "metadata": {
    "total_duration_ms": 180000,
    "scene_count": 12,
    "generated_at": "2026-02-02T12:34:56Z"
  }
}
```

---

## 씬 生成 テンプレート

### intro テンプレート

```tsx
// remotion/src/scenes/intro.tsx
import { AbsoluteFill, useCurrentFrame, interpolate } from "remotion";
import { FadeIn } from "../components/FadeIn";

export const IntroScene: React.FC<{
  title: string;
  tagline: string;
}> = ({ title, tagline }) => {
  const frame = useCurrentFrame();
  const opacity = interpolate(frame, [0, 30], [0, 1]);

  return (
    <AbsoluteFill style={{ backgroundColor: "#000", opacity }}>
      <FadeIn durationInFrames={30}>
        <h1>{title}</h1>
        <p>{tagline}</p>
      </FadeIn>
    </AbsoluteFill>
  );
};

export const DURATION = 150; // 5초 @ 30fps
```

### ui-demo テンプレート（Playwright連携）

```tsx
// remotion/src/scenes/ui-demo.tsx
import { AbsoluteFill, Img, Sequence } from "remotion";

export const UIDemoScene: React.FC<{
  screenshots: string[];
  duration: number;
}> = ({ screenshots, duration }) => {
  const framePerScreenshot = Math.floor(duration / screenshots.length);

  return (
    <AbsoluteFill>
      {screenshots.map((src, i) => (
        <Sequence from={i * framePerScreenshot} durationInFrames={framePerScreenshot}>
          <Img src={src} style={{ width: "100%", height: "100%" }} />
        </Sequence>
      ))}
    </AbsoluteFill>
  );
};
```

### cta テンプレート

```tsx
// remotion/src/scenes/cta.tsx
import { AbsoluteFill, useCurrentFrame, interpolate } from "remotion";

export const CTAScene: React.FC<{
  url: string;
  text: string;
}> = ({ url, text }) => {
  const frame = useCurrentFrame();
  const scale = interpolate(frame, [0, 15], [0.8, 1], {
    extrapolateRight: "clamp",
  });

  return (
    <AbsoluteFill style={{ backgroundColor: "#1a1a1a" }}>
      <div style={{ transform: `scale(${scale})` }}>
        <h2>{text}</h2>
        <p>{url}</p>
      </div>
    </AbsoluteFill>
  );
};

export const DURATION = 150; // 5초 @ 30fps
```

---

## 음성 동기화 규칙（중요）

내레이션이 포함된 영상 생성 시에는以下の 규칙을厳守할 것。

### 1. 음성 파일 길이의 사전 확인

```bash
# 各音声ファイルの長さを確認
for f in public/audio/*.wav; do
  name=$(basename "$f" .wav)
  dur=$(ffprobe -v error -show_entries format=duration -of default=noprint_wrappers=1:nokey=1 "$f")
  frames=$(echo "$dur * 30" | bc | cut -d. -f1)
  echo "$name: ${dur}초 = ${frames}프레임"
done
```

### 2. 씬 길이의 계산식

```
씬 길이 = 1초 대기(30f) + 음성 길이 + 트랜지션 전 여백(20f 이상)
```

| 요소 | 프레임 수 | 설명 |
|------|-----------|------|
| 1초 대기 | 30f | 씬 시작 후, 시각적으로落ち着いた 후 음성 시작 |
| 음성 길이 | 가변 | ffprobe로 사전 확인 |
| 여백 | 20f 이상 | 트랜지션 시작 전에 음성 종료 |

### 3. 음성 시작 타이밍

```
음성 시작 = 씬 시작 프레임 + 30 프레임（1초 대기）
```

### 4. 씬 시작 프레임의 계산（TransitionSeries 사용 시）

```
씬 시작 프레임 = 이전 씬 시작 + 이전 씬 길이 - 트랜지션 길이
```

**예시（트랜지션 15 프레임 경우）**:
```
hook:       0
problem:    175 - 15 = 160
solution:   160 + 415 - 15 = 560
workPlan:   560 + 340 - 15 = 885
...
```

### 5. 구현 템플릿

```tsx
const SCENE_DURATIONS = {
  hook: 175,      // 30 + 121(음성) + 24(여백)
  problem: 415,   // 30 + 360(음성) + 25(여백)
  solution: 340,  // 30 + 286(음성) + 24(여백)
  // ...
};
const TRANSITION = 15;

// 씬 시작 프레임（누적 계산）
// hook:0, problem:160, solution:560, ...

const audioTimings = {
  hook: 30,       // 씬0 + 30
  problem: 190,   // 씬160 + 30
  solution: 590,  // 씬560 + 30
  // ...
};
```

### 6. 흔한 문제와対策

| 문제 | 원인 |对策|
|------|------|------|
| 음성이 겹침 | 이전 음성 종료 전에 다음 음성 시작 | 음성 길이 확인, 씬 길이 조정 |
| 슬라이드 변경과 음성이 어긋남 | TransitionSeries의 오버랩 미고려 | 씬 시작 = 이전 씬 시작 + 이전 씬 길이 - 트랜지션 길이 |
| 음성이 중간에 끊김 | 씬 길이 < 음성 길이 | 씬 길이를 음성 길이 + 여백으로 조정 |
| 무음 시간이 김 | 음성 시작이 너무 늦음 | 씬 시작 + 30f로 통일 |

---

##統合 フェーズ

### 씬 결합

```tsx
// remotion/src/FullVideo.tsx
import { Composition, Series } from "remotion";
import { IntroScene } from "./scenes/intro";
import { UIDemoScene } from "./scenes/ui-demo";
import { CTAScene } from "./scenes/cta";

export const FullVideo: React.FC = () => {
  return (
    <Series>
      <Series.Sequence durationInFrames={150}>
        <IntroScene title="MyApp" tagline="タスク管理を簡単に" />
      </Series.Sequence>
      <Series.Sequence durationInFrames={450}>
        <UIDemoScene screenshots={[...]} duration={450} />
      </Series.Sequence>
      <Series.Sequence durationInFrames={150}>
        <CTAScene url="https://myapp.com" text="今すぐ試す" />
      </Series.Sequence>
    </Series>
  );
};
```

### 트랜지션 추가

```tsx
// トランジション コンポーネント
import { TransitionSeries, linearTiming } from "@remotion/transitions";
import { fade } from "@remotion/transitions/fade";

<TransitionSeries>
  <TransitionSeries.Sequence durationInFrames={150}>
    <IntroScene {...} />
  </TransitionSeries.Sequence>
  <TransitionSeries.Transition
    presentation={fade()}
    timing={linearTiming({ durationInFrames: 15 })}
  />
  <TransitionSeries.Sequence durationInFrames={450}>
    <UIDemoScene {...} />
  </TransitionSeries.Sequence>
</TransitionSeries>
```

---

##レンダリング フェーズ

### 명령 실행

```bash
# MP4 렌더링
npx remotion render remotion/index.ts FullVideo out/video.mp4

# GIF 렌더링（짧은 영상용）
npx remotion render remotion/index.ts FullVideo out/video.gif

# WebM 렌더링（Web용）
npx remotion render remotion/index.ts FullVideo out/video.webm --codec=vp8
```

### 출력 옵션

| 포맷 | 권장 용도 | 옵션 |
|-------------|---------|-----------|
| MP4 | 범용, SNS | `--codec=h264` |
| WebM | Web 임베딩 | `--codec=vp8` |
| GIF | 짧은 루프 | 15초 이하 권장 |

---

## 완료 보고

```markdown
✅ **영상 生成完了**

📁 **출력 파일**:
- `out/video.mp4` (45초, 1080p, 12.3MB)

📊 **生成統計**:
| 항목 | 값 |
|------|-----|
| 씬 수 | 4 |
|並列 에이전트 수 | 3 |
| 생성 시간 | 45초 |
| 렌더링 시간 | 30초 |

🎬 **프리뷰**:
- Studio: `npm run remotion` → http://localhost:3000
- 파일: `open out/video.mp4`
```

---

## エラーハンドリング

### 씬 生成 실패

```
⚠️ 씬 生成エラー

씬「auth-demo」의 生成に失敗しました。
原因: Playwright 캡처 실패 -  앱이起動していない

対処:
1. 앱을起動してください: `npm run dev`
2. 再生成: 「auth-demo を再生成」
3. スキップ: 「このシーンをスキップ」
```

### 렌더링 실패

```
⚠️ 렌더링 エラー

原因: メモリ不足

対処:
1.並列数を減らす: `--concurrency 2`
2. 解像度を下げる: 720p で再試行
3. 씬을分割: 긴 씬을短く分割
```

---

## BGM 支持

### 구현 방법

컴포지션에 `bgmPath` 와 `bgmVolume` プロパティを追加:

```tsx
export const VideoComposition: React.FC<{
  enableAudio?: boolean;
  volume?: number;
  bgmPath?: string;      // BGM 파일パス（staticFile相対）
  bgmVolume?: number;    // BGM音量（0.0-1.0）
}> = ({ enableAudio = true, volume = 1, bgmPath, bgmVolume = 0.25 }) => {
  return (
    <AbsoluteFill>
      {/* 씬 내용 */}

      {/* BGM（ナレーションより控えめに） */}
      {enableAudio && bgmPath && (
        <Audio src={staticFile(bgmPath)} volume={bgmVolume} />
      )}
    </AbsoluteFill>
  );
};
```

### BGM 음량 가이드라인

| 내레이션 여부 | 권장 bgmVolume |
|-----------------|----------------|
| 있음 | 0.20 - 0.30 |
| 없음 | 0.50 - 0.80 |

### 저작권 프리 BGM 입수처

- [DOVA-SYNDROME](https://dova-s.jp/) - 일본어, 무료
- [甘茶の音楽工房](https://amachamusic.chagasi.com/) - 일본어, 무료
- [Pixabay Music](https://pixabay.com/music/) - 영어, 무료

---

## 자막 支持

### 구현 방법

```tsx
// フォント埋め込み（Base64推奨）
const FontStyle: React.FC = () => (
  <style>
    {`
      @font-face {
        font-family: 'CustomFont';
        src: url('${FONT_DATA_URL}') format('opentype');
        font-weight: normal;
        font-style: normal;
      }
    `}
  </style>
);

// 자막 コンポーネント
const Subtitle: React.FC<{ text: string }> = ({ text }) => {
  const frame = useCurrentFrame();
  const opacity = interpolate(frame, [0, 10], [0, 1], {
    extrapolateRight: "clamp",
  });

  return (
    <>
      <FontStyle />
      <div
        style={{
          position: "absolute",
          bottom: 80,
          left: 0,
          right: 0,
          display: "flex",
          justifyContent: "center",
          padding: "0 60px",
        }}
      >
        <div
          style={{
            fontFamily: "'CustomFont', sans-serif",
            fontSize: 32,
            color: "#FFFFFF",
            backgroundColor: "rgba(0, 0, 0, 0.8)",
            padding: "14px 28px",
            borderRadius: 8,
            textAlign: "center",
            maxWidth: 1000,
            lineHeight: 1.5,
            opacity,
          }}
        >
          {text}
        </div>
      </div>
    </>
  );
};
```

### 자막 타이밍 규칙

| 항목 | 값 |
|------|-----|
| 자막 시작 | 음성 시작과 동일 타이밍 |
| 자막 duration | 음성 길이 + 10f（여백）|

### 폰트 임베딩（Base64）

커스텀 폰트를 확실히 읽어들이려면 Base64 임베딩 사용:

```typescript
// src/utils/custom-font.ts
import fs from "fs";
import path from "path";

// 빌드時にBase64エンコード
const fontPath = path.join(__dirname, "../../public/font/MyFont.otf");
const fontBuffer = fs.readFileSync(fontPath);
export const FONT_DATA_URL = `data:font/otf;base64,${fontBuffer.toString("base64")}`;
```

### 자막 데이터 구조

```tsx
const SUBTITLES = [
  { id: "hook", text: "자막 텍스트", start: 30, duration: 120 },
  { id: "problem", text: "다음 자막", start: 175, duration: 178 },
  // ...
];

// 使用
{SUBTITLES.map((sub) => (
  <Sequence key={sub.id} from={sub.start} durationInFrames={sub.duration}>
    <Subtitle text={sub.text} />
  </Sequence>
))}
```

---

## Notes

- 병렬 生成は 독립된 씬に対してのみ 유효
- Playwright 캡처는 사전에 앱이起動している必要がある
- 큰 영상（3분 이상）은分割レンダ링を推奨
- BGM은 내레이션이 들리도록控えめに 설정
- 커스텀 폰트는Base64 임베딩으로 확실히 읽어들이기

---

## Phase 10: 将来的拡張（キャラクター対話 영상）

### 개요

현재 영상 生成는 **단일 내레이션**形式ですが、将来的には以下のような**キャラクター対話 영상**に拡張可能な設計にします：

| 현재 | Phase 10 확장 후 |
|------|----------------|
| 단일 내레이터 | 복수 캐릭터의 대화 |
| 정적 슬라이드 + 음성 | 캐릭터 표시 + 대화 연출 |
| TTS: 1 음성만 | TTS: 캐릭터별 음성 |

### 유즈케이스 예시

```
[導入영상 의 예]

Narrator:  「오늘은 新機能を紹介します」
User:      「これは何ができますの？」
AI Guide:  「簡単に説明しましょう」
```

```
[기술 해설 영상의 예]

Interviewer: 「このアーキテクチャの特徴は？」
Expert:      「スケーラビリティを重視しています」
Reviewer:    「具体的な数値を見てみましょう」
```

### 확장 포인트（설계のみ）

#### 1. Character 정의（`schemas/character.schema.json`）

**이미 구현됨**의 스키마로, 이하를 정의:

```json
{
  "character_id": "narrator",
  "name": "내레이터",
  "role": "narrator",
  "voice": {
    "provider": "google-cloud-tts",
    "voice_id": "ja-JP-Neural2-B",
    "language": "ja",
    "speed": 1.1,
    "style": "professional"
  },
  "appearance": {
    "type": "avatar",
    "position": "left"
  }
}
```

**확장 항목**:
- `voice`: TTS 설정（프로바이더, 음성 ID, 속도, 스타일）
- `appearance`: 비주얼 설정（아바타, 아이콘, 위치）
- `dialogue_style`: 대화 연출（말풍선 스타일, 애니메이션）
- `personality`: 성격 특성（미래의 AI 대화 생성용）

#### 2. Dialogue 씬 정의（미래 사양）

**dialogue.json** 의 구조（구현은 Phase 10 이후）:

```json
{
  "scene_id": "intro-dialogue",
  "type": "dialogue",
  "content": {
    "duration_ms": 15000,
    "exchanges": [
      {
        "character_id": "user",
        "text": "이 기능은 무엇을 할 수 있나요?",
        "timing_ms": 0,
        "duration_ms": 3000,
        "emotion": "curious"
      },
      {
        "character_id": "guide",
        "text": "간단히 설명합니다. 먼저...",
        "timing_ms": 3500,
        "duration_ms": 5000,
        "emotion": "friendly"
      },
      {
        "character_id": "narrator",
        "text": "실제 화면을 봅시다",
        "timing_ms": 9000,
        "duration_ms": 3000,
        "emotion": "neutral"
      }
    ]
  },
  "characters": [
    {
      "$ref": "characters/user.json"
    },
    {
      "$ref": "characters/guide.json"
    },
    {
      "$ref": "characters/narrator.json"
    }
  ],
  "direction": {
    "layout": "split-screen",
    "transition_between_speakers": "highlight"
  }
}
```

#### 3. TTS 연계의 확장 방법

**현재（단일 음성）**:
```javascript
// 1つの音声ファイルを再生
<Audio src={staticFile('narration.wav')} />
```

**Phase 10 확장 후（캐릭터별 음성）**:
```javascript
// 캐릭터별로 TTS 호출
async function generateDialogue(exchanges, characters) {
  const audioFiles = await Promise.all(
    exchanges.map(async (exchange) => {
      const character = characters.find(c => c.character_id === exchange.character_id);

      // TTS API 호출（프로바이더에 따라 분기）
      const audioBuffer = await ttsProvider.synthesize({
        text: exchange.text,
        voiceId: character.voice.voice_id,
        speed: character.voice.speed,
        emotion: exchange.emotion,
      });

      return {
        character_id: exchange.character_id,
        audio: audioBuffer,
        timing_ms: exchange.timing_ms,
        duration_ms: exchange.duration_ms,
      };
    })
  );

  return audioFiles;
}
```

**TTS 프로바이더 연계**:

| 프로바이더 | API 호출 예 |
|-------------|---------------|
| Google Cloud TTS | `textToSpeech.synthesizeSpeech({ voice, input })` |
| ElevenLabs | `elevenlabs.textToSpeech({ voiceId, text })` |
| OpenAI TTS | `openai.audio.speech.create({ voice, input })` |
| AWS Polly | `polly.synthesizeSpeech({ VoiceId, Text })` |

#### 4. 비주얼 연출의 확장

**캐릭터 표시（Remotion 컴포넌트 예）**:

```tsx
// 将来的実装: DialogueScene.tsx
const DialogueScene: React.FC<{
  exchanges: Exchange[];
  characters: Character[];
}> = ({ exchanges, characters }) => {
  const frame = useCurrentFrame();

  return (
    <AbsoluteFill>
      {/* 배경 */}
      <Background />

      {/* 캐릭터 표시 */}
      <CharacterDisplay
        characters={characters}
        activeCharacterId={getCurrentSpeaker(frame, exchanges)}
      />

      {/* 대화 텍스트（말풍선） */}
      <DialogueBubble
        exchange={getCurrentExchange(frame, exchanges)}
      />

      {/* 음성 재생 */}
      {exchanges.map((ex, i) => (
        <Sequence from={ex.timing_ms / 33.33} durationInFrames={ex.duration_ms / 33.33}>
          <Audio src={staticFile(`dialogue/${ex.character_id}_${i}.wav`)} />
        </Sequence>
      ))}
    </AbsoluteFill>
  );
};
```

**애니메이션 예**:
- 话している 캐릭터를ハイライト
- 话していない 캐릭터는 반투명
-  말풍선이 페이드인/아웃
- 캐릭터 아바타가 口パク（선택）

#### 5. 구현 로드맵（Phase 10 이후）

| Phase | 구현 내용 | 우선순위 |
|-------|---------|--------|
| **Phase 10.1** | `character.schema.json` 구현 | ✅ 완료 |
| **Phase 10.2** | TTS 프로바이더 연계（Google Cloud TTS）| High |
| **Phase 10.3** | `DialogueScene` Remotion 컴포넌트 | High |
| **Phase 10.4** | `dialogue.json` 스키마 정의 | Medium |
| **Phase 10.5** | 캐릭터 표시 UI（아바타/아이콘）| Medium |
| **Phase 10.6** | 말풍선 애니메이션 | Low |
| **Phase 10.7** | 복수 TTS 프로바이더 대응（ElevenLabs, OpenAI）| Low |
| **Phase 10.8** | AI 대화 생성（personality 에 따른 자동 생성）| Future |

#### 6. 호환성의 유지

확장은 **하위 호환성을保つ** 설계:

```
기존 video-script.json（단일 내레이션）
    ↓ 그대로 동작
새 dialogue.json（대화 형식）
    ↓ 새로운 씬 타입으로 추가
양자 共存 가능
```

**scene.schema.jsonへの追加**:
```json
{
  "type": {
    "enum": [
      "intro",
      "ui-demo",
      "dialogue",  // ← Phase 10 で追加
      "..."
    ]
  }
}
```

#### 7. 参考実装

기존 프로젝트의 예:
- **Manim Community**: 캐릭터 애니메이션
- **Remotion Templates**: 대화 형식 템플릿
- **Google Cloud TTS**: 다국어·다음성 대응

---

### Phase 10実装時のチェックリスト

将来実装する際は以下を確認：

- [ ] `character.schema.json` が有効（이미 Phase 10.1 로 완료）
- [ ] TTS API 키가 설정됨（Google Cloud TTS 권장）
- [ ] `dialogue.json` 스키마를 정의
- [ ] `DialogueScene.tsx` Remotion 컴포넌트 구현
- [ ] 캐릭터 음성 파일의 명명 규칙 통일
- [ ] 말풍선 스타일의 브랜드 일관성
- [ ] 기존 씬（intro, ui-demo 등）との共存 테스트
- [ ] 性能: 복수 음성의 동시 렌더링 최적화

---

### 정리（Phase 10）

**현황**: 단일 내레이션 영상에 대응
**Phase 10 설계**: 캐릭터 대화 영상으로의 확장 포인트明確化
**이미 구현됨**: `character.schema.json`（캐릭터 정의）
**미구현**: TTS 연계, 대화 씬, 비주얼 연출（미래 구현）

이 설계에 의해, 장래적으로以下が可能になります：
- 복수 캐릭터의 대화 형식 영상
- 캐릭터별 음성 스타일
- 시각적 캐릭터 표시와 대화 연출
- AI 에 의한 대화 생성（personality 설정에 따른）
