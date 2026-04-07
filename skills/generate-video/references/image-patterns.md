# Image Patterns Reference - 이미지 패턴 참조

영상 씬에 최적화된 이미지 패턴의使用ガイド입니다。

---

## 개요

영상 씬에 최적화된 이미지 패턴을 정의합니다. 각 패턴은 특정 목적에 최적화되어 있으며, AI 이미지 생성 프로MPT 템플릿과 연계합니다.

### 패턴 목록

| 패턴 | 용도 | 최적 씬 | 프로MPT 템플릿 |
|---------|------|-----------|---------------------|
| **comparison** | Before/After, 좋은 예/나쁜 예의 대조 | 문제 제시, 개선 효과 제시 | `templates/image-prompts/comparison.txt` |
| **concept** | 추상 개념, 계층 구조, 관계성의 시각화 | 아키텍처 해설, 콘셉트 설명 | `templates/image-prompts/concept.txt` |
| **flow** | 절차, 프로세스, 워크플로우의 도식화 | 데모 절차, 처리 플로우 | `templates/image-prompts/flow.txt` |
| **highlight** | 중요 포인트, 메시지의 강조 | Hook, CTA, 결론 | `templates/image-prompts/highlight.txt` |

---

## 1. Comparison 패턴 {#comparison}

### 목적

Before/After, 좋은 예/나쁜 예など、2つの状態や選択肢を視覚的に対比させる。

### 사용 상황

| 씬 | 예 |
|--------|-----|
| **문제 제시** |既存ツールの煩雑さ vs 本製品のシンプルさ |
| **개선 효과** | 導入前（手動、遅い）vs 導入後（自動、速い） |
| **기능 비교** |従来の方法 vs 新機能 |
| **릴리스 노트** |旧バージョン vs 新バージョン |

### 시각 구성

```
┌──────────────────────────────────────────┐
│                                          │
│  [나쁜 예/Before]  🠖  [좋은 예/After]      │
│                                          │
│  ❌ 문제점1         ✅ 개선점1           │
│  ❌ 문제점2         ✅ 개선점2           │
│  ❌ 문제점3         ✅ 개선점3           │
│                                          │
└──────────────────────────────────────────┘
```

### JSON 예

```json
{
  "type": "comparison",
  "topic": "タスク管理の改善",
  "style": "modern",
  "colorScheme": {
    "primary": "#3B82F6",
    "secondary": "#10B981",
    "background": "#1F2937"
  },
  "comparison": {
    "leftSide": {
      "label": "Before",
      "items": [
        "手動でスプレッドシート管理",
        "更新漏れが頻発",
        "ステータス把握に30分"
      ],
      "icon": "x",
      "sentiment": "negative"
    },
    "rightSide": {
      "label": "After",
      "items": [
        "自動でダッシュボード更新",
        "リアルタイム同期",
        "ステータス把握が一目瞭然"
      ],
      "icon": "check",
      "sentiment": "positive"
    },
    "divider": "arrow"
  }
}
```

### 프로MPT 생성의 포인트

- **왼쪽（Before/나쁜 예）**: 赤系、警告アイコン、散らかった印象
- **오른쪽（After/좋은 예）**: 緑系、チェックアイコン、整理された印象
- **구분**: 명확한 화살표 또는 "VS" 로 시각적 분리
- **텍스트**: 짧고 구체적（각 항목 20자 이내 권장）

### 피해야 할 패턴

| ❌ 피하기 | ✅ 권장 |
|----------|---------|
| 장문의 나열 | 짧은 키워드 |
| 추상적 설명 | 구체적 수치·결과 |
| 중간적 평가 | 명확한 대조 |
| 양쪽에 같은 아이콘 | 다른 감정의 아이콘 |

---

## 2. Concept 패턴 {#concept}

### 목적

추상적인 개념, 계층 구조, 요소 간의 관계성을 시각적으로 표현합니다.

### 사용 상황

| 씬 | 예 |
|--------|-----|
| **아키텍처 해설** | 시스템 구성도, 레이어 구조 |
| **콘셉트 설명** | 철학, 설계 사상, 가치 제공의 도식화 |
| **관계성** | 컴포넌트 간의 의존 관계 |
| **프로세스 전체상** | 에코시스템, 워크플로우 전체 |

