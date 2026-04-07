# Visual Effects Library - 시각 효과 라이브러리

영상 에 임팩트를与える視覚効果のテンプレート集です。

---

## 컬러 팔레트

### Cyberpunk / Neon（권장）

임팩트 있는 테크系 영상向け。

```tsx
const colors = {
  background: "#0A0A0F",  // 딥 다크
  primary: "#00F5FF",     // 시안
  secondary: "#FF00FF",   // 마젠타
  accent: "#7B2FFF",      // 퍼플
  text: "#FFFFFF",
  glow: "rgba(0, 245, 255, 0.5)",
};
```

### Corporate / Professional

비즈니스向け落ち着いたトーン。

```tsx
const colors = {
  background: "#FFFFFF",
  primary: "#FF6B35",     // 오렌지
  secondary: "#004E89",   // 네이비
  accent: "#2EC4B6",      // 티얼
  text: "#1A1A2E",
};
```

---

## 효과 컴포넌트

### GlitchText - 글리치 텍스트

RGB 분리 + 랜덤 오프셋으로 사이버펑크풍 텍스트.

```tsx
import { useCurrentFrame, interpolate, random } from "remotion";

const GlitchText: React.FC<{
  text: string;
  fontSize?: number;
  startFrame?: number;
}> = ({ text, fontSize = 72, startFrame = 0 }) => {
  const frame = useCurrentFrame();
  const adjustedFrame = frame - startFrame;

  // グリッチ強度（最初の20フレームで減衰）
  const glitchIntensity = adjustedFrame < 20
    ? interpolate(adjustedFrame, [0, 20], [20, 0])
    : 0;
  const opacity = interpolate(adjustedFrame, [0, 15], [0, 1], {
    extrapolateRight: "clamp",
  });

  // ランダムオフセット
  const offsetX = glitchIntensity > 0
    ? (random(`x-${frame}`) - 0.5) * glitchIntensity
    : 0;
  const offsetY = glitchIntensity > 0
    ? (random(`y-${frame}`) - 0.5) * glitchIntensity * 0.5
    : 0;

  return (
    <div style={{ position: "relative", opacity }}>
      {/* Red channel (마젠타) */}
      <div
        style={{
          position: "absolute",
          fontSize,
          fontWeight: 800,
          color: "#FF00FF",
          transform: `translate(${offsetX - 3}px, ${offsetY}px)`,
          mixBlendMode: "screen",
          opacity: glitchIntensity > 0 ? 0.8 : 0,
        }}
      >
        {text}
      </div>
      {/* Blue channel (시안) */}
      <div
        style={{
          position: "absolute",
          fontSize,
          fontWeight: 800,
          color: "#00F5FF",
          transform: `translate(${offsetX + 3}px, ${offsetY}px)`,
          mixBlendMode: "screen",
          opacity: glitchIntensity > 0 ? 0.8 : 0,
        }}
      >
        {text}
      </div>
      {/* Main text */}
      <div
        style={{
          fontSize,
          fontWeight: 800,
          color: "#FFFFFF",
          textShadow: "0 0 20px rgba(0, 245, 255, 0.5)",
          transform: `translate(${offsetX}px, ${offsetY}px)`,
        }}
      >
        {text}
      </div>
    </div>
  );
};
```

**사용 예**:
```tsx
<GlitchText text="혁신적인 기능" fontSize={64} startFrame={0} />
```

---

### Particles - 파티클 시스템

부유·수렴하는 파티클 애니메이션.

