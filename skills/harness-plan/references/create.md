# create 서브명령어 — 계획 작성 플로우

아이디어·요구를ヒアリング하고 실행 가능한 Plans.md를 생성합니다.

## Step 0: 대화 컨텍스트 확인

이전 대화에서 요구사항을 추출할 수 있는 경우 확인합니다:

> 계획 작성 방법을 선택해 주세요:
> 1. 이전 대화에서 — 브레인스토밍 내용을 기반으로 계획 작성
> 2. 처음부터 —ヒアリングから開始

"이전 대화에서"의 경우: 요구사항·아이디어·결정 사항을 추출하여 사용자에게 확인.
확인 후, Step 3（기술 조사）으로 건너뜀.

## Step 1: 무엇을 만드는지 물어보기

사용자 입력이 없으면 질문합니다:

> 무엇을 만드나요?
>
> 예: 예약 관리 시스템 / 블로그 사이트 / 작업 관리 앱 / API 서버
>
> 대략적인 아이디어로 OK!

## Step 2: 해상도 높이기（최대 3문）

> 조금 더 알려주세요:
>
> 1. 누가 사용하나요? (자기만? 팀? 일반 공개?)
> 2. 참고하고 싶은 서비스가 있나요?
> 3. 어디까지 만드나요? (MVP? 풀 기능?)

## Step 3: 기술 조사（WebSearch）

사용자에게 묻지 않고 Claude Code가 조사·제안합니다.

```
WebSearch:
- "{{프로젝트 타입}} tech stack 2025"
- "{{유사 서비스}} architecture"
```

## Step 4: 기능 목록 추출

요구사항에서 구체적인 기능 목록을 추출합니다.

예: 예약 관리 시스템의 경우
- 사용자 등록/로그인
- 예약 캘린더 표시
- 예약 생성/편집/취소
- 관리자 대시보드
- 이메일 알림
- 결제 기능

## Step 4.5: optional brief 생성

필요할 때만 brief를 첨부. brief는 Plans.md를 대체하지 않고, 구현의 전제를 짧게 고정하는 보조 자료입니다.

- UI를 포함한 작업에서는 `design brief`
- API를 포함한 작업에서는 `contract brief`
- UI와 API가 혼재된 경우는 brief를 분리합니다

### design brief

UI 작업용 brief에는 최소한 다음을 포함합니다:

- 무엇을 달성したい는지
- 누가 사용하는지
- 중요한 화면 상태
- 외관이나 조작감의 제약
- 완료 조건

### contract brief

API 작업용 brief에는 최소한 다음을 포함합니다:

- 무엇을 받거나 반환하는지
- 입력 검증 조건
- 실패 시의 동작
- 외부 의존
- 완료 조건

## Step 5: 우선순위 매트릭스 작성（2축 평가）

각 기능을 **Impact（영향도）× Risk（리스크/불확실성）**의 2축으로 평가합니다:

- **Impact**: 사용자 가치 × 대상 사용자 수（높음/낮음）
- **Risk**: 기술적 미지 × 외부 의존（높음/낮음）

| Impact＼Risk | 저리스크 | 고리스크 |
|-------------|---------|---------|
| **고 Impact** | ★ **Required** — 최우선（확실히 가치가 나오는 것） | ▲ **Required + [needs-spike]** — 조기 검증 필요 |
| **저 Impact** | ○ **Recommended** — 여력으로 대응 | ✕ **Optional** — 미루기 또는 스코프 축소 |

### `[needs-spike]` 마커

고 Impact × 고 Risk의 작업에는 `[needs-spike]` 마커를 자동 부여합니다.
`[needs-spike]`가 붙은 작업에는 **spike（기술 검증）작업**을 자동 생성하여 선행시킵니다:

```markdown
| N.X-spike | [spike] {{작업명}} 의 기술 검증 | 검증 결과 리포트 작성 | - | cc:TODO |
| N.X       | {{작업명}} [needs-spike] | {{DoD}} | N.X-spike | cc:TODO |
```