### 시각 구성（계층 예）

```
        ┌───────────┐
        │  최상위   │
        └─────┬─────┘
              │
     ┌────────┴────────┐
     │                 │
┌────▼────┐       ┌────▼────┐
│ 레벨 1 │       │ 레벨 1 │
└─────────┘       └────┬────┘
                       │
                  ┌────▼────┐
                  │ 레벨 2 │
                  └─────────┘
```

### JSON 예

```json
{
  "type": "concept",
  "topic": "마이크로서비스 아키텍처",
  "style": "technical",
  "colorScheme": {
    "primary": "#6366F1",
    "secondary": "#8B5CF6",
    "background": "#0F172A"
  },
  "concept": {
    "elements": [
      {
        "id": "api-gateway",
        "label": "API Gateway",
        "description": "全リクエストの入り口",
        "level": 0,
        "icon": "cloud",
        "emphasis": "high"
      },
      {
        "id": "auth-service",
        "label": "인증 서비스",
        "level": 1,
        "parentId": "api-gateway",
        "icon": "server",
        "emphasis": "medium"
      },
      {
        "id": "data-service",
        "label": "데이터 서비스",
        "level": 1,
        "parentId": "api-gateway",
        "icon": "database",
        "emphasis": "medium"
      }
    ],
    "relationships": [
      {
        "from": "api-gateway",
        "to": "auth-service",
        "label": "인증 확인",
        "type": "flow"
      },
      {
        "from": "api-gateway",
        "to": "data-service",
        "label": "데이터取得",
        "type": "flow"
      }
    ],
    "layout": "hierarchy"
  }
}
```

### 레이아웃 타입

| 레이아웃 | 용도 | 시각 이미지 |
|-----------|------|------------|
| **hierarchy** | 계층 구조（조직도, 의존 관계）| 위에서 아래로 트리 |
| **radial** | 중심에서 방사（에코시스템）| 중앙에 주요 요소, 주변에 관련 요소 |
| **grid** | 並列配置（카테고리 분류）| 매트릭스 배치 |
| **flow** | 처리 플로우（파이프라인）| 왼쪽에서 오른쪽으로 흐름 |
| **circular** | 순환 프로세스（라이프사이클）| 원형 |

### 프로MPT 생성의 포인트

- **요소 수**: 2-10개（너무 많으면 보기 어려움）
- **계층**: 최대 3-4 레벨까지
- **아이콘**: 요소의 성질을 直感적으로 표현
- **관계성**: 화살표의 굵기나 색으로 중요도를 표현

### 피해야 할 패턴

| ❌ 피하기 | ✅ 권장 |
|----------|---------|
| 10개 이상의 요소 | 7개 이내로絞り |
| 복잡한 관계선 | 주요 관계만 |
| 긴 설명문 | 짧은 레이블 + 아이콘 |
| 같은 모습의 요소 | 강조도로 차별화 |

---

## 3. Flow 패턴 {#flow}

### 목적

절차, 프로세스, 워크플로우를 시계열 또는 단계 순으로 시각화합니다.

### 사용 상황

| 씬 | 예 |
|--------|-----|
| **데모 절차** |セットアップから実行までのステップ|
| **사용자 플로우** | ログイン → 操作 → 完了の流れ |
| **처리 플로우** | 데이터 파이프라인, CI/CD 플로우 |
| **온보딩** | 初回利用の導線 |

### 시각 구성（수평 예）

```
[1. 시작] ──▶ [2. 입력] ──▶ [3. 처리] ──▶ [4. 완료]
   ⏱2분         ⏱1분         ⏱3초         即座
```

### JSON 예

