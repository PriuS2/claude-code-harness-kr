# Asset Customization Guide - 에셋 사용자 정의 가이드

사용자 정의 에셋（배경, 효과음, 폰트, 이미지）의 오버라이드 방법과 베스트 프랙티스.

---

## 개요

영상 생성에서 사용하는 에셋は、以下の優先順位で読み込まれます：

```
1. ユーザーアセット (~/.harness/video/assets/)    ← 最優先
2. スキルデフォルト (skills/generate-video/assets/) ← フォールバック
3. ビルトインデフォルト (ハードコード)              ← 最終手段
```

이 메커니즘을 통해 스킬 본체를 변경하지 않고 자신만의 에셋을 사용할 수 있습니다.

---

## 디렉토리 구조

### 사용자 에셋 디렉토리

```
~/.harness/video/assets/
├── README.md                    # 사용 가이드（자동 생성）
├── backgrounds/
│   ├── backgrounds.json         # 커스텀 배경 정의
│   └── my-custom-bg.png         # 커스텀 배경 이미지（선택）
├── sounds/
│   ├── sounds.json              # 커스텀 효과음 정의
│   ├── impact.mp3               # 고 강조음
│   ├── pop.mp3                  # 중 강조음
│   ├── transition.mp3           # 장면 전환음
│   └── subtle.mp3               # 저 강조음
├── fonts/
│   ├── MyBrand-Bold.ttf
│   └── MyBrand-Regular.ttf
└── images/
    ├── logo.png
    └── icon.png
```

### 초기화

사용자 에셋 디렉토리 생성:

```bash
node scripts/load-assets.js init
```

또는 수동으로 생성:

```bash
mkdir -p ~/.harness/video/assets/{backgrounds,sounds,fonts,images}
```

---

## 사용자 정의 방법

### 1. 배경 사용자 정의

#### 절차

1. **기본 설정을 복사**:

```bash
cp skills/generate-video/assets/backgrounds/backgrounds.json \
   ~/.harness/video/assets/backgrounds/
```

2. **설정을 편집**:

```json
{
  "version": "1.0.0",
  "backgrounds": [
    {
      "id": "my-brand",
      "name": "My Brand Background",
      "description": "Company brand colors",
      "type": "gradient",
      "colors": {
        "primary": "#1e3a8a",
        "secondary": "#3b82f6",
        "accent": "#60a5fa"
      },
      "gradient": {
        "type": "linear",
        "angle": 135,
        "stops": [
          { "color": "#1e3a8a", "position": 0 },
          { "color": "#3b82f6", "position": 50 },
          { "color": "#60a5fa", "position": 100 }
        ]
      },
      "usage": {
        "scenes": ["intro", "cta"],
        "recommended_for": "Brand-focused content"
      }
    }
  ]
}
```

3. **영상 생성에서 사용**:

```json
{
  "scene": {
    "background": "my-brand"
  }
}
```

#### 배경 타입

| Type | Description | Fields |
|------|-------------|--------|
| `gradient` | 그라데이션 배경 | `colors`, `gradient` |
| `pattern` | 패턴 배경（グリッド等）| `colors`, `gradient`, `pattern` |
| `solid` | 단색 배경 | `colors.primary` |
| `image` | 이미지 배경 | `file` (path to image) |

#### 그라데이션 타입

```json
// Linear gradient
"gradient": {
  "type": "linear",
  "angle": 135,
  "stops": [...]
}

// Radial gradient
"gradient": {
  "type": "radial",
  "stops": [...]
}
```

---

### 2. 효과음 사용자 정의

#### 절차

1. **기본 설정을 복사**:

```bash
cp skills/generate-video/assets/sounds/sounds.json \
   ~/.harness/video/assets/sounds/
```

2. **효과음 파일을 배치**:

```bash
# FreeSoundからダウンロード（CC0ライセンス推奨）
cp ~/Downloads/my-impact.mp3 ~/.harness/video/assets/sounds/impact.mp3
cp ~/Downloads/my-pop.mp3 ~/.harness/video/assets/sounds/pop.mp3
```

3. **설정을 편집**:

```json
{
  "version": "1.0.0",
  "sounds": [
    {
      "id": "impact",
      "name": "Custom Impact",
      "type": "effect",
      "category": "emphasis",
      "emphasis_level": "high",
      "file": {
        "placeholder": "impact.mp3",
        "expected_duration": 0.5,
        "format": "mp3"
      },
      "volume": {
        "default": 0.7,
        "with_narration": 0.4,
        "with_bgm": 0.6
      }
    }
  ]
}
```

#### 권장 형식

