# Utility Functions - 유틸리티 함수

Remotion 시간 단위로 작업하기 위한 변환 유틸리티.

## 개요

Remotion은 **프레임**을 기본 시간 단위로 사용하고, JSON 스키마는 **밀리초**를 사용합니다. 이러한 유틸리티는 시간 단위 간 양방향 변환을 제공합니다.

## 파일

- **`converters.ts`** - TypeScript 버전 (Remotion 컴포넌트용)
- **`converters.js`** - JavaScript 버전 (Node.js 스크립트 및 테스트용)
- **`index.ts`** - 배럴 export

## 핵심 변환

### msToFrames(ms, fps)

밀리초를 프레임으로 변환합니다.

```javascript
msToFrames(1000, 30)  // => 30 (1 second at 30fps)
msToFrames(500, 60)   // => 30 (0.5 seconds at 60fps)
msToFrames(33.33, 30) // => 1 (approximately 1 frame)
```

**Parameters**:
- `ms` (number) - 밀리초 단위 지속 시간
- `fps` (number, optional) - 초당 프레임 (기본값: 30)

**Returns**: 프레임 단위 지속 시간 (가장 가까운 정수로 반올림)

**Throws**: ms < 0 또는 fps <= 0인 경우 에러

---

### framesToMs(frames, fps)

프레임을 밀리초로 변환합니다.

```javascript
framesToMs(30, 30) // => 1000 (30 frames at 30fps = 1 second)
framesToMs(15, 30) // => 500 (0.5 seconds)
framesToMs(1, 30)  // => 33.33
```

**Parameters**:
- `frames` (number) - 프레임 단위 지속 시간
- `fps` (number, optional) - 초당 프레임 (기본값: 30)

**Returns**: 밀리초 단위 지속 시간 (소수점 2자리까지 반올림)

**Throws**: frames < 0 또는 fps <= 0인 경우 에러

---

### secondsToFrames(seconds, fps)

초를 프레임으로 변환합니다.

```javascript
secondsToFrames(1, 30)    // => 30
secondsToFrames(0.5, 60)  // => 30
```

---

### framesToSeconds(frames, fps)

프레임을 초로 변환합니다.

```javascript
framesToSeconds(30, 30) // => 1.00
framesToSeconds(15, 30) // => 0.50
```

---

### msToSeconds(ms)

밀리초를 초로 변환합니다.

```javascript
msToSeconds(1000) // => 1.00
msToSeconds(500)  // => 0.50
```

---

### secondsToMs(seconds)

초를 밀리초로 변환합니다.

```javascript
secondsToMs(1)   // => 1000
secondsToMs(0.5) // => 500
```

---

## 배치 변환

### batchMsToFrames(msValues, fps)

여러 밀리초 값을 프레임으로 변환합니다.

```javascript
batchMsToFrames([1000, 2000, 3000], 30)
// => [30, 60, 90]
```

---

### batchFramesToMs(frameValues, fps)

여러 프레임 값을 밀리초로 변환합니다.

```javascript
batchFramesToMs([30, 60, 90], 30)
// => [1000, 2000, 3000]
```

---

## 타임스탬프 유틸리티

### getFrameAtTimestamp(timestampMs, fps)

특정 타임스탬프에서 프레임 번호를 계산합니다.

```javascript
getFrameAtTimestamp(1500, 30) // => 45
```

---

### getTimestampAtFrame(frameNumber, fps)

특정 프레임에서 타임스탬프를 계산합니다.

```javascript
getTimestampAtFrame(45, 30) // => 1500
```

---

## 검증

### isValidFps(fps)

FPS 값을 검증합니다.

```javascript
isValidFps(30)       // => true
isValidFps(0)        // => false
isValidFps(-30)      // => false
isValidFps(Infinity) // => false
```

---

## 상수

### DEFAULT_FPS

기본 초당 프레임 (30).

