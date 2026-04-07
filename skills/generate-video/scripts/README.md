# Scripts Directory - 스크립트 디렉토리

JSON Schema 자동 생성 및 검증용 스크립트 모음.

## Available Scripts

### generate-schemas.js

JSON Schema에서 Zod 스키마를 자동 생성합니다.

**Usage:**
```bash
npm run generate:schemas
```

**Input:**
- `schemas/*.schema.json` - JSON Schema 파일

**Output:**
- `src/schemas/*.ts` - Zod 스키마 정의
- `src/schemas/index.ts` - 배럴 export

**Example:**
```bash
# Generate all schemas
node scripts/generate-schemas.js

# Or via npm script (권장)
npm run generate:schemas
```

**Dependencies:**
- `json-schema-to-zod` - JSON Schema → Zod 변환
- `zod` - 런타임 검증

---

## Setup

### Install Dependencies

스키마 생성을 위해 필요한 패키지를 설치합니다:

```bash
npm install --save-dev json-schema-to-zod
npm install zod
```

### Add npm Script

`package.json` 에 다음을 추가합니다:

```json
{
  "scripts": {
    "generate:schemas": "node scripts/generate-schemas.js"
  }
}
```

### Pre-commit Hook (선택)

스키마 변경時に自動生成:

```bash
# .husky/pre-commit
npm run generate:schemas
git add src/schemas/
```

---

## Schema Development Workflow

1. **스키마 작성**: `schemas/*.schema.json` 생성
2. **생성 실행**: `npm run generate:schemas`
3. **타입 추론 확인**: `src/schemas/*.ts` 에서 TypeScript 타입 확인
4. **검증**: 생성된 Zod 스키마로 검증

### Example

```typescript
// src/example.ts
import { AssetManifestSchema, type AssetManifest } from './schemas';

// Runtime validation
const data: unknown = { /* ... */ };
const result = AssetManifestSchema.safeParse(data);

if (result.success) {
  const manifest: AssetManifest = result.data;
  console.log('Valid manifest:', manifest);
} else {
  console.error('Validation errors:', result.error.errors);
}
```

---

## Schema Versioning

### Version Format

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "version": "1.0.0",
  "title": "SchemaName",
  ...
}
```

### Breaking Changes

메이저 버전을 올려야 하는 변경:
- Required 필드의 추가
- 필드의 삭제
- 타입의 변경

마이너 버전에서 가능한 변경:
- Optional 필드의 추가
- Enum 값의 추가
- Description의 변경

---

## Troubleshooting

### Schema Generation Errors

**Error**: `Cannot find module 'json-schema-to-zod'`
```bash
npm install --save-dev json-schema-to-zod
```

**Error**: `No .schema.json files found`
- `schemas/` 디렉토리에 `*.schema.json` 파일이 있는지 확인

**Error**: `Invalid JSON`
- JSON Schema의 구문 에러를 체크합니다
- [JSONLint](https://jsonlint.com/) 로 검증

### Zod Schema Issues

**Type inference not working**
```typescript
// ❌ Bad
const schema = AssetManifestSchema;

