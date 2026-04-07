# AI Snapshot Workflow

agent-browser의 `snapshot` 커맨드를活用한 AI 에이전트向け 워크플로.

---

## 개요

`snapshot` 커맨드는 페이지의 접근성 트리를 취득하여, 각 요소에 참조 ID（`@e1`, `@e2` 等）를 부여합니다. これにより:

1. **CSSセレクタ不要**: 동적인 ID나 클래스 명에 의존하지 않음
2. **컨텍스트 파악**: 요소의 역할（button, input, link）이 명확함
3. **결정적 조작**: `@e1` などの参照で確実に操作可能

---

## 基本 워크플로

### Step 1: 페이지 열기

```bash
agent-browser open https://example.com
```

### Step 2: 스냅샷 취득

```bash
agent-browser snapshot -i -c
```

**옵션 설명**:
- `-i, --interactive`: 인터랙티브한 요소（버튼, 링크, 입력 필드 等）만 표시
- `-c, --compact`: 빈 구조 요소 제거해서 콤팩트하게

**출력예**:
```
✓ Example Domain
  https://example.com/

- link "Home" [ref=e1]
- link "About" [ref=e2]
- button "Login" [ref=e3]
- input "Search" [ref=e4]
- button "Search" [ref=e5]
```

### Step 3: 요소 참조로 조작

```bash
# 링크를 클릭
agent-browser click @e1

# 검색 폼에 입력
agent-browser fill @e4 "search query"

# 검색 버튼을 클릭
agent-browser click @e5
```

### Step 4: 결과を確認

```bash
# 새로운 상태를 스냅샷
agent-browser snapshot -i -c
```

---

## Snapshot オプション詳解

### `-i, --interactive`

인터랙티브한 요소만 표시. 조작 대상을 좁히고 싶을 때有効.

```bash
# 인터랙티브 요소만
agent-browser snapshot -i

# 전체 요소（텍스트 노드 포함）
agent-browser snapshot
```

### `-c, --compact`

빈 구조 요소（div, span 等 内容のないもの）を除去.

```bash
# 콤팩트 출력
agent-browser snapshot -c

# 구조도 포함하여 표시
agent-browser snapshot
```

### `-d, --depth <n>`

트리의 깊이를 제한. 큰 페이지에서 개요를 파악하고 싶을 때有効.

```bash
# 깊이 3까지
agent-browser snapshot -d 3
```

### `-s, --selector <sel>`

특정 셀렉터에 스코프를絞る.

```bash
# 폼 내만
agent-browser snapshot -s "form.login"

# 네비게이션 내만
agent-browser snapshot -s "nav"
```

### 조합

```bash
# 권장: 인터랙티브 + 콤팩트
agent-browser snapshot -i -c

# 폼 내의 인터랙티브 요소만
agent-browser snapshot -i -c -s "form"

# 얕은 트리로 개요 파악
agent-browser snapshot -i -d 2
```

---

## 유즈케이스별 워크플로

### 로그인 플로

```bash
# 1. 로그인 페이지를 열기
agent-browser open https://example.com/login

# 2. 스냅샷 취득
agent-browser snapshot -i -c
# 출력:
# - input "Email" [ref=e1]
# - input "Password" [ref=e2]
# - button "Login" [ref=e3]
# - link "Forgot password?" [ref=e4]

# 3. 로그인 정보 입력
agent-browser fill @e1 "user@example.com"
agent-browser fill @e2 "password123"

# 4. 로그인 버튼 클릭
agent-browser click @e3

# 5. 결과 확인
agent-browser snapshot -i -c
agent-browser get url
```

### 폼 전송

```bash
# 1. 폼 페이지를 열기
agent-browser open https://example.com/contact

# 2. 폼 내의 스냅샷
agent-browser snapshot -i -c -s "form"
# 출력:
# - input "Name" [ref=e1]
# - input "Email" [ref=e2]
# - textarea "Message" [ref=e3]
# - button "Send" [ref=e4]

# 3. 폼에 입력
agent-browser fill @e1 "John Doe"
agent-browser fill @e2 "john@example.com"
agent-browser fill @e3 "Hello, this is a test message."

# 4. 전송
agent-browser click @e4

# 5. 확인
agent-browser snapshot -i -c
```

### 네비게이션 탐색

```bash
# 1. 토프 페이지를 열기
agent-browser open https://example.com

# 2. 네비게이션 확인
agent-browser snapshot -i -c -s "nav"
# 출력:
# - link "Home" [ref=e1]
# - link "Products" [ref=e2]
# - link "About" [ref=e3]
# - link "Contact" [ref=e4]

# 3. Products 페이지로
agent-browser click @e2

# 4. 새로운 페이지의 구조 확인
agent-browser snapshot -i -c
```

### 동적 콘텐츠의 조작

```bash
# 1. 페이지 열기
agent-browser open https://example.com/dashboard

# 2. 초기 스냅샷
agent-browser snapshot -i -c

# 3. 드롭다운 열기
agent-browser click @e5

# 4. 대기（동적 콘텐츠의 로드）
agent-browser wait 500

# 5. 새로운 스냅샷（드롭다운 메뉴가 표시됨）
agent-browser snapshot -i -c
# 새로운 요소が出현:
# - menuitem "Option 1" [ref=e10]
# - menuitem "Option 2" [ref=e11]
# - menuitem "Option 3" [ref=e12]

# 6. 옵션을 선택
agent-browser click @e11
```

---

## 트러블슈팅

### 요소를 찾을 수 없는 경우

```bash
# 풀 스냅샷（모든 요소）
agent-browser snapshot

# 특정 셀렉터로 좁히기
agent-browser snapshot -s "#target-element"

# 대기 후 재시도
agent-browser wait 2000
agent-browser snapshot -i -c
```

### 동적 페이지

```bash
# JavaScript 실행 후 스냅샷
agent-browser eval "document.querySelector('#load-more').click()"
agent-browser wait 1000
agent-browser snapshot -i -c
```

### iframe 내의 요소

```bash
# 메인 프레임의 스냅샷
agent-browser snapshot -i -c

# iframe 내는 직접 액세스할 수 없으므로,
# eval로 iframe 내의 조작을 행함
agent-browser eval "document.querySelector('iframe').contentDocument.querySelector('button').click()"
```

---

## 베스트 프랙티스

### 1. 항상 스냅샷부터 시작

조작 전에 반드시 스냅샷을 취득하여, 현재의 상태를 파악합니다.

### 2. 인터랙티브 + 콤팩트를 기본으로

```bash
agent-browser snapshot -i -c
```

### 3. 조작 후는 상태를 확인

```bash
agent-browser click @e1
agent-browser snapshot -i -c  # 결과 확인
```

### 4. 적절한 대기를 넣을 것

동적 콘텐츠가 있는 경우는 대기를 넣을 것:

```bash
agent-browser click @e1
agent-browser wait 500
agent-browser snapshot -i -c
```

### 5. 세션을 활용할 것

인증 상태를 유지하기 위해서 세션을 사용:

```bash
agent-browser --session myapp open https://example.com/login
# ... 로그인 조작 ...
# 이후, 같은 세션으로 조작 계속
agent-browser --session myapp open https://example.com/dashboard
```
