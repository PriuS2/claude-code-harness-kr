# Video Generation JSON Schemas - 영상 생성 JSON 스키마

영상 생성 워크플로우용 JSON Schema 정의입니다. 이 스키마는 시나리오, 씬 및 완전한 영상 스크립트의 구조를 정의합니다.

## 스키마 파일

| 스키마 | 목적 | Version |
|--------|---------|---------|
| **scenario.schema.json** | 섹션이 있는 상위 수준 시나리오 구조 | 1.0.0 |
| **scene.schema.json** | 콘텐츠 및 방향이 포함된 개별 씬 정의 | 1.0.0 |
| **video-script.schema.json** | 메타데이터 및 설정이 포함된 완전한 영상 스크립트 | 1.0.0 |

## 스키마 개요

```
Scenario (고수준 구조)
    │
    ├── Section 1 (인트로)
    │   ├── Scene 1.1
    │   └── Scene 1.2
    │
    ├── Section 2 (데모)
    │   ├── Scene 2.1
    │   ├── Scene 2.2
    │   └── Scene 2.3
    │
    └── Section 3 (CTA)
        └── Scene 3.1

Video Script = Metadata + Scenes + Output Settings
```

## 사용법

### 1. 기본 검증 (의존성 없음)

```bash
node validate-schemas-basic.js
```

외부 의존성 없이 기본 JSON 및 구조 검증을 수행합니다.

### 2. ajv를 사용한 전체 검증

```bash
# Install dependencies first
npm install ajv ajv-formats

# Run full validation
node validate-schemas.js
```

### 3. 프로그램 방식 사용

```javascript
const Ajv = require('ajv');
const addFormats = require('ajv-formats');
const fs = require('fs');

// Initialize ajv
const ajv = new Ajv({ strict: false });
addFormats(ajv);

// Load schemas
const sceneSchema = JSON.parse(fs.readFileSync('scene.schema.json'));
const videoScriptSchema = JSON.parse(fs.readFileSync('video-script.schema.json'));

// Add schemas
ajv.addSchema(sceneSchema);
ajv.addSchema(videoScriptSchema);

// Validate data
const validate = ajv.compile(videoScriptSchema);
const valid = validate(myVideoScriptData);

if (!valid) {
  console.error(validate.errors);
}
```

## 스키마 상세

### scenario.schema.json

영상 시나리오의 상위 수준 구조를 정의합니다.

**주요 필드**:
- `title`: 시나리오 제목
- `description`: 목적 및 콘텐츠 개요
- `sections[]`: 섹션의 순서 있는 리스트
  - `id`: 고유 섹션 식별자
  - `title`: 섹션 이름
  - `description`: 섹션 목적
  - `order`: 표시 순서 (0 인덱스)
  - `duration_estimate_ms`: 예상 지속 시간
- `metadata`: 생성 메타데이터
  - `version`: 스키마 버전
  - `generated_at`: ISO 8601 타임스탬프
  - `video_type`: 타입 열거형 (lp-teaser, intro-demo, etc.)
  - `target_funnel`: 마케팅 펀널 단계

**예시**: [examples/scenario-example.json](examples/scenario-example.json) 참조

### scene.schema.json

콘텐츠, 시각적 방향 및 에셋이 포함된 개별 영상 씬을 정의합니다.

**주요 필드**:
- `scene_id`: 고유 씬 식별자
- `section_id`: 부모 섹션 참조
- `order`: 섹션 내 순서
- `type`: 씬 타입 열거형 (intro, ui-demo, cta, etc.)
- `content`: 씬 콘텐츠
  - `text`: 기본 텍스트
  - `image`: 이미지 에셋 경로
  - `duration_ms`: 씬 지속 시간
  - `url`: Playwright 캡처용
  - `actions[]`: UI 자동화 액션
  - `mermaid`: 다이어그램 정의
  - `code`: 하이라이트가 있는 코드 스니펫
- `direction`: 시각 효과
  - `transition`: 인/아웃 전환
  - `emphasis`: 시각적 강조 효과
  - `background`: 배경 구성
  - `camera`: 3D 카메라 이동
- `assets[]`: 씬 에셋
  - `type`: 에셋 타입 (image, video, audio, font)
  - `source`: 경로 또는 URL
  - `generated`: AI 생성 플래그
- `audio`: 오디오 구성
  - `narration`: 보이스 오버
  - `sfx[]`: 효과음

**예시**: [examples/scene-example.json](examples/scene-example.json) 참조

### video-script.schema.json

모든 씬, 메타데이터 및 출력 설정이 포함된 완전한 영상 스크립트입니다.

**주요 필드**:
- `metadata`: 영상 메타데이터
  - `title`: 영상 제목
  - `version`: 스크립트 버전
  - `created_at`: 생성 타임스탬프
  - `video_type`: 타입 열거형
  - `scenario_id`: 소스 시나리오 참조
- `scenes[]`: scene.schema.json를 참조하는 씬 객체 배열
- `total_duration_ms`: 총 영상 지속 시간
- `output_settings`: 렌더링 구성
  - `width`, `height`: 해상도
  - `fps`: 프레임 레이트 (24, 30, 60)
  - `codec`: 비디오 코덱 (h264, h265, vp9, av1)
  - `format`: 출력 포맷 (mp4, webm, mov, gif)
  - `quality`: 품질 프리셋
  - `preset`: 해상도 프리셋 (1080p, 4k, etc.)
