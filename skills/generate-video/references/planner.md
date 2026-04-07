# Video Planner - 시나리오 플래너

분석 결과에서 씬 구성을 자동 제안하고, 사용자와 확인·조정을 수행합니다.

---

## 개요

`/generate-video` 의 Step 2에서 실행되는 시나리오 플래너입니다.
analyzer.md 의 출력을 받아, 최적의 씬 구성을 제안합니다.

> **重要**: 씬 구성은 [best-practices.md](best-practices.md) 의 펀널별 가이드라인에 따라 설계할 것

## 입력

analyzer.md 로부터의 분석 결과:
- 프로젝트 정보（이름, description）
- 감지된 기능 리스트
- 권장 영상 타입
- 최근 변경점

---

## 펀널별 템플릿 선택

### Step 0: 목적의 확인（필수）

영상 の목을確認し、適切なテンプレートを選択する。

| 목적（펀널） | 영상 타입 | 길이目安 | 구성 핵심 |
|------------------|------------|----------|----------|----------|
| 인지~관심 | LP/광고 티저 | 30-90초 | 고통→결과→CTA |
| 관심→검토 | Intro 데모 | 2-3분 | 1 유즈케이스 완료 |
| 검토→확신 | Demo/릴리스 노트 | 2-5분 | 反論を先に潰す |
| 확신→결정 | 워크스루 | 5-30분 | 実運用+証拠 |
| 지속·활용 | 온보딩 | 30초-수분 | Aha 경험으로的最短パス |

### 90초 티저 템플릿

**용도**: LP/광고, 인지~관심 펀널

```
0:00-0:05 (150f)  → HookScene: 고통 or 원하는 결과
0:05-0:15 (300f)  → ProblemPromise: 대상 사용자와 약속
0:15-0:55 (1200f) → WorkflowDemo: 상징 워크플로우
0:55-1:10 (450f)  → Differentiator: 차별화의 근거
1:10-1:30 (600f)  → CTA: 다음 수단
```

### 3분 Intro 데모 템플릿

**용도**: 검토용, 관심→검토 펀널

```
0:00-0:10 (300f)  → Hook: 결론+고통
0:10-0:30 (600f)  → UseCase: 유즈케이스 선언
0:30-2:20 (3300f) → Demo: 실제 화면에서 완료
2:20-2:50 (900f)  → Objection: 흔한 불안감 1개 해소
2:50-3:00 (300f)  → CTA: 행동 유도
```

### 20분 워크스루 템플릿

**용도**: 결정용, 확신→결정 펀널

```
0:00-1:00   → Intro: 대상과課題
1:00-8:00   → BasicFlow: 기본 플로우
8:00-12:00  → Objections: 反論トップ2
12:00-15:00 → Security: 관리/보안
15:00-20:00 → CaseStudy+CTA: 성공 사례＋CTA
```

## 씬 템플릿

### 공통 씬

| 씬 | 권장 시간 | 내용 | 필수 |
|--------|----------|------|------|
| **인트로** | 3-5초 | 로고 + 태그라인 + 페이드인 | ✅ |
| **CTA** | 3-5초 | URL + 연락처 + 페이드아웃 | ✅ |

### 프로덕트 데모용 씬

| 씬 | 권장 시간 | 내용 |
|--------|----------|------|
| **기능 소개** | 5-10초 | 기능명 + 1줄 설명 |
| **UI 데모** | 10-30초 | Playwright 캡처 |
| **하이라이트** | 5-10초 | 주요 특징을 강조 |

### 아키텍처 해설용 씬

| 씬 | 권장 시간 | 내용 |
|--------|----------|------|
| **개요도** | 5-10초 | 전체 구성의 Mermaid 다이어그램 |
| **상세 해설** | 10-20초 | 각 컴포넌트로 줌인 |
| **데이터 플로우** | 10-15초 | 시퀀스 다이어그램 애니메이션 |

### 릴리스 노트용 씬

| 씬 | 권장 시간 | 내용 |
|--------|----------|------|
| **버전 표시** | 3-5초 | vX.Y.Z + 릴리스일 |
| **변경점 리스트** | 5-15초 | Added/Changed/Fixed 의 애니메이션 |
| **Before/After** | 10-20초 | UI 변경의 사이드바이사이드 비교 |
| **신기능 데모** | 10-30초 | 추가 기능의 UI 데모 |

---

## 시나리오 생성 로직

### Step 1: 영상 타입별 템플릿 선택