```json
{
  "type": "flow",
  "topic": "영상 生成フロー",
  "style": "modern",
  "colorScheme": {
    "primary": "#F59E0B",
    "secondary": "#EF4444",
    "background": "#111827"
  },
  "flow": {
    "steps": [
      {
        "id": "analyze",
        "label": "코드베이스 분석",
        "description": "프로젝트 구조를 자동 감지",
        "order": 1,
        "type": "start",
        "icon": "circle",
        "duration": "10초"
      },
      {
        "id": "plan",
        "label": "시나리오 生成",
        "description": "최적의 영상 구성을 제안",
        "order": 2,
        "type": "process",
        "icon": "square",
        "duration": "20초"
      },
      {
        "id": "generate",
        "label": "並列 生成",
        "description": "각 씬을 동시作成",
        "order": 3,
        "type": "parallel",
        "icon": "rounded",
        "duration": "2분"
      },
      {
        "id": "render",
        "label": "렌더링",
        "description": "최종 영상を出력",
        "order": 4,
        "type": "end",
        "icon": "hexagon",
        "duration": "30초"
      }
    ],
    "direction": "horizontal",
    "arrowStyle": "solid",
    "showNumbers": true
  }
}
```

### 단계 타입

| 타입 | 용도 | 시각 표현 |
|--------|------|---------|
| **start** | 플로우의 시작점 |丸アイコン、緑色|
| **process** |通常の処理ステップ|四角、青色|
| **decision** |条件分岐|ひし形、黄色|
| **parallel** |並列処理|複数アイコン、紫色|
| **subprocess** |サブフロー|角丸四角|
| **end** |플로우의 종료점|二重丸、赤色|

### 프로MPT 생성의 포인트

- **방향**: 横（horizontal）が読みやすい（英語圈向け）
- **단계 수**: 2-10단계（너무 많으면複雑）
- **소요 시간**: 各ステップに時間を表示すると実用的
- **번호**: 順序を明示（showNumbers: true）

### 피해야 할 패턴

| ❌ 피하기 | ✅ 권장 |
|----------|---------|
| 10단계 이상 | 7단계 이내로 통합 |
| 복잡한 분기 | 선형 플로우로 단순화 |
| 긴 단계명 | 動詞 + 名詞で簡潔に |
| 불명확한 순서 | order フィールドで明示 |

---

## 4. Highlight 패턴 {#highlight}

### 목적

단일 메시지, 키워드, 수치를 강조表示합니다。

### 사용 상황

| 씬 | 예 |
|--------|-----|
| **Hook（冒頭）** | "もう手動で消耗していませんか？" |
| **CTA（행동 유도）** | "今すぐ試す" |
| **결론** | "3倍速く、10倍簡単" |
| **중요 메트릭스** | "95%の時間削減" |

### 시각 구성

```
┌────────────────────────────────────────┐
│                                        │
│                                        │
│          ⚡ 3배 빠르고, 10배 간편 ⚡       │
│                                        │
│        自動化で変わる開発体験          │
│                                        │
└────────────────────────────────────────┘
```

### JSON 예

```json
{
  "type": "highlight",
  "topic": "製品価値の強調",
  "style": "gradient",
  "colorScheme": {
    "primary": "#EC4899",
    "accent": "#8B5CF6",
    "background": "#18181B"
  },
  "highlight": {
    "mainText": "95%の時間削減",
    "subText": "手動作業から解放される開発チーム",
    "icon": "rocket",
    "position": "center",
    "effect": "glow",
    "fontSize": "xlarge",
    "emphasis": "high"
  }
}
```

### 이펙트 타입

| 이펙트 | 용도 | 시각 표현 |
|-----------|------|---------|
| **glow** | 신적인 강조（CTA, 결론）| 발광 이펙트 |
| **shadow** | 차분한 강조（Hook）| 드롭 쉐도우 |
| **gradient** | 모던한 인상 | 그라데이션 배경 |
| **outline** | 샤프한 인상 | 윤곽선만 |
| **none** | 미니멀 | 장식 없음 |

### 아이콘과 감정

| 아이콘 | 감정·의미 | 사용 상황 |
|---------|-----------|---------|
| **star** | 우수, 품질 | 기능 소개, 평가 |
| **check** | 완료, 성공 | 도입 효과, 결과 |
| **alert** | 주의 환기 | 문제 제시, 경고 |
| **trophy** | 달성, 승리 | 성과, 실적 |
| **rocket** | 고속, 혁신 | 성능, 신기능 |
| **fire** | 인기, 화제 | 트렌드, 주목 |
| **bolt** | 즉시, 파워 | 속도, 효율 |