```tsx
import { useMemo } from "react";
import { useCurrentFrame, useVideoConfig, interpolate, random } from "remotion";

const Particles: React.FC<{
  count?: number;
  converge?: boolean;      // 중앙에 수렴하는지
  convergeFrame?: number;  // 수렴 완료 프레임
}> = ({ count = 50, converge = false, convergeFrame = 100 }) => {
  const frame = useCurrentFrame();
  const { width, height } = useVideoConfig();

  // useMemo でパーティクル初期位置を固定（重要！）
  const particles = useMemo(() => {
    return Array.from({ length: count }, (_, i) => ({
      id: i,
      startX: random(`px-${i}`) * width,
      startY: random(`py-${i}`) * height,
      speed: 0.5 + random(`speed-${i}`) * 2,
      size: 2 + random(`size-${i}`) * 4,
      hue: random(`hue-${i}`) > 0.5 ? "#00F5FF" : "#FF00FF",
    }));
  }, [count, width, height]);

  return (
    <div style={{ position: "absolute", inset: 0, overflow: "hidden" }}>
      {particles.map((p) => {
        const progress = converge
          ? interpolate(frame, [0, convergeFrame], [0, 1], {
              extrapolateRight: "clamp",
            })
          : 0;

        const targetX = width / 2;
        const targetY = height / 2;

        // 収束 or 浮遊
        const x = converge
          ? interpolate(progress, [0, 1], [p.startX, targetX])
          : p.startX + Math.sin(frame * 0.02 * p.speed + p.id) * 30;
        const y = converge
          ? interpolate(progress, [0, 1], [p.startY, targetY])
          : p.startY + ((frame * p.speed * 0.5) % height);

        const opacity = converge
          ? interpolate(progress, [0, 0.8, 1], [0.8, 0.8, 0])
          : 0.6 + Math.sin(frame * 0.1 + p.id) * 0.4;

        return (
          <div
            key={p.id}
            style={{
              position: "absolute",
              left: x,
              top: y % height,
              width: p.size,
              height: p.size,
              borderRadius: "50%",
              backgroundColor: p.hue,
              boxShadow: `0 0 ${p.size * 2}px ${p.hue}`,
              opacity,
            }}
          />
        );
      })}
    </div>
  );
};
```

**사용 예**:
```tsx
{/* 부유 파티클 */}
<Particles count={80} />

{/* 수렴 파티클（CTA 씬용）*/}
<Particles count={100} converge convergeFrame={150} />
```

---

### ScanLine - 스캔라인

화면을 지나는解析波エフェクト。

```tsx
const ScanLine: React.FC<{ speed?: number }> = ({ speed = 1 }) => {
  const frame = useCurrentFrame();
  const { height } = useVideoConfig();
  const y = (frame * speed * 5) % (height + 100);

  return (
    <div
      style={{
        position: "absolute",
        left: 0,
        right: 0,
        top: y - 50,
        height: 100,
        background: `linear-gradient(180deg, transparent, #00F5FF40, transparent)`,
        boxShadow: "0 0 60px #00F5FF",
      }}
    />
  );
};
```

**사용 예**:
```tsx
{/* 解析中の演出 */}
{frame < 60 && <ScanLine speed={3} />}
```

---

### ProgressBar - 진행 바

병렬 처리의 진행 상황을 시각화.

```tsx
const ProgressBar: React.FC<{ progress: number; label: string }> = ({
  progress,
  label,
}) => {
  return (
    <div style={{ width: 400, marginBottom: 16 }}>
      <div
        style={{
          fontSize: 18,
          color: "#FFFFFF",
          marginBottom: 8,
          fontFamily: "monospace",
        }}
      >
        {label}
      </div>
      <div
        style={{
          height: 8,
          background: "rgba(255,255,255,0.1)",
          borderRadius: 4,
          overflow: "hidden",
        }}
      >
        <div
          style={{
            width: `${progress * 100}%`,
            height: "100%",
            background: "linear-gradient(90deg, #00F5FF, #FF00FF)",
            boxShadow: "0 0 20px #00F5FF",
            borderRadius: 4,
          }}
        />
      </div>
    </div>
  );
};
```

**사용 예**:
```tsx
const agents = [
  { name: "Agent 1: Intro", progress: Math.min(1, frame / 150) },
  { name: "Agent 2: Demo", progress: Math.min(1, (frame - 30) / 180) },
  { name: "Agent 3: CTA", progress: Math.min(1, (frame - 60) / 120) },
];