| Format | Sample Rate | Bit Depth | Notes |
|--------|-------------|-----------|-------|
| MP3 | 44100 Hz | 16-bit | 推奨（互換性高）|
| WAV | 44100 Hz | 16-bit | 高品質（ファイルサイズ大）|
| OGG | 44100 Hz | - | 軽量（ブラウザ互換性注意）|

#### 볼륨 권장값

| Context | Volume Range | Notes |
|---------|--------------|-------|
| 내레이션 있음 | 0.15 - 0.4 | 음성을 방해하지 않음 |
| BGM 있음 | 0.25 - 0.6 | BGM을 대킹 |
| 음성 없음 | 0.3 - 1.0 | 풀 볼륨 OK |

---

### 3. 폰트 사용자 정의

#### 절차

1. **폰트 파일을 배치**:

```bash
cp ~/Downloads/MyFont-Bold.ttf ~/.harness/video/assets/fonts/
cp ~/Downloads/MyFont-Regular.ttf ~/.harness/video/assets/fonts/
```

2. **씬 설정에서 참조**:

```json
{
  "scene": {
    "text": {
      "content": "My Message",
      "font": {
        "family": "MyFont",
        "weight": "bold",
        "file": "~/.harness/video/assets/fonts/MyFont-Bold.ttf"
      }
    }
  }
}
```

#### Remotionでの使用

```typescript
import { loadFont } from '@remotion/google-fonts/Inter';

// カスタムフォント読み込み
const fontFamily = loadFont({
  src: '~/.harness/video/assets/fonts/MyFont-Bold.ttf',
  fontFamily: 'MyFont',
  fontWeight: 'bold',
});
```

#### 권장 형식

| Format | Web Safe | Notes |
|--------|----------|-------|
| TTF | ✅ Yes | 推奨（最も互換性が高い）|
| OTF | ✅ Yes | OpenType機能が使える |
| WOFF/WOFF2 | ✅ Yes | Web最適化（軽量）|

---

### 4. 이미지의 사용자 정의

#### 절차

1. **이미지 파일을 배치**:

```bash
cp ~/Downloads/logo.png ~/.harness/video/assets/images/
cp ~/Downloads/icon.png ~/.harness/video/assets/images/
```

2. **씬 설정에서 참조**:

```json
{
  "scene": {
    "image": {
      "src": "~/.harness/video/assets/images/logo.png",
      "width": 200,
      "height": 100
    }
  }
}
```

#### 권장 형식

| Format | Use Case | Notes |
|--------|----------|-------|
| PNG | 로고, 아이콘 | 투명도対応 |
| JPG | 사진, 배경 |圧縮率高 |
| SVG | 벡터 도형 |拡大しても綺麗 |
| WebP | 모던 환경 | 軽量高品質 |

#### 사이즈 가이드라인

| Asset Type | Recommended Size | Max Size |
|------------|------------------|----------|
| Logo | 500x500 px | 1000x1000 px |
| Icon | 128x128 px | 512x512 px |
| Background | 1920x1080 px | 3840x2160 px |
| Screenshot | 1920x1080 px | 2560x1440 px |

---

## 우선순위 상세

###読み込み順序

`scripts/load-assets.js` は以下の順序でアセットを検索:

```javascript
// 1. ユーザーアセット
const userPath = '~/.harness/video/assets/{category}/{file}';
if (exists(userPath)) return userPath;

// 2. スキルデフォルト
const skillPath = 'skills/generate-video/assets/{category}/{file}';
if (exists(skillPath)) return skillPath;

// 3. ビルトインデフォルト
return getBuiltInDefault();
```

### 부분 오버라이드

일부 에셋만 오버라이드 가능:

```bash
# 배경만 사용자 정의（효과음은 기본값 사용）
cp my-backgrounds.json ~/.harness/video/assets/backgrounds/backgrounds.json
```

### JSON内の 부분 오버라이드

```json
// ~/.harness/video/assets/backgrounds/backgrounds.json
{
  "version": "1.0.0",
  "backgrounds": [
    {
      "id": "my-brand",
      "name": "My Brand"
      // ... カスタム設定
    }
    // "neutral", "highlight" 等は省略 → デフォルトから読み込まれる
  ]
}
```

**注意**: 같은 `id`가 있는 경우、ユーザー設定が優先されます。

---

## 동작 확인

### 테스트 명령

```bash
# アセット読み込みテスト
node scripts/load-assets.js test

# 背景設定表示
node scripts/load-assets.js backgrounds

# 効果音設定表示
node scripts/load-assets.js sounds

# 検索パス表示
node scripts/load-assets.js paths
```

