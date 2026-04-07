# 演出ガイド - 연출 가이드

generate-video 스킬의 시각 연출 시스템의使い分けとベストプラクティスを定義します。

---

## 개요

연출 시스템は以下の 4要素로 구성됩니다:

| 요소 | 역할 | 제어 내용 |
|------|------|---------|
| **transition** | 씬切り替え | 페이드, 슬라이드, 줌, 컷 |
| **emphasis** | 요소 강조 | 3단계 강조 + 효과음 |
| **background** | 배경 디자인 | 5종의 배경 스타일 |
| **timing** | 타이밍 조정 | 대기 시간, 음성 오프셋 |

---

## Transition（트랜지션）

### 4종의 트랜지션

| Type | 용도 | 시각 효과 | 권장 duration |
|------|------|---------|--------------|
| **fade** | 범용的な切り替え | 부드러운 페이드인/아웃 | 500ms (15f) |
| **slideIn** | 다음 화제への移行 | 방향 지정 슬라이드（left/right/top/bottom）| 400ms (12f) |
| **zoom** | 상세への注目誘導 | 줌인/아웃 | 600ms (18f) |
| **cut** | 即座の切り替え | 컷（순간）| 0ms |

###使い分けガイドライン

#### fade（페이드）
- **권장 씬**: 범용, 섹션 시작, 차분한切り替え
- **효과**: 시각적으로 부드럽고, 주의가分散しすぎない
- **예**:
  - 인트로 → 메인 설명
  - 기능 설명 → 다음 기능 설명
  - CTA 전의 차분함

```json
{
  "transition": {
    "type": "fade",
    "duration_ms": 500,
    "easing": "easeInOut"
  }
}
```

#### slideIn（슬라이드인）
- **권장 씬**: 화제 전환, 비교 표시, 단계 진행
- **효과**: 역동적, 다음 내용への期待感
- **direction**:
  - `right`: 전진감（다음 단계）
  - `left`: 과거 참조（Before/After の Before）
  - `top`: 중요한情報の登場
  - `bottom`: 보충情報の追加

```json
{
  "transition": {
    "type": "slideIn",
    "duration_ms": 400,
    "direction": "right",
    "easing": "easeOut"
  }
}
```

#### zoom（줌）
- **권장 씬**: 상세 표시, 강조, 충격적인 정보
- **효과**: 주의 유도, 임팩트
- **예**:
  - 중요한 수치의 표시
  - 문제의 핵심 제시
  - 차별화 포인트 강조

```json
{
  "transition": {
    "type": "zoom",
    "duration_ms": 600,
    "easing": "easeInOut"
  }
}
```

#### cut（컷）
- **권장 씬**: 데모 조작, 고속 전개, 긴장감
- **효과**: 순간적, 템포 업
- **예**:
  - UI 조작의 단계 간
  - 고속 데몬스트레이션
  - 리듬감 있는 기능 소개

```json
{
  "transition": {
    "type": "cut",
    "duration_ms": 0
  }
}
```

### 펀널별 권장 트랜지션

| 펀널 단계 | 권장 트랜지션 | 이유 |
|-------------|-------------------|------|
| 인지（LP/광고） | fade, zoom | 차분함, 충격 |
| 관심（Intro） | slideIn, fade | 역동적, 기대감 |
| 검토（기능 데모） | cut, slideIn | 템포, 효율 |
| 확신（아키텍처） | fade, zoom | 상세, 신뢰 |
| 지속（온보딩） | slideIn, cut | 단계 진행 |

---

## Emphasis（강조）

### 3단계 강조 레벨

| Level | 용도 | 시각 효과 | 권장 효과음 |
|-------|------|---------|-----------|
| **high** |最重要メッセージ| 큰 애니메이션, 밝은 컬러| whoosh, chime |
| **medium** |重要ポイント| 중간 정도 애니메이션, 악센트 컬러| pop |
| **low** |補足情報|控えめな強調,淡いカラー| none, ding |

###使い分けガイドライン

#### high（고 강조）
- **권장 씬**:
  - Hook（첫 번째 충격）
  - CTA（행동 유도）
  - Differentiator（차별화 포인트）
  - 놀라운 결과·수치

- **시각 효과**:
  - 텍스트 크기: 특대
  - 컬러: 선명（기본: `#00F5FF` 시안）
  - 애니메이션: scale 1.2, bounce
  - 효과음: `whoosh` 또는 `chime`

