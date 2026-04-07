# Naming Conventions for Video Generation Schemas - 영상 생성 스키마 명명 규칙

이 문서는 영상 생성 시스템의 모든 JSON 스키마에 대한 통합 명명 규칙을 정의합니다.

## Version
**1.0.0** - 2026-02-03

---

## 1. 시간 단위

### 규칙
**모든 시간 지속 시간은 반드시 밀리초（`_ms` 접미사）를 사용해야 합니다**

### 이유
- 밀리초는 영상 타이밍에 충분한 정밀도를 제공합니다
- 프레임 카운트는 FPS에 따라 다르며 런타임에 계산해야 합니다
- 모든 스키마에서 일관성 확보

### 예시

```json
// ✅ 올바름
{
  "duration_ms": 5000,
  "start_offset_ms": 1000,
  "fade_in_ms": 500
}

// ❌ 올바르지 않음
{
  "duration_frames": 150,
  "duration": 5,
  "durationSec": 5
}
```

### 런타임시의 변환
```javascript
// FPS는 output_settings에서 제공됨
const fps = 30;
const durationMs = 5000;
const durationFrames = Math.floor((durationMs / 1000) * fps); // 150 frames
```

---

## 2. Transition 타입

### 규칙
**Transition 열거형은 반드시 snake_case 값을 사용해야 합니다**

### 표준 열거형
```json
{
  "enum": ["fade", "slide_in", "zoom", "cut"]
}
```

### 정의

| 값 | 설명 | 사용 상황 |
|-------|-------------|----------|
| `fade` | 점진적 불투명도 변경 | 기본, 은은한 전환 |
| `slide_in` | 방향에서 슬라이드 | 역동적인 씬 변경 |
| `zoom` | 확대/축소 | 강조, 드라마틱한 공개 |
| `cut` | 즉각 컷 (전환 없음) | 빠른 전개 콘텐츠 |

### Direction 속성（slide_in 의 경우）
`transition.type === "slide_in"` 인 경우, `direction` 속성을 사용합니다:

```json
{
  "transition": {
    "type": "slide_in",
    "duration_ms": 500,
    "direction": "left"
  }
}
```

**유효한 방향**: `"left"`, `"right"`, `"top"`, `"bottom"`

---

## 3. 속성 명명 케이스

### 규칙
**모든 속성 이름은 반드시 snake_case를 사용해야 합니다**

### 이유
- 기존 코드베이스 규칙과의 일관성
- 복합어 속성의 가독성 향상
- JSON Schema 모범 사례와의 정렬

### 예시

```json
// ✅ 올바름
{
  "primary_color": "#3B82F6",
  "secondary_color": "#10B981",
  "font_size": 48,
  "font_weight": 700,
  "line_height": 1.5,
  "border_radius": 8,
  "glow_intensity": 20
}

// ❌ 올바르지 않음
{
  "primaryColor": "#3B82F6",
  "fontSize": 48,
  "lineHeight": 1.5,
  "borderRadius": 8
}
```

---

## 4. 열거형 값

### 규칙
**열거형 값은 복합어의 경우 소문자와 하이픈을 사용해야 합니다**

### 표준 패턴

#### 씬 타입
```json
["intro", "ui-demo", "architecture", "code-highlight", "changelog", "cta"]
```

#### 시각 스타일
```json
["minimalist", "technical", "modern", "gradient", "flat", "3d"]
```

#### 애니메이션 이징
```json
["linear", "ease-in", "ease-out", "ease-in-out", "ease-in-quad", "ease-out-quad"]
```

#### 배경 타입
```json
["cyberpunk", "corporate", "minimal", "gradient", "particles"]
```

---

## 5. ID 패턴

### 규칙
**ID는 반드시 kebab-case（小文字とハイフォン）를 사용해야 합니다**

### 패턴
```regex
^[a-z0-9-]+$
```

### 예시

```json
// ✅ 올바름
{
  "scene_id": "intro-hero",
  "section_id": "feature-highlights",
  "character_id": "expert-reviewer"
}

// ❌ 올바르지 않음
{
  "scene_id": "introHero",
  "section_id": "feature_highlights",
  "character_id": "ExpertReviewer"
}
```

