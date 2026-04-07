# Slide Quality Check - 슬라이드 이미지 품질 판정

Nano Banana Pro로 생성한 슬라이드 이미지를 Claude가 시각 판정하여 품질을保証합니다。

---

## 개요

`/generate-slide` 의 Step 4에서 실행되는品質判定 로직입니다。
각 패턴（Minimalist, Infographic, Hero Visual）의 2枚を評価し、ベスト1枚を選出します。

---

## 판정 플로우

```
패턴별로 2枚の画像を受け取り
    |
    +--[Step 1] 各画像を個別評価
    |   +-- スライド固有の5基準でスコアリング
    |   +-- 各基準を1-5点で評価
    |   +-- 総合スコアを算出（重み付き平均）
    |
    +--[Step 2] パターン内比較
    |   +-- 両方スコア3以上 → 高い方を採用
    |   +-- 1枚のみスコア3以上 → その1枚を採用
    |   +-- 両方スコア2以下 → 再生成へ
    |
    +--[Step 3] 再生成判定
        +-- 両方 NG → プロンプト改善して再生成（最大3回）
        +-- リトライ上限 → ユーザーに報告
```

---

## 슬라이드 고유의 판정 기준

### 5 기준과 중량

| 기준 | 중량 | 설명 | 평가 포인트 |
|------|------|------|-------------|
| **정보 전달력** | 高（x3） | 프로젝트의 특징이 1매로 전해지는가 | 무슨 프로젝트인지一眼で理解できる、가치 제안이 명확 |
| **레이아웃 밸런스** | 高（x3） | 시각적 완성도, 여백 사용 | 요소 배치가 정리되어 있음, 시선 유도이 자연스러움 |
| **텍스트 가독성** | 中（x2） | AI 생성 텍스트가 읽히는가 | 문자화けなし、フォントサイズが適切、대비 충분 |
| **프로페셔널감** | 中（x2） | 비즈니스 용도에耐えるクオリティか | 칩함 없음、색使い가 세련됨、统一感がある |
| **브랜드 통합성** | 低（x1） | 지정된 색·톤과의 일치 | 지정 톤에沿っている、컬러が 크게 벗어나いない |

### 종합 스코어 산출

```
종합 스코어 = (정보 전달력 x 3 + 레이아웃 x 3 + 텍스트 가독성 x 2 + 프로페셔널감 x 2 + 브랜드 통합성 x 1) / 11
```

### 스코어 정의

| 스코어 | 판정 | 설명 |
|--------|------|------|
| 5 | Excellent | 완벽, 즉시 채용 |
| 4 | Good | 양호, 채용 가능 |
| 3 | Acceptable | 허용 범위, 다른 것이 없으면 채용 |
| 2 | Poor | 문제 있음, 재생성 권장 |
| 1 | Unacceptable | 사용 불가, 반드시 재생성 |

### 채용 임곣값

```
채용 임곣값 = 3（Acceptable 이상）
```

---

## 패턴별 추가 체크

### Minimalist

| 체크 항목 | 합격 기준 |
|-------------|---------|
| 여백 활용 | 충분한 여백이 있어 답답하지 않음 |
| 타이포그래피 | 폰트가 읽기 쉽고 계층이 명확 |
| 심플함 | 요소가 최소한으로絞り込まれ 있음 |

### Infographic

| 체크 항목 | 합격 기준 |
|-------------|---------|
| 정보의 구조화 | 시각적으로 섹션나뉘어 있음 |
| 아이콘/도 활용 | 텍스트만이 아닌 시각 요소가 있음 |
| 데이터 시각화 | 수치나 특징이 도식화되어 있음 |

### Hero Visual

| 체크 항목 | 합격 기준 |
|-------------|---------|
| 비주얼 임팩트 | 눈을 끄는 대담한 비주얼 |
| 캐치프레이즈 | 메시지가 명확하고 읽기 쉬움 |
| 정서적诉求 | 프로젝트의 가치를 감각적으로 전달하고 있음 |

---

## Claude 에 의한 판정 프롬프트

### 이미지 평가 프롬프트

````text
다음 슬라이드 이미지를 평가하세요.

## 평가 대상
- 패턴: {pattern_name}（Minimalist / Infographic / Hero Visual）
- 프로젝트: {project_name}
- 개요: {project_description}
- 기대 톤: {tone}

## 평가 기준（각 1-5점）
1. 정보 전달력（중량: 高）— 프로젝트의 특징이 1매로 전해지는가
2. 레이아웃 밸런스（중량: 高）— 시각적 완성도, 여백 사용
3. 텍스트 가독성（중량: 中）— 글자가 읽히는가, 문자化けがないか
4. 프로페셔널감（중량: 中）— 비즈니스 용도에耐えるクオリティか
5. 브랜드 통합성（중량: 低）— 지정된 색·톤과의 일치

## 출력 형식
```json
{
  "scores": {
    "information_delivery": 1-5,
    "layout_balance": 1-5,
    "text_readability": 1-5,
    "professionalism": 1-5,
    "brand_consistency": 1-5
  },
  "total_score": 1-5,
  "verdict": "OK" | "NG",
  "strengths": ["좋은 점1", "좋은 점2"],
  "issues": ["문제점1", "문제점2"],
  "improvement_suggestions": ["개선안1", "개선안2"]
}
```
````