- **예**:
  - "3배 빨라진다" → high emphasis
  - "지금 무료로试用" → high emphasis

```json
{
  "emphasis": {
    "level": "high",
    "text": ["3배 빨라진다"],
    "sound": "whoosh",
    "color": "#00F5FF",
    "position": "center"
  }
}
```

#### medium（中強調）
- **권장 씬**:
  - 기능 설명의 요점
  - 워크플로우의 단계
  - 문제 제시（Problem）
  - 해결책（Solution）

- **시각 효과**:
  - 텍스트 크기: 대
  - 컬러: 악센트（기본: `#FFC700` 골드）
  - 애니메이션: scale 1.1, fade-in
  - 효과음: `pop`

- **예**:
  - "단계 1: 설정" → medium emphasis
  - "이런 문제는 없습니까?" → medium emphasis

```json
{
  "emphasis": {
    "level": "medium",
    "text": ["단계 1: 설정"],
    "sound": "pop",
    "color": "#FFC700",
    "position": "top"
  }
}
```

#### low（저 강조）
- **권장 씬**:
  - 보충 정보
  - 추가 기능의 가벼운 소개
  - 주석
  - 상세 정보への링크

- **시각 효과**:
  - 텍스트 크기: 보통
  - 컬러:淡い（기본: `#A8DADC` 라이트 블루）
  - 애니메이션: fade-in 만
  - 효과음: `none` 또는 `ding`

- **예**:
  - "※詳細はドキュメント参照" → low emphasis
  - "그 외 다수의 기능" → low emphasis

```json
{
  "emphasis": {
    "level": "low",
    "text": ["※詳細 はドキュメント参照"],
    "sound": "none",
    "color": "#A8DADC",
    "position": "bottom"
  }
}
```

###効果音の選び方

| Sound | 음성 특징 | 권장 용도 |
|-------|---------|---------|
| **whoosh** | 바람 소리, 다이나믹 | high emphasis, 화면 전환 |
| **chime** | 차임, 아름다운 울림 | CTA, 성공 표시 |
| **pop** | 팝, 가벼움 | medium emphasis, 버튼 표시 |
| **ding** | 작은 방울 소리 | low emphasis, 가벼운 알림 |
| **none** | 무음 | 조용한 정보, 연속 표시 |

### 펀널별 권장 강조 레벨

| 펀널 단계 | 주요 강조 | 보조 강조 |
|-------------|---------|---------|
| 인지（LP/광고） | high 多用 | medium 適度 |
| 관심（Intro） | high 1-2회 | medium 多用 |
| 검토（기능 데모） | medium 主体 | low 보충 |
| 확신（아키텍처） | medium 適度 | low 多用 |
| 지속（온보딩） | high 목표 | medium 단계 |

---

## Background（배경）

### 5종의 배경 스타일

| Type | 시각 특징 | 용도 | 컬러 예 |
|------|---------|------|---------|
| **cyberpunk** | 네온, 그리드, 미래감 | 테크系,先進性アピール| `#0a0e27` + `#00f5ff` |
| **corporate** | 세련됨, 신뢰감, プロフェッショナル| BtoB, 엔터프라이즈 | `#1a1a2e` + `#16213e` |
| **minimal** | 심플, 클린, 집중| 説明重視, ドキュメント| `#ffffff` + `#f0f0f0` |
| **gradient** | 칼라풀, 다이나믹, 친근| BtoC, 카주얼 | `#667eea` → `#764ba2` |
| **particles** | 다이나믹 파티클, 에너지틱| Hook, CTA, 충격 | `#000000` + particles |

###使い分けガイドライン

#### cyberpunk（사이버펑크）
- **권장 씬**:
  - 테크놀로지의先進性을诉求
  - 개발자向け 도구
  - AI/ML 기능 소개
  - 아키텍처 다이어그램

- **특징**:
  - 네온 그리드
  - 글리치 이펙트
  - 青·시안系 컬러

```json
{
  "background": {
    "type": "cyberpunk",
    "primaryColor": "#0a0e27",
    "secondaryColor": "#00f5ff",
    "opacity": 0.9
  }
}
```

#### corporate（코포레이트）
- **권장 씬**:
  - BtoB 프로덕트
  - 엔터프라이즈 기능
  - 보안·신뢰성诉求
  - 실적·사례紹介

- **특징**:
  - 다크 블루系
  - 클린な그라데이션
  - 차분한 분위기