```
권장 영상 타입に基づいてベーステンプレートを選択:
    │
    ├─ LP/광고 티저（30-90초）
    │   └─ Hook → ProblemPromise → WorkflowDemo → Differentiator → CTA
    │
    ├─ Intro 데모（2-3분）
    │   └─ Hook → UseCase宣言 → 実画面Demo → Objection → CTA
    │
    ├─ 릴리스 노트（1-3분）
    │   └─ Hook → バージョン → Before/After → 新機能Demo → CTA
    │
    ├─ 아키텍처 해설（5-30분）
    │   └─ Intro → 概要図 → 詳細解説×N → データフロー → 管理/セキュリティ → CTA
    │
    └─ 온보딩（30초-수분）
        └─ Welcome → クイックウィン → 次のステップ
```

**중요한 원칙**:
- 冒頭でロゴや会社紹介を長く出さない（離脱 방지）
- CTA는 마지막だけでなく途中에도 배치
- 기능 나열ではなく「고통→해결」의 스토리

### Step 2: 감지된 기능에서 씬 생성

```python
# 의사 코드
for feature in detected_features:
    if feature.type == "auth":
        add_scene("인증 플로우 데모", duration=15, source="playwright")
    elif feature.type == "dashboard":
        add_scene("대시보드 소개", duration=20, source="playwright")
    elif feature.type == "api":
        add_scene("API 개요", duration=10, source="mermaid")
```

### Step 3: 시간 배분의 최적화

| 영상 길이 | 권장 용도 | 씬 수目安 |
|--------|----------|-------------|
| 15초 | SNS 광고 | 3-4 |
| 30초 | 쇼트 영상 | 5-6 |
| 60초 | 표준 데모 | 8-10 |
| 2-3분 | 상세 해설 | 15-20 |

---

## 사용자 확인 플로우

### 제안 표시

```markdown
🎬 시나리오 플랜

**영상 타입**: 프로덕트 데모
**총 시간**: 45초

| # | 씬 | 시간 | 내용 | 소스 |
|---|--------|------|------|--------|
| 1 | 인트로 | 5초 | MyApp - 태스크 관리를 쉽게 | 템플릿 |
| 2 | 인증 플로우 | 15초 | 로그인 화면 데모 | Playwright |
| 3 | 대시보드 | 20초 | 주요 기능 소개 | Playwright |
| 4 | CTA | 5초 | myapp.com | 템플릿 |

이 구성으로 괜찮습니까?
1. OK, 生成開始
2. 편집하고 싶음
3. 취소
```

### AskUserQuestion 구현

```
AskUserQuestion:
  question: "이 시나리오로 영상을 생성할까요?"
  header: "시나리오 확인"
  options:
    - label: "OK, 생성 시작"
      description: "이 씬 구성으로 영상을 생성합니다"
    - label: "편집하고 싶음"
      description: "씬의 추가/삭제/변경을 수행합니다"
    - label: "취소"
      description: "영상 생성을中止します"
```

### 편집 모드

사용자가「편집하고 싶음」을 선택한 경우:

```markdown
📝 시나리오 편집

다음 명령으로 편집할 수 있습니다：

- **추가**: 「기능 X의 데모를 추가」
- **삭제**: 「씬 2를 삭제」
- **변경**: 「인트로를 3초로 단축」
- **교체**: 「씬 2와 3을 교체」
- **완료**: 「이 것으로 OK」

무엇을 편집할까요?
```

---

## 출력 형식

planner.md 의 출력（generator.md 로의 입력）:

```yaml
video:
  type: "product-demo"
  total_duration: 45
  resolution: "1080p"
  fps: 30

scenes:
  - id: 1
    name: "intro"
    duration: 5
    template: "intro"
    content:
      title: "MyApp"
      tagline: "タスク管理を簡単に"
      logo: "public/logo.svg"

  - id: 2
    name: "auth-demo"
    duration: 15
    template: "ui-demo"
    source: "playwright"
    content:
      url: "http://localhost:3000/login"
      actions:
        - click: "[data-testid=email-input]"
        - type: "user@example.com"
        - click: "[data-testid=login-button]"

  - id: 3
    name: "dashboard"
    duration: 20
    template: "ui-demo"
    source: "playwright"
    content:
      url: "http://localhost:3000/dashboard"
      actions:
        - wait: 1000
        - scroll: "down"

  - id: 4
    name: "cta"
    duration: 5
    template: "cta"
    content:
      url: "https://myapp.com"
      text: "今すぐ試す"
```

---

## Notes

- 씬 수가 너무 많은 경우 자동으로 우선순위가 낮은 것을 제안에서 제외
- 사용자가 수동으로 씬을 추가하는 것도 가능
- Playwright 소스의 씬은 앱이起動している必要がある
