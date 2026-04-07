# 마이그레이션 가이드: 스키마 명명 및 단위 표준화 (11.2)

**Date**: 2026-02-03
**Phase**: 11.2 - Naming & Unit Standardization
**Status**: ✅ Complete

---

## 개요

이번 마이그레이션은 모든 영상 생성 스키마에서 명명 규칙과 시간 단위를 표준화합니다.

### 주요 변경 사항

1. **시간 단위**: 모든 `duration_frames` → `duration_ms` (밀리초)
2. **트랜지션 열거형**: `"slideIn"` → `"slide_in"` (snake_case)
3. **속성 명명**: 모든 camelCase → snake_case
4. **열거형 값**: 소문자와 밑줄의 표준화

---

## 수정된 파일

### 1. direction.schema.json

| Before | After | 이유 |
|--------|-------|--------|
| `duration_frames` | `duration_ms` | 시간 단위 표준화 |
| `"slideIn"` enum | `"slide_in"` | 열거형 명명 규칙 |
| `primaryColor` | `primary_color` | 속성 명명 규칙 |
| `secondaryColor` | `secondary_color` | 속성 명명 규칙 |
| `delay_before` (frames) | `delay_before_ms` | 시간 단위 표준화 |
| `delay_after` (frames) | `delay_after_ms` | 시간 단위 표준화 |
| `audio_start_offset` (frames) | `audio_start_offset_ms` | 시간 단위 표준화 |

**기본값 변경**:
- `duration_ms`: 15 frames @ 30fps → 500ms
- `audio_start_offset_ms`: 30 frames @ 30fps → 1000ms

### 2. animation.schema.json

| Before | After | 이유 |
|--------|-------|--------|
| `duration_frames` | `duration_ms` | 시간 단위 표준화 |
| `"slideIn"` enum | `"slide_in"` | 열거형 명명 규칙 |
| `delay` | `delay_ms` | 시간 단위 표준화 |
| `overshootClamping` | `overshoot_clamping` | 속성 명명 규칙 |
| `inputRange` | `input_range` | 속성 명명 규칙 |
| `outputRange` | `output_range` | 속성 명명 규칙 |
| `extrapolateLeft` | `extrapolate_left` | 속성 명명 규칙 |
| `extrapolateRight` | `extrapolate_right` | 속성 명명 규칙 |

**최대 기간**: 300 frames → 10000ms (10초)

### 3. emphasis.schema.json

| Before | After | 이유 |
|--------|-------|--------|
| `start_frame` | `start_ms` | 시간 단위 표준화 |
| `duration_frames` | `duration_ms` | 시간 단위 표준화 |
| `trigger_frame` | `trigger_ms` | 시간 단위 표준화 |
| `glowIntensity` | `glow_intensity` | 속성 명명 규칙 |
| `borderRadius` | `border_radius` | 속성 명명 규칙 |
| `fontSize` | `font_size` | 속성 명명 규칙 |
| `fontWeight` | `font_weight` | 속성 명명 규칙 |
| `fontFamily` | `font_family` | 속성 명명 규칙 |
| `lineHeight` | `line_height` | 속성 명명 규칙 |
| `letterSpacing` | `letter_spacing` | 속성 명명 규칙 |
| `textTransform` | `text_transform` | 속성 명명 규칙 |
| `pulseSpeed` | `pulse_speed` | 속성 명명 규칙 |
| `"fadeIn"` enum | `"fade_in"` | 열거형 명명 규칙 |
| `"slideIn"` enum | `"slide_in"` | 열거형 명명 규칙 |
| `"zoomIn"` enum | `"zoom_in"` | 열거형 명명 규칙 |
| `"fadeOut"` enum | `"fade_out"` | 열거형 명명 규칙 |
| `"slideOut"` enum | `"slide_out"` | 열거형 명명 규칙 |
| `"zoomOut"` enum | `"zoom_out"` | 열거형 명명 규칙 |

**기본값 변경**:
- `duration_ms`: 30 frames @ 30fps → 1000ms
- `animation.duration_ms`: 15 frames @ 30fps → 500ms
- `pulse_speed`: 0.1 (per-frame) → 1.0 (per-second, 1 cycle/sec)

### 4. scene.schema.json

| Before | After | 이유 |
|--------|-------|--------|
| `"slideIn"` enum (transition.in) | `"slide_in"` | 열거형 명명 규칙 |
| `"slideIn"` enum (transition.out) | `"slide_in"` | 열거형 명명 규칙 |

### 5. video-script.schema.json

| Before | After | 이유 |
|--------|-------|--------|
| `"slide"` enum | `"slide_in"` | 열거형 명명 규칙 |
| `"none"` enum | `"cut"` | 열거형 표준화 |

### 6. visual-patterns.schema.json