```json
{
  "background": {
    "type": "corporate",
    "primaryColor": "#1a1a2e",
    "secondaryColor": "#16213e",
    "opacity": 1
  }
}
```

#### minimal（미니멀）
- **권장 씬**:
  - コンテンツに集中させたい
  - 복잡한図表·코드 の 表示
  - 온보딩
  - ドキュメント的な説明

- **특징**:
  - 白·그레이系
  - 심플
  - 視認性重視

```json
{
  "background": {
    "type": "minimal",
    "primaryColor": "#ffffff",
    "secondaryColor": "#f0f0f0",
    "opacity": 1
  }
}
```

#### gradient（그라데이션）
- **권장 씬**:
  - BtoC 프로덕트
  - 친근함訴求
  - 인트로·CTA
  - 카주얼な톤

- **특징**:
  - 칼라풀な그라데이션
  - 부드러운 인상
  - 시각적으로楽しい

```json
{
  "background": {
    "type": "gradient",
    "primaryColor": "#667eea",
    "secondaryColor": "#764ba2",
    "opacity": 0.95
  }
}
```

#### particles（파티클）
- **권장 씬**:
  - Hook（開始時の衝撃）
  - CTA（행동 유도）
  - 중요한 전환점
  - 에너지틱한 인상

- **특징**:
  - 다이나믹 파티클
  - 에너지감
  - 注意誘導

```json
{
  "background": {
    "type": "particles",
    "primaryColor": "#000000",
    "secondaryColor": "#00f5ff",
    "opacity": 0.8
  }
}
```

### 펀널별 권장 배경

| 펀널 단계 | 권장 배경 | 이유 |
|-------------|---------|------|
| 인지（LP/광고） | particles, gradient | 시각적 임팩트 |
| 관심（Intro） | gradient, cyberpunk | 친근함,先進性 |
| 검토（기능 데모） | minimal, corporate | 집중, 신뢰감 |
| 확신（아키텍처） | corporate, cyberpunk | 프로페셔널 |
| 지속（온보딩） | minimal, gradient | 심플, 친절 |

---

## Timing（타이밍）

### 타이밍 파라미터

| Parameter | 용도 | 권장값 |
|-----------|------|--------|
| **delay_before** | 씬開始前の待機 | 0-15f（0-500ms）|
| **delay_after** | 씬終了後の待機 | 0-30f（0-1000ms）|
| **audio_start_offset** | 音声開始オフセット | 30f（1000ms, 標準）|

###使い分けガイドライン

#### delay_before（開始前待機）
- **용도**:
  - トランジション後の視覚的な落ち着き
  - 前シーンの余韻
  - 注意を引く間

- **권장값**:
  - `0f`: トランジションで十分な場合
  - `5-10f`: 가벼운間
  - `15f`:しっかりした間

```json
{
  "timing": {
    "delay_before": 10
  }
}
```

#### delay_after（終了後待機）
- **용도**:
  - 音声終了後の余韻
  - CTA表示時間の確保
  - 読む時間の確保

- **권장값**:
  - `0f`: 即座に次へ
  - `15-20f`: 標準的な余韻
  - `30f`:しっかり読ませる

```json
{
  "timing": {
    "delay_after": 20
  }
}
```

#### audio_start_offset（音声開始オフセット）
- **용도**:
  - 씬表示後、音声開始までの待機
  - 視覚的に落ちてから音声

- **권장값**:
  - `30f`（1000ms）: 標準（권장）
  - `15f`（500ms）: 고속 전개
  - `45f`（1500ms）: 여유롭kus

```json
{
  "timing": {
    "audio_start_offset": 30
  }
}
```

### 음성 동기화의 중요 규칙

> **重要**: ナレーション付き動画では以下を厳守

1. **씬 길이의 계산식**:
   ```
   duration_ms = audio_start_offset + 음성 길이 + delay_after
   ```

2. **음성 길이의 사전 확인**:
   ```bash
   ffprobe -v error -show_entries format=duration \
     -of default=noprint_wrappers=1:nokey=1 audio/scene.wav
   ```

3. **트랜지션과의 조정**:
   ```
   씬 시작 = 前씬 시작 + 前씬 길이 - トランジ션 길이
   음성 시작 = 씬 시작 + audio_start_offset
   ```

4. **여백의 확보**:
   - トランジション開始前に音声が終了すること
   - 최소한 `delay_after: 20f` 를 확보

---

## 베스트 프랙티스

