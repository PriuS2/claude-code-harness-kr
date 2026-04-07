# Remotion Visual Components - Remotion 시각 컴포넌트

Phase 5: 시각 컴포넌트 구현

## 컴포넌트

### 1. EmphasisBox

3단계 강조 표시 컴포넌트.

**Features**:
- 3 levels: `high`, `medium`, `low`
- 5 styles: `bold`, `glitch`, `underline`, `highlight`, `glow`
- Pulse animation support
- Glow effects
- Sound effect integration
- Customizable colors and fonts

**Usage**:
```tsx
import { EmphasisBox } from './components';

<EmphasisBox
  level="high"
  text="Important Message"
  color="#00F5FF"
  enablePulse={true}
  enableGlow={true}
  sound="pop"
  startFrame={30}
  durationFrames={90}
/>
```

**Props**:
- `level`: `'high' | 'medium' | 'low'` - Emphasis intensity
- `text`: `string` - Text to display
- `color`: `string` - Primary color (hex)
- `sound`: `'none' | 'pop' | 'whoosh' | 'chime' | 'ding'` - Sound effect
- `style`: `'bold' | 'glitch' | 'underline' | 'highlight' | 'glow'`
- `enablePulse`: `boolean` - Enable pulse animation
- `enableGlow`: `boolean` - Enable glow effect
- `startFrame`: `number` - Start frame (relative to scene)
- `durationFrames`: `number` - Duration in frames

---

### 2. TransitionWrapper

4종류의 트랜지션 이펙트로 콘텐츠를 래프.

**Features**:
- 4 types: `fade`, `slideIn`, `zoom`, `cut`
- Remotion `interpolate` and `spring` support
- 4 easing functions: `linear`, `easeIn`, `easeOut`, `easeInOut`
- Customizable slide direction
- Spring physics option
- Preset configurations

**Usage**:
```tsx
import { TransitionWrapper, TransitionPresets } from './components';

<TransitionWrapper
  type="slideIn"
  duration={20}
  direction="right"
  easing="easeInOut"
>
  <YourContent />
</TransitionWrapper>

// Or use presets
<TransitionWrapper {...TransitionPresets.fadeIn(15)}>
  <YourContent />
</TransitionWrapper>
```

**Props**:
- `type`: `'fade' | 'slideIn' | 'zoom' | 'cut'` - Transition type
- `duration`: `number` - Duration in frames (default: 15)
- `direction`: `'left' | 'right' | 'top' | 'bottom'` - Slide direction
- `easing`: `'linear' | 'easeIn' | 'easeOut' | 'easeInOut'`
- `useSpring`: `boolean` - Use spring physics instead of interpolation
- `springConfig`: `{ damping, stiffness, mass }` - Spring parameters
- `delay`: `number` - Delay before transition starts (frames)

**Presets**:
- `TransitionPresets.fadeIn(duration)`
- `TransitionPresets.fadeOut(duration)`
- `TransitionPresets.slideFromRight(duration)`
- `TransitionPresets.slideFromLeft(duration)`
- `TransitionPresets.zoomIn(duration)`
- `TransitionPresets.springBounce()`

---

### 3. ProgressIndicator

섹션 위치 표시 컴포넌트.

**Features**:
- 3 styles: `bar`, `dots`, `minimal`
- 4 positions: `top`, `bottom`, `left`, `right`
- Auto-detection of current section
- Animated transitions
- Optional section labels
- 3 sizes: `small`, `medium`, `large`

**Usage**:
```tsx
import { ProgressIndicator, createSections } from './components';

const sections = createSections([
  { id: 'intro', name: 'Intro', startFrame: 0, durationFrames: 90 },
  { id: 'demo', name: 'Demo', startFrame: 90, durationFrames: 180 },
  { id: 'cta', name: 'CTA', startFrame: 270, durationFrames: 60 },
]);

<ProgressIndicator
  sections={sections}
  position="bottom"
  style="dots"
  showLabels={true}
  activeColor="#00F5FF"
  size="medium"
/>
```

**Props**:
- `sections`: `Section[]` - Array of sections
- `currentIndex`: `number` - Current section (auto-detected if omitted)
- `position`: `'top' | 'bottom' | 'left' | 'right'`
- `style`: `'bar' | 'dots' | 'minimal'`
- `showLabels`: `boolean` - Show section names
- `activeColor`: `string` - Color for active section
- `inactiveColor`: `string` - Color for inactive sections
- `size`: `'small' | 'medium' | 'large'`
- `animated`: `boolean` - Animate transitions

**Section Type**:
```typescript
interface Section {
  id: string;
  name: string;
  startFrame: number;
  endFrame: number;
  color?: string;
}
```

---

### 4. BackgroundLayer

5종류의 애니메이션 배경 레이어.

**Features**:
- 5 types: `neutral`, `highlight`, `dramatic`, `tech`, `warm`
- Static image or video support
- Animated gradients
- Type-specific effects:
  - `tech`: Animated grid overlay
  - `dramatic`: Vignette effect
  - `highlight`: Floating particles
  - `warm`: Pulsing radial gradient
- Blur and overlay support
- Customizable colors