spike 작업의 완료 조건은 "검증 결과 리포트（실현 가능/불가능/설계 변경 필요）를 남기는 것"입니다.

## Step 5.5: TDD 스킵 판단（기본有効）

TDD는 기본으로有効입니다. 다음 중 하나에 해당하는 작업만 `[skip:tdd]` 마커를 부여하여 스킵합니다:

| 스킵 조건 | 이유 |
|-------------|------|
| 문서/댓글만 | 실행 코드에 영향하지 않음 |
| 설정 파일만（JSON, YAML, .env） | 테스트 대상 로직이 없음 |
| 1행 이하의 단순 수정（typo） | 테스트 비용이 효과를 상회 |
| 스타일/포맷 변경만 | 동작에 영향하지 않음 |
| 의존 관계 업데이트만 | 구현 로직 변경 없음 |
| README/CHANGELOG 업데이트 | 문서만 |
| 리팩토링（동작 변경 없음） | 기존 테스트로 커버됨 |

위에 해당하지 않는 작업은 TDD가 자동 적용됩니다（테스트 선행 권장）.

## Step 5.7: Plans.md v3 포맷 사양

Plans.md v3은 다음의 포맷 확장을 포함합니다:

### Phase 헤더의 Purpose 행（선택）

각 Phase의 헤더에 1행의 Purpose（목적）를 기재할 수 있습니다. 입력이 없으면 생략합니다:

```markdown
### Phase N.X: [단계명] [Px]

Purpose: [이 단계가 해결할 과제를 1행으로]
```

- **기본값**: 입력을 요구하지 않음（빈칸으로 생략）
- **기재시의 효과**: breezing Phase 0의 스코프 확인에 표시됨
- **생성 규칙**: 사용자가 단계의 목적을 명시적으로 진술한 경우에만 자동 기재

### Artifact 표기（Status 컬럼）

작업 완료時に commit hash를 Status에 부여합니다:

```markdown
| Task | 내용 | DoD | Depends | Status |
|------|------|-----|---------|--------|
| 1.1  | ... | ... | - | cc:완료 [a1b2c3d] |
| 1.2  | ... | ... | 1.1 | cc:TODO |
```

- **형식**: `cc:완료 [7문자hash]`
- **부여 타이밍**: `harness-work` Solo Step 7에서 자동 부여
- **하위 호환**: 해시 없는 `cc:완료`도 계속有効

### 영향 파일 목록

v3 포맷에 관련된 파일:

| 파일 | 영향 |
|---------|------|
| `skills/harness-plan/references/create.md` | Step 6 템플릿에 Purpose 행 추가 |
| `skills/harness-plan/references/sync.md` | 차이 검출で `cc:완료 [hash]` 형식 인식 |
| `skills/harness-work/SKILL.md` | Solo Step 7에서 hash 부여, 실패時再티켓화 |
| `skills/harness-sync/SKILL.md` | --snapshot으로 스냅샷 저장 |
| `skills/breezing/SKILL.md` | Progress Feed에서 진행 표시 |

## Step 6: Plans.md 생성

품질 마커 + DoD + Depends를 자동 생성하여 Plans.md를 생성합니다.

