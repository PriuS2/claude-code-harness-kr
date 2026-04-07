# Security Reviewer Profile

`harness-review --security`로起動하는 보안 전용 리뷰 프로필입니다.
OWASP Top 10을 기반으로, 인증·인가·기밀 정보·의존 패키지의 취약성을 포괄적으로 체크합니다.

> **Read-only 제약**: 이 프로필로 동작하는 reviewer는
> Read / Grep / Glob / Bash（읽기 전용 명령만）를 사용합니다.
> Write / Edit / 쓰기 系 Bash는一切実行하지 않습니다.

---

## Security Review 플로우

### Step 1: 대상 범위를特定

```bash
# 변경 파일을 수집（BASE_REF는 호출원에서引き継ぐ）
CHANGED_FILES="$(git diff --name-only --diff-filter=ACMR "${BASE_REF:-HEAD~1}")"
git diff "${BASE_REF:-HEAD~1}" -- ${CHANGED_FILES}
```

### Step 2: OWASP Top 10 체크

각 항목을 **변경 차분**과 **관련 파일**에 대해 확인합니다.

#### A01: 액세스 제어 불량 (Broken Access Control)

| 체크 항목 | 확인 방법 |
|------------|---------|
| 승인 체크 누락 | 라우트/엔드포인트 정의에 인증 미들웨어가 적용되어 있는지 |
| 수평越高권 액세스 | 사용자 소유 리소스 취득時に `userId` 등으로 필터링하고 있는지 |
| 수직越高권 액세스 | 롤 체크（admin/user/guest 등）가 적절히 구현되어 있는지 |
| IDOR | URL 파라미터나 요청 본문의 ID가 승인 없이受け入れ되고 있는지 |
| 디렉토리 트래버설 | `../`를含むパス操作がサニタイズされているか |

**검출 패턴（Grep로 확인）**:
```bash
# 인증 없음 라우트 후보
grep -rn "app\.\(get\|post\|put\|delete\|patch\)" --include="*.ts" --include="*.js"
# userId 없이 DB 취득
grep -rn "findById\|findOne\|select.*where" --include="*.ts"
```

#### A02: 암호화 실패 (Cryptographic Failures)

| 체크 항목 | 확인 방법 |
|------------|---------|
| 평문での機密情報保存 | 비밀번호, 토큰, PII가 평문으로 DB/로그에 저장되어 있는지 |
|弱的 해시 알고리즘 | MD5 / SHA1をパスワードハッシュに使用していないか |
| 안전하지 않은 랜덤 | `Math.random()`을 인증 토큰 생성에 사용하고 있는지 |
| TLS 강도 | HTTP（非HTTPS）での機密データ送受信がないか |
| 키のハードコード | 암호키·IV가 상수로埋め겨져 있지 있는지 |

**검출 패턴**:
```bash
grep -rn "md5\|sha1\|Math\.random\(\)" --include="*.ts" --include="*.js"
grep -rn "createHash.*md5\|createHash.*sha1" --include="*.ts"
grep -rn "http://" --include="*.ts" --include="*.js" --include="*.env*"
```

#### A03: 인젝션 (Injection)

| 체크 항목 | 확인 방법 |
|------------|---------|
| SQL 인젝션 | 사용자 입力を 문자열 결합으로 SQL에組み込んでいる지 |
| NoSQL 인젝션 | MongoDB 등에서 `$where`나 입력 값을 연산자로 사용하고 있는지 |
| 커맨드 인젝션 | `exec()` / `spawn()`에 사용자 입력을 전달하고 있는지 |
| LDAP 인젝션 | LDAP 쿼리에 살균 처리 없는 입력을 사용하고 있는지 |
| 템플릿 인젝션 | 템플릿 엔진에 사용자 입력을 직접 전달하고 있는지 |

**검출 패턴**:
```bash
grep -rn "exec\|execSync\|spawn" --include="*.ts" --include="*.js"
grep -rn "\`SELECT\|\"SELECT\|'SELECT" --include="*.ts" --include="*.js"
grep -rn "\$where\|\$\[" --include="*.ts" --include="*.js"
```

#### A04:不安全한 설계 (Insecure Design)

| 체크 항목 | 확인 방법 |
|------------|---------|
| 레이트 리밋 부족 | 인증 엔드포인트에 레이트 리밋이 구현되어 있는지 |
| TOCTOU 경쟁 상태 | 체크 後・使用前の状態変更を悪用できないか |
| 비즈니스 로직 결함 | 상태 전이가不正한 순서로 실행되지 않는지 |

#### A05: 보안 설정 오류 (Security Misconfiguration)

| 체크 항목 | 확인 방법 |
|------------|---------|
| 기본 인증 정보 | 기본 비밀번호/사용자명이 그대로 사용되지 않고 있는지 |
| 상세 에러 메시지 | 스택 트레이스나 내부 정보가 本番でクライアントに返されないか |
| 불필요한 기능의有効化 | 디버그 엔드포인트/관리 화면이 本番で有効でないか |
| HTTP 보안 헤더 | HSTS, CSP, X-Frame-Options 등이 설정되어 있는지 |
| CORS 설정 | `Access-Control-Allow-Origin: *`가 本番で設定されていないか |