### 프로MPT 생성의 포인트

- **짧음이 생명**: 메인 텍스트는 10자 이내가理想
- **수치**: 구체적 수치는説得力が高い（"95%", "3배"）
- **대비**: "빠르고, 간편" 처럼 2개의 가치를 並べる
- **감정**: 아이콘 + 이펙트로 감정을 증폭

### 피해야 할 패턴

| ❌ 피하기 | ✅ 권장 |
|----------|---------|
| 장문（20자 이상）| 짧은 캐치프레이즈 |
| 복수의 주장 | 1개에 집중 |
| 수수한 디자인 | 이펙트로目立たせる|
| 작은 폰트 | xlarge 권장 |

---

## 패턴 선택 가이드

### 씬 타입별 권장 패턴

| 씬 타입 | 제1 권장 | 제2 권장 | 용도 |
|------------|---------|---------|------|
| **Hook** | highlight | comparison | 강한 첫인상 |
| **Problem** | comparison | concept | 현황의 과제를 명확히 |
| **Solution** | concept | flow | 해결책의 메커니즘 |
| **Demo** | flow | comparison | 절차의可視化|
| **Differentiator** | comparison | concept | 차별화 포인트 |
| **CTA** | highlight | - | 행동 유도 |

### 퍼널별 사용 빈도

| 패턴 | 인지·관심 | 검토 | 확신 | 지속 |
|---------|-----------|------|------|------|
| **comparison** | ★★★ | ★★★ | ★★☆ | ★☆☆ |
| **concept** | ★☆☆ | ★★★ | ★★★ | ★★☆ |
| **flow** | ★★☆ | ★★★ | ★★☆ | ★★★ |
| **highlight** | ★★★ | ★★☆ | ★★★ | ★☆☆ |

### 복수 패턴의 조합

**90초 티저（LP/광고向け）の例**:

| 초수 | 씬 | 패턴 | 내용 |
|------|--------|---------|------|
| 0-5초 | Hook | **highlight** | "もう手動で消耗していませんか？" |
| 5-15초 | Problem | **comparison** | Before（手動）vs After（自動） |
| 15-55초 | Solution | **flow** | セットアップ → 実行 → 完了の3ステップ |
| 55-70秒 | Proof | **concept** | 아키텍처의 견고성 |
| 70-90秒 | CTA | **highlight** | "今すぐ無料で始める" |

---

## 구현시의주의 사항

### 1. JSON Schema バリデーション

- **必須**: `type`, `topic` フィールドは必須
- **oneOf**: パターンに応じた専用フィールドが必須（例: type="comparison" なら comparison フィールド必須）
- **バリデーション**: `scripts/validate-visual-pattern.js` で検証

### 2. 프로MPT 템플릿과의 연계

- **テンプレート**: `templates/image-prompts/{type}.txt` を使用
- **プレースホルダー**: `{{topic}}`, `{{items}}`, `{{style}}` 等を JSON 値で置換
- **生成**: `references/image-generator.md` が実際の 生成を担当

### 3. 이미지 품질 체크

- **自動判定**: `references/image-quality-check.md` で品質評価
- **再試行**: 不合格の場合、最大3回まで再生成
- **결정성**: seed 値を保存し、再現性を確保

### 4. 에셋 관리

- **出力先**: `out/video-{id}/assets/generated/`
- **マニフェスト**: `assets.manifest.schema.json` に記録
- **해시**: SHA-256 で改ざん検出

---

## 관련 문서

- [visual-patterns.schema.json](../schemas/visual-patterns.schema.json) - JSON Schema 정의
- [image-generator.md](./image-generator.md) - AI 이미지 生成実装
- [image-quality-check.md](./image-quality-check.md) - 品質判定 로직
- [templates/image-prompts/](../templates/image-prompts/) - 프로MPT 템플릿
- [best-practices.md](./best-practices.md) - 영상 전체의 베스트 프랙티스

---

**작성일**: 2026-02-02
**대상Phase**: Phase 6 - 이미지 生成パターン
**유지보수**: 스키마 변경時に更新