### 2枚比較 프롬프트

````text
다음 2枚の 슬라이드画像を比較し、より適切な方を選択してください。

## 평가 대상
- 패턴: {pattern_name}
- 프로젝트: {project_name}

## 이미지 1 의 평가
{image_1_evaluation}

## 이미지 2 의 평가
{image_2_evaluation}

## 선택 기준의 우선순위
1. 정보 전달력（최우선）
2. 프로페셔널감
3. 레이아웃 밸런스

## 출력 형식
```json
{
  "selected": "1" | "2",
  "reason": "선택 이유",
  "comparison_notes": "비교의 상세"
}
```
````

---

## 패턴 내 선발 로직

### 기본 규칙

```
1. 両方スコア3以上:
   → 스코어가 높은 쪽을 채용
   → 同スコア → 우선 기준에서 판정:
      1. 정보 전달력이 높은 쪽
      2. 프로페셔널감이 높은 쪽
      3. 후보 1을 기본으로 채용

2. 1枚のみスコア3以上:
   → 스코어 3 이상인 쪽을 채용

3. 両方スコア2以下:
   → 재생성（프롬프트 개선）
   → 리트리 상한（3回）に達したら:
      → 스코어가 높은 쪽을「仮採用」として使用자에게 보고
      → 使用자가続行 or スキップを選択
```

### 리트리 제어

```
max_retries = 3（패턴별）
```

| 리트리 | 개선 전략 |
|---------|---------|
| 1회 |初期プロンプトで生成|
| 2회 |品質指摘を反映。구체적修飾語を追加|
| 3회 |스타일을大幅変更、構図指示をより具体的に|

---

## 판정 결과의 구조

### 개별 평가 결과

```json
{
  "image_id": "minimalist_1",
  "pattern": "minimalist",
  "scores": {
    "information_delivery": 4,
    "layout_balance": 5,
    "text_readability": 3,
    "professionalism": 4,
    "brand_consistency": 4
  },
  "total_score": 4.1,
  "verdict": "OK",
  "strengths": [
    "여백 활용이 뛰어나다",
    "프로젝트명이一眼で読める"
  ],
  "issues": [
    "기능 리스트의 텍스트가 다소 작다"
  ],
  "improvement_suggestions": [
    "텍스트 크기를 크게 하거나 기능 수를 줄이라"
  ]
}
```

### 패턴 선발 결과

```json
{
  "pattern": "minimalist",
  "candidates": [
    {"id": "minimalist_1", "total_score": 4.1, "verdict": "OK"},
    {"id": "minimalist_2", "total_score": 3.5, "verdict": "OK"}
  ],
  "selected": "minimalist_1",
  "reason": "레이아웃 밸런스와 정보 전달력이 뛰어나다",
  "output_path": "out/slides/selected/minimalist.png"
}
```

---

## 품질 보고서 생성

Step 5에서 `out/slides/quality-report.md` 를 생성:

```markdown
# Slide Quality Report

## 생성 정보
- 프로젝트: {project_name}
- 생성 일시: {datetime}
- 애스팩트 비율: {aspect_ratio}
- 톤: {tone}

## 결과 요약

| 패턴 | 후보1 | 후보2 | 채용 | 스코어 | 리트리 |
|---------|-------|-------|------|--------|---------|
| Minimalist | {score}/5 | {score}/5 | 후보{n} | {score}/5 | 0회 |
| Infographic | {score}/5 | {score}/5 | 후보{n} | {score}/5 | 0회 |
| Hero Visual | {score}/5 | {score}/5 | 후보{n} | {score}/5 | 0회 |

## 상세 평가

### Minimalist

#### 후보 1 (minimalist_1.png)
- 정보 전달력: {score}/5
- 레이아웃 밸런스: {score}/5
- 텍스트 가독성: {score}/5
- 프로페셔널감: {score}/5
- 브랜드 통합성: {score}/5
- **종합: {score}/5 — {verdict}**
- 강점: {strengths}
- 과제: {issues}

#### 후보 2 (minimalist_2.png)
...

### Infographic
...

### Hero Visual
...

## 출력 파일

| 파일 | 설명 |
|---------|---|
| `out/slides/selected/minimalist.png` | Minimalist 패턴 최상 |
| `out/slides/selected/infographic.png` | Infographic 패턴 최상 |
| `out/slides/selected/hero.png` | Hero Visual 패턴 최상 |
```

---

## 임곣값 조정

### 사용자 요청 대응

```
「もっと厳しく」→ 採用 임곣값을 4로 인상
「とりあえずで」→ 採用 임곣값을 2로 인하
「この画像でいい」→ 品質判定をスキップして採用
```

---

## 관련 문서

- [slide-generator.md](./slide-generator.md) — 이미지 생성 로직
- [generate-video/references/image-quality-check.md](../../generate-video/references/image-quality-check.md) — 영상用品질 판정（구조의 참고）