{agents.map((agent) => (
  <ProgressBar key={agent.name} progress={agent.progress} label={agent.name} />
))}
```

---

### 3D Parallax - 패럴랙스 효과

깊이感のある 3D 카드 표시.

```tsx
const ParallaxCard: React.FC<{
  children: React.ReactNode;
  delay: number;
  color: string;
}> = ({ children, delay, color }) => {
  const frame = useCurrentFrame();

  const opacity = interpolate(frame, [delay, delay + 30], [0, 1], {
    extrapolateLeft: "clamp",
    extrapolateRight: "clamp",
  });
  const z = interpolate(frame, [delay, delay + 30], [-100, 0], {
    extrapolateLeft: "clamp",
    extrapolateRight: "clamp",
  });
  const rotateY = interpolate(frame, [delay, delay + 30], [45, 0], {
    extrapolateLeft: "clamp",
    extrapolateRight: "clamp",
  });

  return (
    <div
      style={{
        width: 280,
        height: 160,
        background: `linear-gradient(135deg, ${color}30, ${color}10)`,
        border: `2px solid ${color}`,
        borderRadius: 16,
        display: "flex",
        justifyContent: "center",
        alignItems: "center",
        opacity,
        transform: `translateZ(${z}px) rotateY(${rotateY}deg)`,
        boxShadow: `0 0 40px ${color}40`,
      }}
    >
      {children}
    </div>
  );
};
```

**사용 예**:
```tsx
<div style={{ display: "flex", gap: 40, perspective: 1000 }}>
  <ParallaxCard delay={30} color="#00F5FF">LP/광고</ParallaxCard>
  <ParallaxCard delay={70} color="#FF00FF">Intro 데모</ParallaxCard>
  <ParallaxCard delay={110} color="#7B2FFF">릴리스 노트</ParallaxCard>
</div>
```

---

## 조합 예시

### 임팩트 중시 Hook 씬

```tsx
const HookScene: React.FC = () => {
  const frame = useCurrentFrame();

  return (
    <AbsoluteFill style={{ background: "#0A0A0F" }}>
      <Particles count={80} />
      <div
        style={{
          position: "absolute",
          inset: 0,
          display: "flex",
          flexDirection: "column",
          justifyContent: "center",
          alignItems: "center",
        }}
      >
        <GlitchText text="코드에서 영상으로" fontSize={64} startFrame={0} />
        <div style={{ height: 20 }} />
        <GlitchText text="자동 생성되는 시대へ" fontSize={64} startFrame={15} />
      </div>
      {frame < 30 && <ScanLine speed={3} />}
    </AbsoluteFill>
  );
};
```

### CTA 씬（파티클 수렴）

```tsx
const CTAScene: React.FC = () => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();

  const logoScale = spring({ frame: frame - 60, fps, config: { damping: 200 } });
  const pulse = Math.sin(frame / 10) * 0.03 + 1;

  return (
    <AbsoluteFill style={{ background: "#0A0A0F" }}>
      <Particles count={100} converge convergeFrame={150} />
      <div
        style={{
          position: "absolute",
          inset: 0,
          display: "flex",
          flexDirection: "column",
          justifyContent: "center",
          alignItems: "center",
        }}
      >
        <div
          style={{
            opacity: interpolate(frame, [60, 90], [0, 1], {
              extrapolateRight: "clamp",
            }),
            transform: `scale(${Math.max(0, logoScale)})`,
          }}
        >
          <Img src={staticFile("logo.png")} style={{ width: 120, height: 120 }} />
        </div>
        <div
          style={{
            marginTop: 40,
            padding: "16px 48px",
            background: "linear-gradient(90deg, #00F5FF, #FF00FF)",
            borderRadius: 12,
            fontSize: 24,
            fontWeight: 700,
            color: "#0A0A0F",
            transform: `scale(${pulse})`,
            boxShadow: "0 0 40px rgba(0, 245, 255, 0.6)",
          }}
        >
          지금 바로 시도
        </div>
      </div>
    </AbsoluteFill>
  );
};
```

---

## 주의 사항

| 항목 | 규칙 |
|------|--------|
| `random()` | 引数でシード指定必須（フレーム毎に同じ値） |
| `useMemo` | パーティクル等の大量オブジェクトは必ずメモ化 |
| `interpolate` | `extrapolateRight: "clamp"` で値の暴走防止 |
| `spring` | `config: { damping: 200 }` で滑らかに |
| CSS animations | 使用禁止、Remotion の `useCurrentFrame()` を使う |

---

## References

- [generator.md](generator.md) - 並列 생성 엔진
- [best-practices.md](best-practices.md) - 영상 제작 베스트 프랙티스