---

## 6. 색상 형식

### 규칙
**색상은 반드시 `#` 접두사와 함께 대문자 HEX 형식을 사용해야 합니다**

### 패턴
```regex
^#[0-9A-F]{6}$
```

### 예시

```json
// ✅ 올바름
{
  "primary_color": "#3B82F6",
  "accent_color": "#F59E0B"
}

// ❌ 올바르지 않음
{
  "primary_color": "#3b82f6",  // 소문자
  "accent_color": "3B82F6",    // # 누락
  "text_color": "rgb(59, 130, 246)"  // HEX 아님
}
```

### RGBA 예외
투명도의 경우 `rgba()` 형식을 사용합니다:

```json
{
  "background_color": "rgba(0, 0, 0, 0.8)"
}
```

---

## 7. 예약 키워드

### 음성 속성
- `fade_in_ms` / `fade_out_ms` - 음성 페이드 시간
- `start_offset_ms` - 음성/내레이션 시작 전 지연
- `master_volume` -全局 볼륨（0.0 - 1.0）

### 시각 속성
- `duration_ms` - 밀리초 단위 지속 시간
- `transition` - 전환 구성 객체
- `emphasis` - 강조/하이라이트 구성
- `background` - 배경 구성

### 메타데이터 속성
- `created_at` / `updated_at` - ISO 8601 타임스탬프
- `version` - 시맨틱 버전（예: "1.0.0"）
- `description` - 사람이 읽을 수 있는 설명

---

## 8. 마이그레이션 가이드

### `duration_frames` 에서 `duration_ms` 로

**이전:**
```json
{
  "transition": {
    "type": "fade",
    "duration_frames": 15
  }
}
```

**이후:**
```json
{
  "transition": {
    "type": "fade",
    "duration_ms": 500
  }
}
```

**변환 공식** (30 FPS 가정):
```
duration_ms = (duration_frames / 30) * 1000
```

### `slideIn` 에서 `slide_in` 로

**이전:**
```json
{
  "transition": {
    "type": "slideIn",
    "duration_frames": 15
  }
}
```

**이후:**
```json
{
  "transition": {
    "type": "slide_in",
    "duration_ms": 500,
    "direction": "left"
  }
}
```

### camelCase에서 snake_case로

**이전:**
```json
{
  "background": {
    "primaryColor": "#3B82F6",
    "secondaryColor": "#10B981"
  }
}
```

**이후:**
```json
{
  "background": {
    "primary_color": "#3B82F6",
    "secondary_color": "#10B981"
  }
}
```

---

## 9. 스키마 검증

모든 스키마는以下の 규칙 대해 검증해야 합니다:

### 체크리스트
- [ ] `duration_frames` 속성 없음（`duration_ms` 사용）
- [ ] 트랜지션 열거형: `["fade", "slide_in", "zoom", "cut"]`
- [ ] 모든 속성이 `snake_case` 사용
- [ ] 모든 열거형 값이 `소문자-하이픈` 사용
- [ ] 모든 ID가 패턴 `^[a-z0-9-]+$` 일치
- [ ] 모든 HEX 색상이 패턴 `^#[0-9A-F]{6}$` 일치

---

## 10. 예외

### Character 스키마 (Phase 10+)
`character.schema.json` 은 TTS 프로바이더 API（예: spring 애니메이션의 `overshootClamping`）와의 호환성을 위해 일부 camelCase 속성을 유지할 수 있습니다.

### 외부 API
외부 API（Remotion, TTS 프로바이더）와 인터페이스할 때, 명명 차이를 처리하는 변환 레이어가 있어야 합니다.

---

## 관련 문서

- [Schema Phase Plan](../PLANS.md) - Phase 11.2: Naming & Unit Standardization
- [Animation Schema](../schemas/animation.schema.json)
- [Direction Schema](../schemas/direction.schema.json)
- [Scene Schema](../schemas/scene.schema.json)
