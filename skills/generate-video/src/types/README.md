# Component Type Definitions - 컴포넌트 타입 정의

Remotion 컴포넌트용 TypeScript 타입 정의로, JSON 스키마와 동기화됩니다.

## 개요

이 디렉토리는 `schemas/` 의 JSON 스키마 정의와 일치하는 TypeScript 타입 정의를 포함합니다. 이러한 타입은 Remotion 컴포넌트 사용時にタイプ セーフティを確保します。

## 파일

- **`components.ts`** - Remotion 컴포넌트의 핵심 타입 정의
- **`index.ts`** - 편의을 위한 배럴 export

## 타입 정의

### TransitionConfig

`TransitionWrapper` 컴포넌트의 구성.

```typescript
interface TransitionConfig {
  type: 'fade' | 'slide_in' | 'zoom' | 'cut';
  duration_ms: number;
  easing?: 'linear' | 'easeIn' | 'easeOut' | 'easeInOut' | ...;
  spring?: { damping?: number; stiffness?: number; mass?: number; };
  delay_ms?: number;
  direction?: 'left' | 'right' | 'top' | 'bottom';
  opacity_range?: [number, number];
  scale_range?: [number, number];
  slide_distance?: number;
}
```

**Maps to**: `schemas/animation.schema.json`

**Example**:
```typescript
const transition: TransitionConfig = {
  type: 'fade',
  duration_ms: 500,
  easing: 'easeInOut'
};
```

### EmphasisConfig

`EmphasisBox` 컴포넌트의 구성.

```typescript
interface EmphasisConfig {
  level: 'subtle' | 'medium' | 'strong';
  effect: 'glow' | 'pulse' | 'outline' | 'none';
  duration_ms?: number;
  text?: string;
  color?: string;
  glow_intensity?: number;
  enable_pulse?: boolean;
  font_size?: number;
  background_color?: string;
  enable_background?: boolean;
}
```

**Maps to**: `schemas/emphasis.schema.json`

**Example**:
```typescript
const emphasis: EmphasisConfig = {
  level: 'strong',
  effect: 'glow',
  text: 'Important!',
  color: '#00F5FF',
  glow_intensity: 30
};
```

### BackgroundConfig

`BackgroundLayer` 컴포넌트의 구성.

```typescript
interface BackgroundConfig {
  type: 'solid' | 'gradient' | 'image' | 'video' | 'particles';
  value: string;
  opacity?: number;
  blur?: number;
  animated?: boolean;
  overlay_color?: string;
  overlay_opacity?: number;
  secondary_color?: string;
  gradient_angle?: number;
}
```

**Maps to**: `schemas/visual-patterns.schema.json` background section

**Example**:
```typescript
const background: BackgroundConfig = {
  type: 'gradient',
  value: '#1a1a1a',
  secondary_color: '#2a2a2a',
  gradient_angle: 135,
  animated: true
};
```

## Type Guards

런타임에 객체가 예상 타입을 준수하는지 확인하는 함수:

```typescript
// 객체가 유효한 TransitionConfig인지 확인
if (isTransitionConfig(obj)) {
  // TypeScript는 여기서 obj가 TransitionConfig임을 압니다
}

// 객체가 유효한 EmphasisConfig인지 확인
if (isEmphasisConfig(obj)) {
  // TypeScript는 여기서 obj가 EmphasisConfig임을 압니다
}

// 객체가 유효한 BackgroundConfig인지 확인
if (isBackgroundConfig(obj)) {
  // TypeScript는 여기서 obj가 BackgroundConfig임을 압니다
}
```

## Remotion 컴포넌트에서의 사용

### Example: TransitionConfig 사용

```tsx
import { TransitionConfig } from '../src/types/components';
import { msToFrames } from '../src/utils/converters';

interface MyComponentProps {
  transition: TransitionConfig;
}

export const MyComponent: React.FC<MyComponentProps> = ({ transition }) => {
  const durationFrames = msToFrames(transition.duration_ms, 30);

  return (
    <TransitionWrapper
      type={transition.type}
      duration={durationFrames}
      easing={transition.easing}
    >
      {/* Your content */}
    </TransitionWrapper>
  );
};
```

## 스키마 동기화

이러한 타입은 JSON 스키마와手動으로 동기화됩니다. 스키마가 변경될 때:

1. 해당 타입 정의를 업데이트합니다
2. 필요한 경우 타입 가드를 업데이트합니다
3. 호환성을確保하기 위해 테스트를 실행합니다

### 자동 스키마 생성

JSON 스키마에서 자동 타입 생성을 원하면, 다음을 사용합니다:

```bash
npm run generate:schemas
```

이것은 `src/schemas/` 에 런타임 검증에 사용할 수 있는 Zod 스키마를 생성합니다.

## 관련 파일

- **스키마**: `schemas/*.schema.json` - JSON Schema 정의
- **Zod 스키마**: `src/schemas/*.ts` - 자동 생성된 Zod 스키마
- **컴포넌트**: `remotion/components/*.tsx` - 이러한 유틸리티를 사용하는 Remotion 컴포넌트
- **유틸리티**: `src/utils/converters.ts` - 프레임/ms 변환 유틸리티