**검출 패턴**:
```bash
grep -rn "cors.*origin.*\*\|allowedOrigins.*\*" --include="*.ts" --include="*.js"
grep -rn "debug.*true\|NODE_ENV.*development" --include="*.ts"
grep -rn "console\.log.*password\|console\.log.*token\|console\.log.*secret" --include="*.ts"
```

#### A06: 취약하고 오래된 컴포넌트 (Vulnerable and Outdated Components)

| 체크 항목 | 확인 방법 |
|------------|---------|
| 알려진 취약성을 가진 패키지 | `package.json`의 의존 관계에 CVE가 보고된 버전이 있는지 |
| `npm audit` 의 결과 | high / critical 취약성이 방치되어 있지는지 |
|ロックファイルとの整合性 | `package-lock.json` / `yarn.lock`이 最新か |

**확인 명령**:
```bash
# package.json의 의존 관계를 확인（読み取りのみ）
cat package.json | grep -E '"dependencies"|"devDependencies"' -A 50 | head 60
#ロックファイルの存在確認
ls -la package-lock.json yarn.lock pnpm-lock.yaml 2>/dev/null
```

#### A07: 식별과 인증의 실패 (Identification and Authentication Failures)

| 체크 항목 | 확인 방법 |
|------------|---------|
| 브루트포스 방지 | 로그인 시도 횟수 제한·계정 잠금이 구현되어 있는지 |
|弱的 비밀번호 정책 | 최소 문자 수·복잡성 요건이 설정되어 있는지 |
| 세션 고정 공격 | 로그인後にセッション ID가 再生成されているか |
| 세션有効期限 | 장기간 유효한 세션/토큰이 적절히失効するか |
| JWT 검증 | `alg: none`이나脆弱한鍵での署名を受け入れていないか |

**검출 패턴**:
```bash
grep -rn "jwt\.verify\|jwt\.sign" --include="*.ts" --include="*.js"
grep -rn "expiresIn.*\|expire.*" --include="*.ts"
grep -rn "algorithm.*none\|alg.*none" --include="*.ts" --include="*.js"
```

#### A08: 소프트웨어와 데이터 무결성의 실패 (Software and Data Integrity Failures)

| 체크 항목 | 확인 방법 |
|------------|---------|
| 신뢰할 수 없는 소스からのコード実行 | 외부 CDN / URLから動的にスクリプトを読み込んでいないか |
| 역직렬화 | 신뢰할 수 없는 데이터를 직접 `eval()` / `Function()`에 전달하고 있는지 |
| CI/CD 파이프라인의 보호 | 빌드 스크립트가 외부 입력을 무 검증으로 실행하고 있는지 |

**검출 패턴**:
```bash
grep -rn "eval(\|new Function(" --include="*.ts" --include="*.js"
grep -rn "require(.*\$\|import(.*\$" --include="*.ts" --include="*.js"
```

#### A09: 보안 로그와 모니터링의 실패 (Security Logging and Monitoring Failures)

| 체크 항목 | 확인 방법 |
|------------|---------|
| 인증 실패의 로그 | 로그인 실패·권한 에러가 기록되어 있는지 |
| 기밀 정보의 로그 출력 | 비밀번호·토큰·PII가 로그에 포함되어 있지는지 |
| 로그 인젝션 | 사용자 입력이 로그에 직접 쓰여지고 있는지（CRLF 인젝션） |

#### A10: 서버 사이드 요청 위조 (SSRF)

| 체크 항목 | 확인 방법 |
|------------|---------|
| 사용자 지정 URLへのリクエスト | 사용자 입력의 URL에 대해 내부 네트워크 액세스가 가능한지 |
| URL 밸리데이션 | 허용 도메인 목록이나 IP 필터링이 구현되어 있는지 |
| 리다이렉트 추종 | 요청 라이브러리가 내부 주소로의 리다이렉트를 추종하지 않는지 |

**검출 패턴**:
```bash
grep -rn "fetch(\|axios\.\|got(\|request(" --include="*.ts" --include="*.js"
```

---

## 인증·인가 리뷰 포인트

### 인증 플로우

```
1. 입력 검증 → 型・長さ・形式チェックがあるか
2. 인증 처리 →タイミング攻撃対策（constantTimeCompare等）があるか
3. 토큰発行 →十分なエントロピー（crypto.randomBytes等）があるか
4. 토큰保存 → httpOnly + Secure + SameSite Cookieか、LocalStorageか
5. 토큰検証 →署名・有効期限・失効チェックが完全か
6. 로그아웃 → 서버側でのトークン無効화가実装されているか
```

### 인가 플로우

```
1. 각 엔드포인트에 필요한 역할이 명시되어 있는지
2. 미들웨어와 라우트 핸들러 양쪽에서 체크되고 있는지（다층 방어）
3. 프론트엔드의 비표시에만 의존하지 않고 있는지（백엔드 필수）
4. 리소스 오너십 검증이 누락되지 않고 있는지
```