**Usage**:
```tsx
import { BackgroundLayer, getRecommendedBackground } from './components';

// Generated gradient background
<BackgroundLayer
  type="tech"
  animated={true}
  opacity={0.8}
/>

// Image background
<BackgroundLayer
  type="neutral"
  src="/path/to/background.jpg"
  blur={5}
  overlayColor="rgba(0,0,0,0.3)"
/>

// Video background
<BackgroundLayer
  type="highlight"
  src="/path/to/background.mp4"
  isVideo={true}
  opacity={0.6}
/>

// Auto-select based on scene type
const bgType = getRecommendedBackground('intro'); // Returns 'highlight'
```

**Props**:
- `type`: `'neutral' | 'highlight' | 'dramatic' | 'tech' | 'warm'`
- `src`: `string` - Path to image/video (optional)
- `isVideo`: `boolean` - Is the source a video?
- `primaryColor`: `string` - Primary gradient color (hex)
- `secondaryColor`: `string` - Secondary gradient color (hex)
- `opacity`: `number` - Background opacity (0-1)
- `animated`: `boolean` - Enable animations
- `blur`: `number` - Blur intensity (pixels)
- `overlayColor`: `string` - Overlay tint color
- `overlayOpacity`: `number` - Overlay opacity (0-1)

**Background Types**:
| Type | Primary | Secondary | Use Case |
|------|---------|-----------|----------|
| `neutral` | Dark gray | Light gray | General content, demos |
| `highlight` | Cyan | Magenta | Intros, CTAs, highlights |
| `dramatic` | Black | Red | Hooks, problem statements |
| `tech` | Dark blue | Navy | Architecture, technical content |
| `warm` | Orange | Yellow | Conclusions, warm CTAs |

---

## 스키마와의 통합

모든 컴포넌트는 Phase 4 스키마와 함께 사용하도록 설계되었습니다:

- `EmphasisBox` ← `emphasis.schema.json`
- `TransitionWrapper` ← `animation.schema.json`
- `BackgroundLayer` ← `direction.schema.json` (background section)

**예시 통합**:
```typescript
import { EmphasisBox, TransitionWrapper, BackgroundLayer } from './components';
import { EmphasisSchema, AnimationSchema, DirectionSchema } from '../schemas';

// Load direction data from JSON
const direction = DirectionSchema.parse(directionData);

// Use in Remotion composition
<>
  <BackgroundLayer
    type={direction.background.type}
    primaryColor={direction.background.primaryColor}
    opacity={direction.background.opacity}
  />

  <TransitionWrapper
    type={direction.transition.type}
    duration={direction.transition.duration_frames}
    easing={direction.transition.easing}
  >
    <EmphasisBox
      level={direction.emphasis.level}
      text={direction.emphasis.text[0]}
      sound={direction.emphasis.sound}
      color={direction.emphasis.color}
    />
  </TransitionWrapper>
</>
```

---

## 애니메이션 성능

모든 컴포넌트는 최적의 성능을 위해 Remotion의 네이티브 `interpolate` 및 `spring` 함수를 사용합니다:

- **CPU 효율적**: 과도한 React 리렌더링 없음
- **예측 가능**: 결정적 애니메이션
- **부드러움**: 1920x1080에서 60fps

**모범 사례**:
1. 자연스러운 모션（바운스, 탄성）에는 `spring` 사용
2. 선형/이징된 모션에는 `interpolate` 사용
3. 애니메이션 섹션에서 복잡한 CSS 필터 피하기
4. 레이아웃 변경보다 CSS 변형 선호

---

## 테스트

각 컴포넌트는 Remotion Studio에서 개별적으로 테스트할 수 있습니다:

```bash
cd remotion
npm run dev
```

`src/Root.tsx` 에서 테스트 컴포지션을 생성합니다:

```tsx
import { Composition } from 'remotion';
import { EmphasisBox, TransitionWrapper, ProgressIndicator, BackgroundLayer } from './components';

export const RemotionRoot = () => (
  <>
    <Composition
      id="EmphasisTest"
      component={EmphasisBox}
      durationInFrames={180}
      fps={30}
      width={1920}
      height={1080}
      defaultProps={{
        level: 'high',
        text: 'Test Emphasis',
        enablePulse: true,
      }}
    />
    {/* More test compositions... */}
  </>
);
```

---

## 다음 단계

### Phase 6: 이미지 생성 패턴

시각 컴포넌트가 구현되었으므로, AI 생성 이미지와 통합합니다:

1. **Task 6.1**: `visual-patterns.schema.json` 정의
2. **Task 6.2**: 이미지 프롬프트 템플릿 생성
3. **Task 6.3**: comparison/concept/flow 패턴 구현
4. **Task 6.4**: Nano Banana Pro 와 통합

### 통합 포인트

- AI 생성 배경과 함께 `BackgroundLayer` 사용
- AI 생성 다이어그램에 `EmphasisBox` 오버레이
- `TransitionWrapper` 로 AI 이미지 애니메이션
- `ProgressIndicator` 로 생성 진행률 표시

---

## 라이선스

Claude Code Harness - generate-video skill의 일부.
MIT 라이선스.
