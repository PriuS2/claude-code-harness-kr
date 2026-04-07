# Browser Automation with agent-browser

agent-browser CLI를 사용한 브라우저 자동화의 상세 가이드.

---

## 설치

```bash
# 글로벌 설치
npm install -g agent-browser

# Chromium을 다운로드
agent-browser install

# Linux의 경우, 시스템 의존성도
agent-browser install --with-deps
```

---

## 기본 조작

### 페이지 열기

```bash
# 기본
agent-browser open https://example.com

# 브라우저를 표시하여 열기（디버깅용）
agent-browser open https://example.com --headed

# 커스텀 헤더 포함
agent-browser open https://api.example.com --headers '{"Authorization": "Bearer token"}'
```

### 클릭

```bash
# 요소 참조로 클릭（권장）
agent-browser click @e1

# CSS 셀렉터로 클릭
agent-browser click "button.submit"

# 더블 클릭
agent-browser dblclick @e1
```

### 입력

```bash
# 폼을 클리어&입력
agent-browser fill @e1 "hello@example.com"

#追加入력（클리어하지 않음）
agent-browser type @e1 "追加テキスト"

# 키를 누름
agent-browser press Enter
agent-browser press Tab
agent-browser press "Control+a"
```

### 폼 조작

```bash
# 체크박스
agent-browser check @e1
agent-browser uncheck @e1

# 셀렉트 박스
agent-browser select @e1 "option-value"

# 파일 업로드
agent-browser upload @e1 /path/to/file.pdf
```

### 스크롤

```bash
# 방향 지정
agent-browser scroll down
agent-browser scroll up 500

# 요소를 표시
agent-browser scrollintoview @e1
```

---

## 정보 취득

```bash
# 텍스트 취득
agent-browser get text @e1

# HTML 취득
agent-browser get html @e1

# 어트리뷰트 취득
agent-browser get attr href @e1

# 값 취득（input）
agent-browser get value @e1

# 현재 URL
agent-browser get url

# 페이지 타이틀
agent-browser get title

# 요소 수
agent-browser get count "li.item"

# 요소의 위치와 사이즈
agent-browser get box @e1
```

---

## 상태 체크

```bash
# 표시되어 있는가
agent-browser is visible @e1

#有効か（disabledでないか）
agent-browser is enabled @e1

# 체크되어 있는가
agent-browser is checked @e1
```

---

## 대기

```bash
# 요소가 표시될 때까지 대기
agent-browser wait @e1
agent-browser wait "button.loaded"

# 시간으로 대기（밀리초）
agent-browser wait 2000
```

---

## 스크린샷

```bash
# 기본
agent-browser screenshot

# 파일명 지정
agent-browser screenshot output.png

# 풀 페이지
agent-browser screenshot --full page.png

# PDF로 저장
agent-browser pdf document.pdf
```

---

## JavaScript 실행

```bash
# 스크립트 실행
agent-browser eval "document.title"
agent-browser eval "localStorage.getItem('token')"
agent-browser eval "window.scrollTo(0, document.body.scrollHeight)"
```

---

## 네트워크 조작

```bash
# 요청을 모의
agent-browser network route "*/api/users" --body '{"users": []}'

# 요청을 블로
agent-browser network route "*/analytics/*" --abort

# 라우트 해제
agent-browser network unroute "*/api/users"

# 요청 히스토리
agent-browser network requests
agent-browser network requests --filter "api"
agent-browser network requests --clear
```

---

## Cookie/Storage

```bash
# Cookie 취득
agent-browser cookies get

# Cookie 설정
agent-browser cookies set '{"name": "session", "value": "abc123", "domain": "example.com"}'

# Cookie 클리어
agent-browser cookies clear

# LocalStorage
agent-browser storage local get "key"
agent-browser storage local set "key" "value"
agent-browser storage local clear

# SessionStorage
agent-browser storage session get "key"
```

---

## 탭 관리

```bash
# 새로운 탭 열기
agent-browser tab new

# 탭 목록
agent-browser tab list

# 탭 전환
agent-browser tab 2

# 탭을 닫음
agent-browser tab close
```

---

## 브라우저 설정

```bash
# 뷰포트 사이즈
agent-browser set viewport 1920 1080

# 디바이스 에뮬레이션
agent-browser set device "iPhone 12"

# 위치 정보
agent-browser set geo 35.6762 139.6503

# 오프라인 모드
agent-browser set offline on
agent-browser set offline off

# 다크 모드
agent-browser set media dark
agent-browser set media light

# 인증 정보
agent-browser set credentials admin password123
```

---

## 디버깅

```bash
# 콘솔 로그 표시
agent-browser console
agent-browser console --clear

# 페이지 에러 표시
agent-browser errors
agent-browser errors --clear

# 요소 하이라이트
agent-browser highlight @e1

# 트레이스 녹화
agent-browser trace start
# ... 조작 ...
agent-browser trace stop trace.zip
```

---

## Find 커맨드（고级的 요소 검색）

```bash
# 역할로 검색하여 클릭
agent-browser find role button click --name "Submit"

# 텍스트로 검색
agent-browser find text "Click here" click

# 라벨로 검색
agent-browser find label "Email" fill "test@example.com"

# 플레이스홀더로 검색
agent-browser find placeholder "Enter your name" fill "John"

# 테스트 ID로 검색
agent-browser find testid "submit-btn" click

# 처음/마지막/n번째
agent-browser find first "button" click
agent-browser find last "input" fill "text"
agent-browser find nth 2 "li" click
```

---

## 마우스 조작（로우레벨）

```bash
# 마우스 이동
agent-browser mouse move 100 200

# 마우스 버튼
agent-browser mouse down
agent-browser mouse up
agent-browser mouse down right

# 휠
agent-browser mouse wheel 100
agent-browser mouse wheel 100 50  # dy, dx
```

---

## 드래그&드롭

```bash
# 요소 간의 드래그
agent-browser drag @e1 @e2

# 좌표 지정
agent-browser drag @e1 "500,300"
```

---

## 세션 관리

```bash
# 네임드 세션
agent-browser --session myapp open https://example.com

# 세션 목록
agent-browser session list

# 현재 세션명
agent-browser session

# 환경 변수로도 지정 가능
AGENT_BROWSER_SESSION=myapp agent-browser snapshot
```

---

## JSON 출력

```bash
# JSON 포맷으로 출력
agent-browser snapshot --json
agent-browser get text @e1 --json
agent-browser network requests --json
```

---

## 커스텀 브라우저

```bash
# 커스텀 실행 파일
agent-browser --executable-path /path/to/chrome open https://example.com

# 환경 변수로도 지정 가능
AGENT_BROWSER_EXECUTABLE_PATH=/path/to/chrome agent-browser open https://example.com
```