###期待される出力

```
🧪 Testing asset loader...

🎨 Loading backgrounds...
  ✅ Loaded user backgrounds from: ~/.harness/video/assets/backgrounds/backgrounds.json

🔊 Loading sounds...
  ✅ Loaded skill sounds from: skills/generate-video/assets/sounds/sounds.json

📂 Asset paths:
{
  "user": "~/.harness/video/assets",
  "skill": "skills/generate-video/assets"
}
```

---

## 트러블슈팅

### 문제: 에셋이読み込まれない

**원인**: 파일 경로가 잘못됨

**해결책**:
```bash
# パスを確認
node scripts/load-assets.js paths

# ファイルの存在確認
ls -la ~/.harness/video/assets/backgrounds/
```

### 문제: JSON解析エラー

**원인**: JSON 형식이 잘못됨

**해결책**:
```bash
# JSONの妥当性チェック
cat ~/.harness/video/assets/backgrounds/backgrounds.json | jq .

# エラーメッセージを確認
node scripts/load-assets.js test
```

### 문제: 효과음이 재생되지 않음

**원인**: 파일 형식이 지원되지 않음

**해결책**:
```bash
# MP3に変換
ffmpeg -i input.wav -codec:a libmp3lame -b:a 192k output.mp3

# ファイル情報確認
ffprobe output.mp3
```

### 문제: 폰트가 표시되지 않음

**원인**: 폰트 파일 경로가 해결되지 않음

**해결책**:
```typescript
// 絶対パスを使用
const fontPath = path.join(os.homedir(), '.harness/video/assets/fonts/MyFont.ttf');
```

---

## 베스트 프랙티스

### 1. 버전 관리

커스텀 에셋을 Git 관리하고 싶은 경우:

```bash
# プロジェクトルートに配置
project-root/
├── .video-assets/
│   ├── backgrounds/
│   ├── sounds/
│   └── fonts/
└── .gitignore  # .harness/ は除外

# シンボリックリンク作成
ln -s $(pwd)/.video-assets ~/.harness/video/assets
```

### 2. 팀 공유

팀으로 공통 에셋 사용:

```bash
# 共有リポジトリ
git clone https://github.com/company/video-assets.git ~/.harness/video/assets
```

### 3. 프로젝트별 에셋

프로젝트마다 다른 에셋:

```bash
# 環境変数で切り替え
export VIDEO_ASSETS_DIR=/path/to/project-specific/assets

# load-assets.js で環境変数を参照
const assetsDir = process.env.VIDEO_ASSETS_DIR || defaultPath;
```

### 4. 라이선스 관리

```
~/.harness/video/assets/
└── LICENSES.md    # 各アセットのライセンス情報
```

```markdown
# Asset Licenses

## Sounds

- impact.mp3: CC0, from freesound.org/s/12345
- pop.mp3: CC BY 3.0, by Author Name

## Fonts

- MyFont-Bold.ttf: SIL Open Font License
```

---

## 샘플 모음

### 브랜드 컬러 배경

```json
{
  "id": "brand-primary",
  "name": "Brand Primary",
  "type": "gradient",
  "colors": {
    "primary": "#your-brand-color",
    "secondary": "#your-secondary-color"
  },
  "gradient": {
    "type": "linear",
    "angle": 135,
    "stops": [
      { "color": "#your-brand-color", "position": 0 },
      { "color": "#your-secondary-color", "position": 100 }
    ]
  },
  "usage": {
    "scenes": ["intro", "outro", "cta"]
  }
}
```

### 커스텀 효과음 세트

```json
{
  "id": "whoosh",
  "name": "Whoosh Transition",
  "type": "effect",
  "category": "transition",
  "file": {
    "placeholder": "whoosh.mp3",
    "expected_duration": 0.6
  },
  "volume": {
    "default": 0.5,
    "with_narration": 0.3
  },
  "timing": {
    "offset_before_visual": -0.1
  }
}
```

### 기업 로고

```json
{
  "scene": {
    "image": {
      "src": "~/.harness/video/assets/images/company-logo.png",
      "width": 300,
      "height": 150,
      "position": "top-right"
    }
  }
}
```

---

## 참조

- **Asset Loader**: `scripts/load-assets.js`
- **Default Backgrounds**: `assets/backgrounds/backgrounds.json`
- **Default Sounds**: `assets/sounds/sounds.json`
- **BackgroundLayer Component**: `remotion/src/components/BackgroundLayer.tsx`
- **Plans.md**: Phase 7 - Asset Foundation

---

## 업데이트 로그

- **2026-02-02**: 초판 작성（Phase 7 구현）