- `audio_settings`: 글로벌 오디오
  - `bgm`: 배경 음악 구성
  - `master_volume`: 마스터 볼륨 컨트롤
- `branding`: 브랜드 구성
  - `logo`: 로고 경로
  - `colors`: 브랜드 컬러
  - `fonts`: 폰트 구성
- `transitions`: 글로벌 전환 설정

**예시**: [examples/video-script-example.json](examples/video-script-example.json) 참조

## 씬 타입

| 타입 | 설명 | 사용 상황 |
|------|-------------|----------|
| `intro` | 오프닝 타이틀/로고 | 첫 번째 씬, 브랜드 소개 |
| `ui-demo` | Playwright가 있는 UI 워크스루 | 기능 시연 |
| `architecture` | 시스템 아키텍처 다이어그램 | 기술적 설명 |
| `code-highlight` | 하이라이트가 있는 코드 스니펫 | 개발자 중심 콘텐츠 |
| `changelog` | 릴리스 노트 표시 | 버전 업데이트 |
| `cta` | 행동 유도 | 최종 씬, 전환 |
| `feature-highlight` | 특정 기능 초점 | 기능 마케팅 |
| `problem-promise` | 문제 + 해결 주장 | 가치 제안 |
| `workflow` | 멀티 스텝 워크플로우 시연 | 프로세스 설명 |
| `objection` | 일반적인 이의 처리 | 이의 처리 |
| `custom` | 커스텀 씬 타입 | 유연한 사용 |

## 영상 타입 (video_type)

| 타입 | 지속 시간 | 펀널 단계 | 목적 |
|------|----------|--------------|----------|
| `lp-teaser` | 30-90s | Awareness | 랜딩 페이지, 소셜 광고 |
| `intro-demo` | 2-3min | Interest | 제품 소개 |
| `release-notes` | 1-3min | Consideration | 기능 업데이트 |
| `architecture` | 5-30min | Decision | 기술 심층 분석 |
| `onboarding` | 30s-3min | Retention | 사용자 온보딩 |
| `custom` | 가변 | Any | 커스텀 목적 |

## 오디오 동기화 규칙

내레이션을 사용할 때, 다음 타이밍 규칙을 따르세요:

| 규칙 | 값 | 이유 |
|------|-------|--------|
| **Audio start** | Scene start + 1000ms | 1초 틈 |
| **Scene length** | 1000ms + audio length + 500ms | 전환을 위한 패딩 |
| **Transition** | 450-500ms overlap | 부드러운 크로스페이드 |
| **Scene start calc** | Previous scene start + duration - 450ms | 오버랩 처리 |

**항상 먼저 오디오 지속 시간을 확인하세요**:
```bash
ffprobe -v error -show_entries format=duration \
  -of default=noprint_wrappers=1:nokey=1 audio.mp3
```

## 검증

### 스키마별 필수 필드

**scenario.schema.json**:
- ✅ title, description, sections, metadata

**scene.schema.json**:
- ✅ scene_id, section_id, order, type, content
- ✅ content.duration_ms

**video-script.schema.json**:
- ✅ metadata, scenes, total_duration_ms, output_settings
- ✅ output_settings: width, height, fps
- ✅ metadata: title, version, created_at

### 일반적인 검증 에러

| 에러 | 원인 | 해결책 |
|-------|-------|--------|
| `Missing required property` | 필수 필드가 제공되지 않음 | 누락된 필드 추가 |
| `Invalid enum value` | 잘못된 type/format 값 | 허용된 열거형 값 사용 |
| `Pattern mismatch` | ID 형식不正确 | 소문자와 하이픈 사용 |
| `Invalid date-time` | 타임스탬프 형식 잘못됨 | ISO 8601 형식 사용 |
| `Invalid $ref` | 스키마 참조 끊어짐 | scene.schema.json이 로드되었는지確認 |

## 예시

모든 예시 파일은 `examples/` 디렉토리에 있습니다:

1. **scenario-example.json** - 90초 티저 시나리오
2. **scene-example.json** - 이펙트가 있는 인트로 씬
3. **video-script-example.json** - 완전한 영상 스크립트

## 통합

이 스키마는 다음에서 사용됩니다:

1. **Planner** (planner.md) - 시나리오 및 씬 구조 생성
2. **Generator** (generator.md) - video-script.json를 읽고 영상 렌더링
3. **Validation** - 생성된 데이터가 예상 구조를 준수하는지 확인

## 버전 이력

| 버전 | 날짜 | 변경사항 |
|---------|------|---------|
| 1.0.0 | 2026-02-02 | 핵심 스키마와 함께 초기 출시 |

## 참조

- [JSON Schema Draft-07](https://json-schema.org/draft-07/json-schema-release-notes.html)
- [ajv Documentation](https://ajv.js.org/)
- [Best Practices Guide](../references/best-practices.md)
- [Planner Reference](../references/planner.md)
- [Generator Reference](../references/generator.md)
