---
name: agent-browser
description: "브라우저를 손발처럼 조종합니다. 페이지 이동,フォーム 입력, 스크린샷, 무엇이든 OK。Use when users ask to navigate websites, fill forms, take screenshots, extract web data, test web apps, or automate browser workflows. Trigger phrases include 'go to [url]', 'click on', 'fill out the form', 'take a screenshot', 'scrape', 'automate', 'test the website', 'log into', or any browser interaction request. Do NOT load for: sharing URLs, embedding links, screenshot image files."
description-en: "Control browser like hands and feet. Navigate, fill forms, screenshot, bring it on. Use when users ask to navigate websites, fill forms, take screenshots, extract web data, test web apps, or automate browser workflows. Trigger phrases include 'go to [url]', 'click on', 'fill out the form', 'take a screenshot', 'scrape', 'automate', 'test the website', 'log into', or any browser interaction request. Do NOT load for: sharing URLs, embedding links, screenshot image files."
description-ja: "브라우저를 손발처럼 조종합니다. 페이지 이동,フォーム入力, 스크린샷, 무엇이든 OK。Use when users ask to navigate websites, fill forms, take screenshots, extract web data, test web apps, or automate browser workflows. Trigger phrases include 'go to [url]', 'click on', 'fill out the form', 'take a screenshot', 'scrape', 'automate', 'test the website', 'log into', or any browser interaction request. Do NOT load for: sharing URLs, embedding links, screenshot image files."
allowed-tools: ["Bash", "Read"]
user-invocable: false
context: fork
argument-hint: "[url] [--headless]"
---

# Agent Browser Skill

브라우저 자동화를 행하는 스킬. agent-browser CLI를 사용하여, UI 디버깅・검증・자동 조작을 실행합니다.

---

## 트리거 프레이즈

이 스킬は以下のフレーズ로 자동 기동합니다:

- 「페이지를 열어줘」「URL을 확인해줘」
- 「클릭해줘」「입력해줘」「폼에」
- 「스크린샷을 찍어줘」
- 「UI를 확인해줘」「화면을 테스트해줘」
- "open this page", "click on", "fill the form", "screenshot"

---

## 기능 상세

| 기능 | 상세 |
|------|------|
| **브라우저 자동화** | See [references/browser-automation.md](${CLAUDE_SKILL_DIR}/references/browser-automation.md) |
| **AI 스냅샷 워크플로** | See [references/ai-snapshot-workflow.md](${CLAUDE_SKILL_DIR}/references/ai-snapshot-workflow.md) |

## 실행 절차

### Step 0: agent-browser의 확인

```bash
# 설치 확인
which agent-browser

# 미설치의 경우
npm install -g agent-browser
agent-browser install
```

### Step 1: 사용자의 요청을 분류

| 요청 타입 | 대응 액션 |
|----------------|---------------|
| URL 열기 | `agent-browser open <url>` |
| 요소 클릭 | 스냅샷 → `agent-browser click @ref` |
| 폼 입력 | 스냅샷 → `agent-browser fill @ref "text"` |
| 상태 확인 | `agent-browser snapshot -i -c` |
| 스크린샷 | `agent-browser screenshot <path>` |
| 디버깅 | `agent-browser --headed open <url>` |

### Step 2: AI 스냅샷 워크플로 (권장)

대부분의 조작에서, 먼저 **스냅샷을 취득**してから要素参照로 조작합니다:

```bash
# 1. 페이지 열기
agent-browser open https://example.com

# 2. 스냅샷 취득（AI向け、インタラクティブ要素のみ）
agent-browser snapshot -i -c

# 출력예:
# - link "Home" [ref=e1]
# - button "Login" [ref=e2]
# - input "Email" [ref=e3]
# - input "Password" [ref=e4]
# - button "Submit" [ref=e5]

# 3. 요소 참조로 조작
agent-browser click @e2           # Login 버튼을 클릭
agent-browser fill @e3 "user@example.com"
agent-browser fill @e4 "password123"
agent-browser click @e5           # Submit
```

### Step 3: 결과의 확인

```bash
# 현재 상태를 스냅샷으로 확인
agent-browser snapshot -i -c

# 또는 URL을 확인
agent-browser get url

# 스크린샷을 취득
agent-browser screenshot result.png
```

---

##クイックリファレンス

### 기본 조작

| 커맨드 | 설명 |
|---------|------|
| `open <url>` | URL 열기 |
| `snapshot -i -c` | AI向け 스냅샷 |
| `click @e1` | 요소 클릭 |
| `fill @e1 "text"` | 폼에 입력 |
| `type @e1 "text"` | 텍스트 입력 |
| `press Enter` | 키를 누름 |
| `screenshot [path]` | 스크린샷 |
| `close` | 브라우저를 닫음 |

### 네비게이션

| 커맨드 | 설명 |
|---------|------|
| `back` | 뒤로 |
| `forward` | 앞으로 |
| `reload` | 리로드 |

### 정보 취득

| 커맨드 | 설명 |
|---------|------|
| `get text @e1` | 텍스트 취득 |
| `get html @e1` | HTML 취득 |
| `get url` | 현재 URL |
| `get title` | 페이지 타이틀 |

### 대기

| 커맨드 | 설명 |
|---------|------|
| `wait @e1` | 요소를 대기 |
| `wait 1000` | 1초 대기 |

### 디버깅

| 커맨드 | 설명 |
|---------|------|
| `--headed` | 브라우저를 표시 |
| `console` | 콘솔 로그 |
| `errors` | 페이지 에러 |
| `highlight @e1` | 요소를 하이라이트 |

---

## 세션 관리

복수의 탭/세션을 병렬 관리:

```bash
# 세션을 지정
agent-browser --session admin open https://admin.example.com
agent-browser --session user open https://example.com

# 세션 목록
agent-browser session list

# 특정 세션으로 조작
agent-browser --session admin snapshot -i -c
```

---

## MCP ブラウザツールとの使い分け

| 도구 | 권장도 | 용도 |
|--------|--------|------|
| **agent-browser** | ★★★ | 제일 선택. AI向け 스냅샷이 강력함 |
| chrome-devtools MCP | ★★☆ | Chrome이 이미 열려있는 경우 |
| playwright MCP | ★★☆ | 복잡한 E2E 테스트 |

**원칙**: 먼저 agent-browser를試して、うまくいかない 경우에만 MCP 도구를 사용.

---

## 주의 사항

- agent-browser는 헤드리스 모드가 기본
- `--headed` 옵션으로 브라우저를 표시 가능
- 세션은 명시적으로 `close`할 때까지 유지됨
- 인증이 필요한 사이트는 세션을 활용