// ✅ Good
import { type AssetManifest } from './schemas';
const manifest: AssetManifest = { /* ... */ };
```

---

## 검증 스크립트 (Phase 2)

### validate-scene.js

개별 씬 JSON을 `scene.schema.json` 에 대해 검증합니다.

**Usage:**
```bash
node scripts/validate-scene.js <scene-file.json>
```

**Example:**
```bash
node scripts/validate-scene.js schemas/examples/scene-example.json
```

**Output:**
```json
{
  "valid": true,
  "errors": []
}
```

**Exit Codes:**
- `0` - 검증 성공
- `1` - 검증 실패 (스키마 에러)
- `2` - 파일을 찾을 수 없거나 유효하지 않은 JSON

---

### validate-scenario.js

시나리오 JSON을 `scenario.schema.json` 에 대해 검증합니다.
시맨틱 체크도 실행:
- 섹션 ID의 고유성
- 섹션 순서의 correctness
- Duration의 타당성

**Usage:**
```bash
node scripts/validate-scenario.js <scenario-file.json>
```

**Example:**
```bash
node scripts/validate-scenario.js schemas/examples/scenario-example.json
```

**Semantic Checks:**
- ✅ Section ID uniqueness
- ✅ Section order sequence (0, 1, 2, ...)
- ✅ Duration estimates (negative, excessive values)

**Exit Codes:**
- `0` - 검증 성공
- `1` - 검증 실패 (스키마 또는 시맨틱 에러)
- `2` - 파일을 찾을 수 없거나 유효하지 않은 JSON

---

### validate-video.js

완전한 비디오 스크립트 JSON을 E2E로 검증합니다.
Critical 에러는 중지, Warning은 로그 출력 후 계속.

**Usage:**
```bash
node scripts/validate-video.js <video-script-file.json>
```

**Example:**
```bash
node scripts/validate-video.js schemas/examples/video-script-example.json
```

**E2E Validation Checks:**
- ✅ Scene ID uniqueness (모든 씬에서)
- ✅ Scene order sequence (각 섹션 내에서)
- ✅ Total duration calculation
- ⚠️ Asset file existence
- ⚠️ Audio sync validation
- ⚠️ Resolution/aspect ratio

**Severity Levels:**
| Level | Behavior | Examples |
|-------|----------|----------|
| **Critical** | 검증 중지, exit code 1 | Duplicate IDs, invalid schema |
| **Warning** | 경고 로그, 계속 | Missing assets, unusual aspect ratio |

**Output:**
```json
{
  "valid": true,
  "errors": [],
  "warnings": [
    {
      "severity": "warning",
      "path": "/scenes/0/assets/0/source",
      "message": "Asset not found: \"assets/intro.png\"",
      "keyword": "asset-missing"
    }
  ]
}
```

**Exit Codes:**
- `0` - 검증 성공 (경고는 괜찮음)
- `1` - 검증 실패 (critical 에러)
- `2` - 파일을 찾을 수 없거나 유효하지 않은 JSON

---

## Asset Management (Phase 7)

### load-assets.js

에셋（배경, 효과음, 폰트, 이미지）의 로드 및 사용자 오버라이드 대응.

**Priority System:**
1. User assets: `~/.harness/video/assets/`
2. Skill defaults: `skills/generate-video/assets/`
3. Built-in defaults: 하드코딩된 폴백

**Usage:**
```bash
# Load backgrounds configuration
node scripts/load-assets.js backgrounds

# Load sounds configuration
node scripts/load-assets.js sounds

# Show asset search paths
node scripts/load-assets.js paths

# Initialize user asset directory
node scripts/load-assets.js init

# Test all loading functions
node scripts/load-assets.js test
```

**Programmatic Usage:**
```javascript
const { loadBackgrounds, loadSounds, loadAssetFile } = require('./scripts/load-assets.js');

// Load configurations
const backgrounds = loadBackgrounds();
// → { version: "1.0.0", backgrounds: [...] }

const sounds = loadSounds();
// → { version: "1.0.0", sounds: [...] }

// Load specific asset file
const assetPath = loadAssetFile('sounds', 'impact.mp3');
// → "/path/to/impact.mp3" or null
```

**Functions:**
- `loadBackgrounds()` - 배경 구성 로드
- `loadSounds()` - 효과음 구성 로드
- `loadAssetFile(category, filename)` - 특정 에셋 파일 로드
- `updateManifest(manifestPath, assets)` - 에셋 매니페스트 업데이트
- `getAssetPaths()` - 에셋 검색 경로 가져오기 (debug)
- `initUserAssetDir()` - `~/.harness/video/assets/` 초기화

**Asset Types:**
- **backgrounds** - 5 타입: neutral, highlight, dramatic, tech, warm
- **sounds** - 4 타입: impact, pop, transition, subtle
- **fonts** - 커스텀 폰트 파일 (TTF, OTF, WOFF)
- **images** - 커스텀 이미지 (PNG, JPG, SVG, WebP)

**Customization:**
See [references/asset-customization.md](../references/asset-customization.md) for detailed customization guide.

**Test:**
```bash
npm test -- asset-loader.test.js
```

---

## Future Scripts (Phase 3+)

향후 추가 예정인 스크립트:

- `merge-scenes.js` - 씬 JSON 머지
- `optimize-assets.js` - 에셋 최적화
- `generate-thumbnails.js` - 썸네일 자동 생성
- `render-video.js` - 비디오 렌더링 (Phase 8)

---

## References

- [JSON Schema](https://json-schema.org/)
- [Zod Documentation](https://zod.dev/)
- [json-schema-to-zod](https://github.com/StefanTerdell/json-schema-to-zod)
- [Asset Customization Guide](../references/asset-customization.md)
