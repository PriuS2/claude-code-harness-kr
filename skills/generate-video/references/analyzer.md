# Video Analyzer - 코드베이스 분석 엔진

프로젝트를 자동 분석하고 영상 생성에 필요한 정보를 추출합니다.

---

## 개요

`/generate-video` 의 Step 1에서 실행되는 분석 엔진입니다.
코드베이스와 프로젝트 자산을 분석하여 최적의 영상 구성을 판정합니다.

## 분석 항목

### 1. 프레임워크 감지

| 감지 대상 | 판정 방법 |
|---------|---------|
| Next.js | `next.config.*` の存在 |
| React | `package.json` 의 dependencies |
| Vue | `vue.config.*` 또는 `nuxt.config.*` |
| Svelte | `svelte.config.*` |
| Express/Fastify | `package.json` 의 dependencies |

**실행 명령**:
```bash
# package.json から依存関係を抽出
cat package.json | jq '.dependencies, .devDependencies'

# 設定ファイルの存在確認
ls -la *.config.* 2>/dev/null
```

### 2. 주요 기능 감지

| 기능 | 감지 패턴 |
|------|-------------|
| 인증 | `auth/`, `login/`, `@clerk`, `@auth0`, `supabase` |
| 결제 | `payment/`, `billing/`, `stripe`, `@stripe` |
| 대시보드 | `dashboard/`, `admin/`, `analytics` |
| API | `api/`, `routes/`, `trpc`, `graphql` |
| DB | `prisma/`, `drizzle/`, `@supabase` |

**실행 명령**:
```bash
# 디렉토리 구조에서 기능을 추정
find src app -type d -name "auth" -o -name "login" -o -name "dashboard" 2>/dev/null

# 패키지에서 기능을 추정
grep -E "clerk|stripe|supabase|prisma" package.json
```

### 3. UI 컴포넌트 감지

| 항목 | 감지 방법 |
|------|---------|
| 페이지 수 | `app/**/page.tsx` 또는 `pages/**/*.tsx` 의 카운트 |
| 컴포넌트 수 | `components/**/*.tsx` 의 카운트 |
| UI 라이브러리 | `shadcn`, `radix`, `chakra`, `mui` 의 감지 |

**실행 명령**:
```bash
# 페이지 수 카운트
find . -name "page.tsx" -o -name "page.jsx" 2>/dev/null | wc -l

# 컴포넌트 수 카운트
find . -path "*/components/*" -name "*.tsx" 2>/dev/null | wc -l
```

### 4. 프로젝트 자산 분석

| 자산 | 용도 |
|------|------|
| `package.json` | 프로젝트명, description |
| `README.md` | 프로젝트 개요, 태그라인 |
| `Plans.md` | 완료タスク（릴리스 노트용）|
| `CHANGELOG.md` | 변경점（릴리스 노트용）|
| `.claude/memory/decisions.md` | 기술적 의사결정（아키텍처 해설용）|

**실행 명령**:
```bash
# 프로젝트 정보 추출
cat package.json | jq '{name, description, version}'

# README の最初の段落を抽出
head -20 README.md
```

---

## 영상 타입 자동 판정

### 판정 로직

```
분석 결과에서 영상 타입을 판정:
    │
    ├─ CHANGELOG が最近更新（7日以内）
    │   └─ → 릴리스 노트 영상
    │
    ├─ 큰 구조 변경（新ディレクトリ追加等）
    │   └─ → 아키텍처 해설
    │
    ├─ UI 변경 多（コンポーネント追加/変更）
    │   └─ → 프로덕트 데모
    │
    └─ 복수 조건に該当
        └─ → 복합 영상（사용자에게 확인）
```

### 판정 기준

| 타입 | 조건 |
|--------|------|
| **릴리스 노트** | `git log --since="7 days ago"` 에 tag/release 가 있음 |
| **아키텍처** | 새로운 `src/*/` 디렉토리, 대규모 리팩터 |
| **프로덕트 데모** | UI 컴포넌트의 추가/변경 |
| **기본값** | 프로덕트 데모（가장 범용적）|

---

## 출력 형식

분석 결과는以下の 형식으로 출력:

```yaml
project:
  name: "MyAwesomeApp"
  description: "태스크 관리를 쉽게"
  version: "1.2.0"

framework:
  primary: "Next.js"
  ui_library: "shadcn/ui"

features:
  - name: "인증"
    type: "auth"
    path: "src/app/(auth)/"
    provider: "Clerk"
  - name: "대시보드"
    type: "dashboard"
    path: "src/app/dashboard/"
  - name: "API"
    type: "api"
    path: "src/app/api/"

stats:
  pages: 12
  components: 45
  api_routes: 8

recent_changes:
  changelog_updated: true
  last_release: "2026-01-20"
  major_changes:
    - "인증 플로우 추가"
    - "대시보드 개선"

recommended_video_type: "release-notes"
confidence: 0.85
```

---

## 실행 예

```
📊 프로젝트 분석 중...

✅ 분석 완료

| 항목 | 결과 |
|------|------|
| 프로젝트명 | MyAwesomeApp |
| 프레임워크 | Next.js 14 |
| UI 라이브러리 | shadcn/ui |
| 페이지 수 | 12 |
| 컴포넌트 수 | 45 |

🔍 감지된 기능:
- 인증（Clerk）
- 대시보드
- API（8 엔드포인트）

📋 최근 변경:
- v1.2.0 릴리스（3일 전）
- 인증 플로우 추가
- 대시보드 개선

🎬 권장 영상 타입: 릴리스 노트 영상
   이유: 최근 릴리스가 있고 주요 기능 추가가 있습니다
```

---

## Notes

- 분석은 비파괴적（파일을 변경하지 않음）
- 대규모 프로젝트라도 수 초 내에 완료
- 감지되지 않는 기능은 수동으로 추가 가능（planner.md 에서）