| Before | After | 이유 |
|--------|-------|--------|
| `colorScheme` | `color_scheme` | 속성 명명 규칙 |
| `leftSide` | `left_side` | 속성 명명 규칙 |
| `rightSide` | `right_side` | 속성 명명 규칙 |
| `arrowStyle` | `arrow_style` | 속성 명명 규칙 |
| `showNumbers` | `show_numbers` | 속성 명명 규칙 |
| `mainText` | `main_text` | 속성 명명 규칙 |
| `subText` | `sub_text` | 속성 명명 규칙 |
| `fontSize` | `font_size` | 속성 명명 규칙 |
| `aspectRatio` | `aspect_ratio` | 속성 명명 규칙 |

---

## 호환성이 없는 변경 사항

### JSON 작성자용

이전 명명을 사용하는 기존 JSON 파일이 있는 경우:

```json
// ❌ 이전 형식（더 이상 유효하지 않음）
{
  "transition": {
    "type": "slideIn",
    "duration_frames": 15
  },
  "emphasis": {
    "text": [{
      "start_frame": 0,
      "duration_frames": 30
    }]
  }
}

// ✅ 새 형식（필수）
{
  "transition": {
    "type": "slide_in",
    "duration_ms": 500
  },
  "emphasis": {
    "text": [{
      "start_ms": 0,
      "duration_ms": 1000
    }]
  }
}
```

### 코드용

해당 속성에 액세스하는 TypeScript/JavaScript 코드가 있는 경우:

```typescript
// ❌ 이전 코드（변경됨）
const duration = scene.transition.duration_frames;
const color = scene.background.primaryColor;

// ✅ 새 코드
const duration = scene.transition.duration_ms;
const color = scene.background.primary_color;
```

### 검증기용

JSON Schema 검증기는 이전 속성명을 거부합니다. 모든 참조를 업데이트하세요.

---

## 변환 공식

### Frames에서 Milliseconds로

```javascript
// 30 FPS 가정（영상 생성의 표준）
const fps = 30;
const durationMs = Math.floor((durationFrames / fps) * 1000);

// 예시:
// 15 frames → 500ms
// 30 frames → 1000ms
// 60 frames → 2000ms
```

### Milliseconds에서 Frames로 (런타임)

```javascript
// Remotion으로 렌더링할 때
const fps = outputSettings.fps; // video-script.schema.json에서
const durationFrames = Math.floor((durationMs / 1000) * fps);
```

---

## 검증

모든 스키마가以下の 검증을 통과했습니다:

- ✅ `duration_frames` 속성이 남아 있지 않음
- ✅ 모든 트랜지션 열거형이 `slide_in` 사용（`slideIn` 아님）
- ✅ 모든 속성이 `snake_case` 사용（camelCase 아님）
- ✅ 모든 열거형 값이 소문자와 밑줄 사용

### 직접 검증 실행

```bash
node -e "
const fs = require('fs');
const schemas = fs.readdirSync('./skills/generate-video/schemas')
  .filter(f => f.endsWith('.schema.json'))
  .map(f => './skills/generate-video/schemas/' + f);

schemas.forEach(file => {
  const content = fs.readFileSync(file, 'utf8');
  const issues = [];

  if (content.includes('duration_frames')) issues.push('duration_frames found');
  if (content.includes('\"slideIn\"')) issues.push('slideIn found');

  if (issues.length) {
    console.log(file, '❌', issues.join(', '));
  } else {
    console.log(file, '✅');
  }
});
"
```

---

## 롤백 지침

변경 사항을 롤백해야 하는 경우:

```bash
# 이전 커밋으로 모든 스키마 롤백
git checkout HEAD~1 skills/generate-video/schemas/

# 명명 규칙 문서 삭제
rm skills/generate-video/references/naming-conventions.md

# 이 마이그레이션 가이드 삭제
rm skills/generate-video/references/MIGRATION-11.2.md
```

---

## 다음 단계

1. **기존 테스트 데이터 업데이트** - 새 명명 규칙 사용
2. **코드 생성기 업데이트** (있는 경우) - 새 형식 출력
3. **이전 속성명을 참조하는 문서 업데이트**
4. **하위 호환성이 필요한 경우 버전 관리 고려**

---

## 관련 문서

- [Naming Conventions](./naming-conventions.md) - 포괄적 명명 규칙
- [Schema Phase Plan](../PLANS.md) - Phase 11.2 태스크 상세
- [All Schemas](../schemas/) - 업데이트된 스키마 파일

---

## 체크리스트

다음 경우에 마이그레이션이 완료됩니다:

- [x] 모든 스키마가 `duration_frames` 대신 `duration_ms` 사용
- [x] 모든 트랜지션 열거형이 `["fade", "slide_in", "zoom", "cut"]` 로 표준화
- [x] 모든 속성이 `snake_case` 사용
- [x] 모든 열거형 값이 소문자와 밑줄 사용
- [x] 명명 규칙 문서 작성됨
- [x] 마이그레이션 가이드 작성됨
- [x] 모든 스키마가 검증 통과

**Status**: ✅ Complete (2026-02-03)