---

## 기밀 정보의 다루기

### 하드코드 검출

```bash
# API 키·시크릿 비슷한 패턴
grep -rn "api[_-]key\s*=\s*['\"][^'\"]\|secret\s*=\s*['\"][^'\"]" \
  --include="*.ts" --include="*.js" --include="*.sh"

# AWS / GCP / Azure 인증 정보
grep -rn "AKIA\|sk-[a-zA-Z0-9]\{20\}\|AIza" --include="*.ts" --include="*.js"

# JWT 서명 키의 하드코드
grep -rn "jwt.*secret.*=\s*['\"][^'\"]\{8,\}" --include="*.ts" --include="*.js"

# .env 파일へのコミット
git diff "${BASE_REF:-HEAD~1}" -- .env .env.local .env.production
```

### 환경 변수의 적절한 이용

| 좋은 패턴 | 나쁜 패턴 |
|------------|------------|
| `process.env.DATABASE_URL` | `"postgresql://user:pass@localhost/db"` |
| `process.env.JWT_SECRET` | `const JWT_SECRET = "my-super-secret"` |
| `process.env.API_KEY` | `const API_KEY = "sk-abc123..."` |

### .env 파일의 관리

- `.env.example`에 더미 값이 기재되어 있는지
- `.env` / `.env.local`이 `.gitignore`에 포함되어 있는지
- 本番 시크릿이 `.env.production`에 커밋되지 않았는지

```bash
# .gitignore의 확인
grep -n "\.env" .gitignore 2>/dev/null
# 리포지에 .env 파일이 포함되어 있는지
git diff "${BASE_REF:-HEAD~1}" --name-only | grep "\.env"
```

---

## 의존 패키지의 알려진 취약성 체크

### package.json의 확인 절차

1. 변경된 `package.json`을 읽음
2.新規追加・バージョンアップされたパッケージ을特定
3. 알려진 CVE 데이터베이스（NVD, Snyk, GitHub Advisory）との照合を推奨

```bash
# 변경된 패키지를 확인
git diff "${BASE_REF:-HEAD~1}" -- package.json package-lock.json

#現在の依存関係バージョンを確認
cat package.json | python3 -c "import json,sys; d=json.load(sys.stdin); [print(k,v) for d2 in [d.get('dependencies',{}),d.get('devDependencies',{})] for k,v in d2.items()]" 2>/dev/null
```

### 고리스크 패키지 카테고리

| 카테고리 | 주의점 |
|---------|--------|
| 인증 라이브러리 | passport, jsonwebtoken, bcrypt — 버전에 의존한 취약성이 많음 |
| HTTP 클라이언트 | axios, node-fetch, got — SSRF 대책의 기본 설정を確認 |
| 템플릿 엔진 | handlebars, ejs, pug — RCE 취약성의 과거 사례 있음 |
| XML 파서 | xml2js, fast-xml-parser — XXE 공격에 주의 |
| 직렬화 | serialize-javascript, node-serialize — RCE 리스크 |
| 이미지 처리 | sharp, imagemagick — 버퍼 오버플로 계열의 취약성 |

---

## Security Review 출력 형식

일반적인 Code Review와 같은 JSON 스키마를 사용하지만, `reviewer_profile: "security"`를 설정합니다.

```json
{
  "schema_version": "review-result.v1",
  "verdict": "APPROVE | REQUEST_CHANGES",
  "reviewer_profile": "security",
  "critical_issues": [
    {
      "severity": "critical",
      "category": "Security",
      "owasp": "A03:2021 - Injection",
      "location": "src/api/users.ts:42",
      "issue": "사용자 입력을 직접 SQL 문자열에 결합하고 있습니다",
      "suggestion": "프리페어드 스테이트먼트 또는 ORM을 사용하세요",
      "cwe": "CWE-89"
    }
  ],
  "major_issues": [],
  "observations": [],
  "recommendations": []
}
```

### Security 고유 필드

| 필드 | 설명 |
|----------|------|
| `owasp` | 해당하는 OWASP Top 10 카테고리（예: `A01:2021 - Broken Access Control`） |
| `cwe` | 해당하는 CWE 번호（예: `CWE-89`） |
| `cvss_estimate` | CVSS 스코어의概算（Critical: 9.0+, High: 7.0-8.9, Medium: 4.0-6.9） |

### Verdict 판단 기준（Security 모드）

Security 모드에서는 일반보다 엄격한 기준을 적용합니다.

| 중요도 | 정의 | verdict |
|--------|------|---------|
| **critical** | RCE, 인증 바이패스, 기밀 정보의 직접 노출, SQLi/CMDi | 1건이라도 REQUEST_CHANGES |
| **major** | 불충분한 승인 체크, 하드코딩된 기밀 정보, 취약한 암호화 | 1건이라도 REQUEST_CHANGES |
| **minor** | 보안 헤더 누락, 과도한 에러 정보, 경미한 설정 잘못 | APPROVE（修正推奨添え） |
| **recommendation** | 보안 베스트 프랙티스 제안 | APPROVE |