### 1. 펀널별演出の組み合わせ

#### 90초LP/광고ティザー（인지~관심）
```json
{
  "hook": {
    "transition": { "type": "zoom", "duration_ms": 600 },
    "emphasis": { "level": "high", "sound": "whoosh" },
    "background": { "type": "particles" },
    "timing": { "delay_before": 10, "delay_after": 20 }
  },
  "problem": {
    "transition": { "type": "slideIn", "direction": "right", "duration_ms": 400 },
    "emphasis": { "level": "medium", "sound": "pop" },
    "background": { "type": "gradient" },
    "timing": { "delay_before": 0, "delay_after": 15 }
  },
  "cta": {
    "transition": { "type": "zoom", "duration_ms": 600 },
    "emphasis": { "level": "high", "sound": "chime" },
    "background": { "type": "particles" },
    "timing": { "delay_before": 15, "delay_after": 30 }
  }
}
```

#### 3분Introデモ（관심→검토）
```json
{
  "intro": {
    "transition": { "type": "fade", "duration_ms": 500 },
    "emphasis": { "level": "high", "sound": "whoosh" },
    "background": { "type": "gradient" },
    "timing": { "delay_before": 0, "delay_after": 20 }
  },
  "demo": {
    "transition": { "type": "cut", "duration_ms": 0 },
    "emphasis": { "level": "medium", "sound": "pop" },
    "background": { "type": "minimal" },
    "timing": { "delay_before": 0, "delay_after": 10 }
  },
  "cta": {
    "transition": { "type": "fade", "duration_ms": 500 },
    "emphasis": { "level": "high", "sound": "chime" },
    "background": { "type": "gradient" },
    "timing": { "delay_before": 10, "delay_after": 30 }
  }
}
```

### 2.効果音の適切な使用

**규칙**:
- 1 영상 내에서効果음는 최대 **5-7회** 까지
- 연속 씬では効果音を控える（慣れによる効果減少）
- high emphasis에는 반드시効果음을付ける
- medium emphasis 는 선택적
- low emphasis 는基本上無音

### 3.背景の統一感

**규칙**:
- 1 영상 내에서背景タイプは **2-3종류** 까지
- セクション単位で統一（section내는 같은 배경）
- Hook/CTAのみ特別な背景（particles）を許容

### 4.トランジションのリズム

**규칙**:
- 같은 트랜지션을 3회 이상 연속 사용하지 않기
- 고속 전개（cut）と緩急（fade/zoom）を組み合わせる
- 섹션 시작은 fade 또는 zoom 권장

### 5.強調レベルの配分

**규칙（90초 영상 경우）**:
- high: 2-3회（Hook, Differentiator, CTA）
- medium: 5-8회（주요 메시지）
- low: 적절히（보충 정보）

---

## 설계 체크리스트

씬의 연출 설계時に以下を確認：

### 트랜지션
- [ ] 씬의 목적에 맞는 트랜지션を選択
- [ ] 같은 트랜지션의 연속 사용を避けている
- [ ] duration_ms 는 적절한가（fade: 500ms, slideIn: 400ms, zoom: 600ms）

### 강조
- [ ] 강조 레벨은 적절한가（high:最重要のみ）
- [ ] 효과음의 사용 횟수는 적절한가（전체 5-7회 이내）
- [ ] text 배열에強調すべきキーワードを指定

### 배경
- [ ] 펀널 단계에 맞는 배경 타입を選択
- [ ] 섹션 내에서 배경을 통일
- [ ] primaryColor, secondaryColor 는指定したか

### 타이밍
- [ ] audio_start_offset 는 30f（표준）か
- [ ] 씬 길이 = audio_start + 음성 길이 + delay_after
- [ ] 트랜지션 시작 전에 음성이 종료된다

###全体バランス
- [ ] 효과음의 사용은 5-7회 이내
- [ ] 배경 타입는 2-3종류 이내
- [ ] high emphasis 는 2-3회 이내

---

## 관련 문서

- [generator.md](./generator.md) - 병렬 생성 플로우
- [visual-effects.md](./visual-effects.md) - 시각 이펙트 라이브러리
- [schemas/direction.schema.json](../schemas/direction.schema.json) - 연출 스키마 정의
- [schemas/emphasis.schema.json](../schemas/emphasis.schema.json) - 강조 스키마 정의
- [schemas/animation.schema.json](../schemas/animation.schema.json) - 애니메이션 스키마 정의
