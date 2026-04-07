---
name: ci-fix-failing-tests
description: "CI에서 실패한 테스트를 수정하기 위한 가이드. CI실패의 원인이 특정된 후, 자동 수정을 시도하려는 경우 사용합니다."
allowed-tools: ["Read", "Edit", "Bash"]
---

# CI Fix Failing Tests

CI에서 실패한 테스트를 수정하는 스킬.
테스트 코드의 수정, 또는 본체 코드의 수정을 행합니다.

---

## 입력

- **실패 테스트 정보**: 테스트명, 에러 메시지
- **테스트 파일**: 실패한 테스트의 소스
- **테스트 대상 코드**: 테스트 대상의 구현

---

## 출력

- **수정된 코드**: 테스트 또는 구현의 수정
- **테스트 통과 확인**

---

## 실행 절차

### Step 1: 실패 테스트의 특정

```bash
# 로컬에서 테스트 실행
npm test 2>&1 | tail -50

# 특정 파일의 테스트
npm test -- {{test-file}}
```

### Step 2: 에러 타입의 분류

#### 타입 A: 어설션 실패

```
Expected: "expected value"
Received: "actual value"
```

→ 구현이 기대와 다르거나, 테스트의 기대값이 잘못됨

#### 타입 B: 타임아웃

```
Timeout - Async callback was not invoked within the 5000ms timeout
```

→ 비동기 처리가 완료되지 않거나, 시간이 너무 오래 걸림

#### 타입 C: 형 에러

```
TypeError: Cannot read properties of undefined
```

→ null/undefined 의 접근, 또는 초기화의 문제

#### 타입 D: 모의 관련

```
expected mockFn to have been called
```

→ 모의 설정 부족, 또는 호출이 행해지지 않음

### Step 3: 수정 전략의 결정

```markdown
## 수정 방버判断

1. **테스트가 올바른 경우** → 구현을 수정
2. **구현이 올바른 경우** → 테스트를 수정
3. **둘 다 수정이 필요**   → 구현을 우선

판단 기준:
- 사양・요구사항에 비추어どちらが正しいか
- 최근 변경은何か
- 다른 테스트에 대한 영향
```

### Step 4: 수정의 구현

#### 어설션 실패의 수정

```typescript
// 테스트의 기대값이 잘못된 경우
it('calculates correctly', () => {
  // 수정 전
  expect(calculate(2, 3)).toBe(5)
  // 수정 후（사양이 곱셈인 경우）
  expect(calculate(2, 3)).toBe(6)
})

// 구현이 잘못된 경우
// → 구현 파일을 수정
```

#### 타임아웃의 수정

```typescript
// 타임아웃을 연장
it('fetches data', async () => {
  // ...
}, 10000)  // 10초로 연장

// 또는 async/await를 올바르게 사용
it('fetches data', async () => {
  await waitFor(() => {
    expect(screen.getByText('Data')).toBeInTheDocument()
  })
})
```

#### 모의 관련의 수정

```typescript
// 모의 설정 추가
vi.mock('../api', () => ({
  fetchData: vi.fn().mockResolvedValue({ data: 'mock' })
}))

// beforeEach에서 리셋
beforeEach(() => {
  vi.clearAllMocks()
})
```

### Step 5: 수정 후의 확인

```bash
# 실패 테스트를 재실행
npm test -- {{test-file}}

# 전체 테스트 실행（리그레션 확인）
npm test
```

---

## 수정 패턴 모음

### 스냅샷 업데이트

```bash
# 스냅샷의 업데이트
npm test -- -u

# 특정 테스트만
npm test -- {{test-file}} -u
```

### 비동기 테스트의 수정

```typescript
// findBy를 사용（자동 대기）
const element = await screen.findByText('Text')

// waitFor를 사용
await waitFor(() => {
  expect(mockFn).toHaveBeenCalled()
})
```

### 모의 데이터의 업데이트

```typescript
// 구현의 변경에 맞춰 모의를 업데이트
const mockData = {
  id: 1,
  name: 'Test',
  createdAt: new Date().toISOString()  // 새로운 필드
}
```

---

## 수정 후의 체크리스트

- [ ] 실패하던 테스트가 통과함
- [ ] 다른 테스트가 고장나지 않음
- [ ] 구현의 의도와 일치함
- [ ] 너무 완화된 테스트가 되지 않음

---

## 완료 보고 포맷

```markdown
## ✅ 테스트 수정 완료

### 수정 내용

| 테스트 | 문제 | 수정 |
|-------|------|------|
| `{{테스트명}}` | {{문제}} | {{수정 내용}} |

### 확인 결과

```
Tests: {{passed}} passed, {{total}} total
```

### 다음 액션

「커밋해줘」또는「CI를 재실행해줘」
```

---

## 주의 사항

- **테스트를 삭제하지 말 것**: 삭제는 최후의 수단
- **skip은 일시적으로**: 항구적인 skip은 금지
- **근본 원인을 특정**: 표면적인 수정을 피할 것