```javascript
const { DEFAULT_FPS } = require('./converters');
console.log(DEFAULT_FPS); // => 30
```

---

### FPS_PRESETS

일반 FPS 프리셋.

```javascript
const { FPS_PRESETS } = require('./converters');

FPS_PRESETS.CINEMA   // => 24 (cinematic video)
FPS_PRESETS.STANDARD // => 30 (standard video)
FPS_PRESETS.HD       // => 60 (high frame rate)
FPS_PRESETS.SMOOTH   // => 120 (very smooth)
```

---

## 사용 예

### Example 1: 스키마 Duration을 Remotion 프레임으로 변환

```typescript
import { msToFrames } from '../src/utils/converters';
import { useVideoConfig } from 'remotion';

export const MyScene: React.FC<{ durationMs: number }> = ({ durationMs }) => {
  const { fps } = useVideoConfig();
  const durationFrames = msToFrames(durationMs, fps);

  return <Sequence durationInFrames={durationFrames}>{/* ... */}</Sequence>;
};
```

### Example 2: 애니메이션 타이밍 일괄 변환

```javascript
const { batchMsToFrames } = require('./src/utils/converters');

const animationTimings = {
  intro: 1000,
  main: 3000,
  outro: 1000,
};

const fps = 30;
const [introFrames, mainFrames, outroFrames] = batchMsToFrames(
  [animationTimings.intro, animationTimings.main, animationTimings.outro],
  fps
);

console.log({ introFrames, mainFrames, outroFrames });
// => { introFrames: 30, mainFrames: 90, outroFrames: 30 }
```

### Example 3: 다양한 FPS로 작업

```javascript
const { msToFrames, FPS_PRESETS } = require('./src/utils/converters');

// Cinema (24fps)
const cinema = msToFrames(1000, FPS_PRESETS.CINEMA); // => 24

// Standard (30fps)
const standard = msToFrames(1000, FPS_PRESETS.STANDARD); // => 30

// HD (60fps)
const hd = msToFrames(1000, FPS_PRESETS.HD); // => 60
```

### Example 4: 라운드트립 변환

```javascript
const { msToFrames, framesToMs } = require('./src/utils/converters');

const originalMs = 1500;
const frames = msToFrames(originalMs, 30); // => 45
const backToMs = framesToMs(frames, 30);   // => 1500

console.log(originalMs === backToMs); // => true
```

---

## 에러 처리

모든 변환 함수는 입력을 검증하고 설명적인 에러를 throw합니다:

```javascript
msToFrames(-100, 30)
// Error: msToFrames: milliseconds must be non-negative, got -100

framesToMs(30, 0)
// Error: framesToMs: fps must be positive, got 0

secondsToFrames(-1, 30)
// Error: secondsToFrames: seconds must be non-negative, got -1
```

---

## 테스트

모든 변환을 검증하기 위해 테스트를 실행합니다:

```bash
npm test -- tests/converters.test.js
```

테스트 커버리지:
- ✅ 기본 변환
- ✅ 기본 파라미터
- ✅ 배치 연산
- ✅ 엣지 케이스 (0, 매우 작은 값, 매우 큰 값)
- ✅ 에러 처리
- ✅ 라운드트립 변환
- ✅ 다양한 FPS 값

---

## 성능

모든 변환 함수는 성능에 최적화된 경량입니다:

- **산술 연산만** (과도한 계산 없음)
- **정밀도를 위한 소수점 2자리까지 반올림**
- **외부 의존성 없음**

Remotion의 렌더링 루프에서 사용해도 성능 문제 없이 적합합니다.

---

## 관련 파일

- **타입 정의**: `src/types/components.ts` - 컴포넌트 타입 정의
- **테스트**: `tests/converters.test.js` - 포괄적 테스트 스위트
- **Remotion 컴포넌트**: `remotion/components/*.tsx` - 이러한 유틸리티를 사용하는 컴포넌트