### 품질 마커 부여 로직
```
작업 내용을 분석
    ↓
├── "auth" "login" "API" → [feature:security]
├── "component" "UI" "screen" → [feature:a11y]
├── "fix" "bug" → [bugfix:reproduce-first]
├── "docs" "comment" "README" "CHANGELOG" → [skip:tdd]
├── "config" "json" "yaml" "env" → [skip:tdd]
├── "style" "format" "lint" → [skip:tdd]
├── "refactor" (동작 변경 없음) → [skip:tdd]
├── "payment" "billing" → [feature:security]
└── 기타 → 마커 없음（TDD는 기본有効）

推論結果はあくまでデフォルト値。ユーザーが具体的な受入条件を指定した場合はそちらを優先する。

### DoD 자동推論 로직

作業の「内容」からキーワードベースで DoD を推論し、自动埋めする:

| 작업内容的 키워드 | DoD 推論 |
|---------------------|---------|
| "作成" "新規" "追加" | 파일이 존재하고 기대하는 구조를 가짐 |
| "テスト" "test" | 테스트 통과（`npm test` / `pytest` 등） |
| "修正" "fix" "bug" | 문제가 재현하지 않게 됨 |
| "UI" "画面" "コンポーネント" | 표시 확인（스크린샷 or 브라우저） |
| "API" "エンドポイント" | curl/httpie로 응답 확인 |
| "設定" "config" | 설정값이 반영됨 |
| "ドキュメント" "docs" | 파일이 존재하고 링크切れ 없음 |
| "マイグレーション" "DB" | 마이그레이션 실행 가능 |
| "リファクタリング" | 기존 테스트 전 통과 + lint 에러 0 |

### Depends 자동推論 로직

단계 내 작업 간의 의존 관계를 다음 규칙으로推論します:

1. **DB/스키마系 작업** → 다른 구현 작업에서 의존됨（선행 작업）
2. **UI 작업** → API/로직 작업에 의존（후행 작업）
3. **테스트/검증 작업** → 구현 작업에 의존（마지막）
4. **설정/환경 작업** → 다른 작업에서 의존됨（선행 작업）
5. **명확한 의존이 없는 작업** → `-`（병렬 실행 가능）

推論に自信がない場合は `-`にして、ユーザーに確認を求める。

**生成テンプレート**:

```markdown
# [프로젝트명] Plans.md

작성일: YYYY-MM-DD

---

## Phase 1: [단계명]

Purpose: [단계의 목적（생략可）]

| Task | 내용 | DoD | Depends | Status |
|------|------|-----|---------|--------|
| 1.1  | [작업 설명] [feature:security] | [검증 가능한 완료 조건] | - | cc:TODO |
| 1.2  | [작업 설명] | [검증 가능한 완료 조건] | 1.1 | cc:TODO |
```

**Purpose 행**:
- 사용자가 단계의 목적을 진술한 경우에만 자동 기재
- 입력이 없으면 Purpose 행ごと 생략（빈 행으로 두지 않음）
- 1행으로完結（複数行 금지）

**DoD（Definition of Done）記법**:
- 검증 가능한 1행으로 작성（예: "테스트 통과""마이그레이션 실행 가능""lint 에러 0"）
- "잘 됐으면 좋겠어""제대로 작동해"는 금지. Yes/No로 판정 가능한 형태로 작성

**Depends 記법**:
- 의존 없음: `-`
- 단일 의존: 작업 번호（예: `1.1`）
- 복수 의존: 쉼표 구분（예: `1.1, 1.2`）
- 단계 의존: 단계 번호（예: `Phase 1`）

### Team mode output

사용자가 team mode를 명시한 경우에만, Plans.md와 별도로 issue bridge의 dry-run도 안내합니다.

- tracking issue는 1개만
- 작업별 sub-issue payload를 나열
- Plans.md는 정본을 유지
- `scripts/plans-issue-bridge.sh --team-mode`의 dry-run을 그대로 사용할 수 있는 형태로 안내

## Step 7: 다음 액션 안내

> Plans.md 완성!
>
> 다음 단계:
> - `harness-work`로 구현 시작
> - 또는 "Phase 1부터 시작해줘"라고 말하기
> - 기능 추가는 `harness-plan add [기능명]`
> - 기능 후투스는 `harness-plan update [작업] blocked`

## CI 모드（--ci）

ヒアリングなし. 기존 Plans.md를 그대로 이용하여 작업 분해만 수행.

1. Plans.md를 읽음
2. cc:TODO 작업을 우선순위 순으로 목록화
3. 병렬 가능한 작업에 `[P]` 마크 부여
4. 다음 실행 작업 제안
