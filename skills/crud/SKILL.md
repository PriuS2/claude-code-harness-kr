---
name: crud
description: "CRUD를 재빨리 자동생성. 보일러플레이트는 AI에게 맡기세요. Use when user mentions CRUD, entity generation, or wants to create API endpoints. Do NOT load for: UI component creation, form design, database schema discussions."
description-en: "Auto-generate CRUD quickly. Boilerplate left to AI. Use when user mentions CRUD, entity generation, or wants to create API endpoints. Do NOT load for: UI component creation, form design, database schema discussions."
description-ja: "CRUD를 재빨리 자동생성. 보일러플레이트는 AI에게 맡기세요. Use when user mentions CRUD, entity generation, or wants to create API endpoints. Do NOT load for: UI component creation, form design, database schema discussions."
allowed-tools: ["Read", "Write", "Edit", "Bash", "Grep", "Glob"]
argument-hint: "<entity-name>"
user-invocable: false
---

# CRUD Skill

지정된 엔티티(테이블)에 대해 **production-ready 수준**의 CRUD 기능을 자동 생성합니다.

## Quick Reference

- "**task 관리를 위한 CRUD 생성해줘**" → `/crud tasks`
- "**검색과 페이징도 추가해줘**" → 함께 포함
- "**권한 포함해줘 (누가 보기/수정 가능)**" → authorization/rules 함께 설정

## Deliverables

- CRUD + validation + authorization + tests, **완전한 production-safe 세트**
- 기존 DB/code와 최소한의 diff 유지

**기능**:
- Validation (Zod) 자동 추가
- Auth/authorization (Row Level Security) 자동 설정
- Relations (one-to-many, many-to-many) 지원
- Pagination, search, filters
- 자동 생성된 테스트 케이스

---

## Auto-invoke Skills

**이 스킬은 다음 스킬을 반드시 Skill tool로 호출해야 합니다**:

| Skill | Purpose | When to Call |
|-------|---------|--------------|
| `impl` | Implementation (parent skill) | CRUD feature implementation |
| `verify` | Verification (parent skill) | Post-implementation verification |

---

## Execution Flow

상세 단계는 아래 phases를 참조하세요.

### Phase 1: Entity Analysis

1. $ARGUMENTS에서 entity name 파싱
2. 기존 schema 감지 (Prisma, Drizzle, raw SQL)
3. 필드 타입과 relations 추론

### Phase 2: CRUD Generation

1. 필요시 model/schema 생성
2. API endpoints 생성 (REST or tRPC)
3. validation schemas 추가 (Zod)
4. authorization rules 설정

### Phase 3: Test Generation

1. 각 endpoint에 대한 unit tests 생성
2. integration tests 추가
3. test fixtures 생성

### Phase 4: Verification

1. type check 실행
2. tests 실행
3. build 검증

---

## Supported Frameworks

| Framework | Detection | Generated Files |
|-----------|-----------|-----------------|
| **Next.js + Prisma** | `prisma/schema.prisma` | API routes, Prisma client |
| **Next.js + Drizzle** | `drizzle.config.ts` | API routes, Drizzle queries |
| **Express** | `express` in package.json | Controllers, routes |
| **Hono** | `hono` in package.json | Route handlers |

---

## Output Structure

```
src/
├── lib/
│   └── validations/
│       └── {entity}.ts        # Zod schemas
├── app/api/{entity}/
│   ├── route.ts              # GET (list), POST (create)
│   └── [id]/
│       └── route.ts          # GET, PUT, DELETE
└── tests/
    └── {entity}.test.ts      # Test cases
```

---

## Related Skills

- `impl` - Feature implementation
- `verify` - Build verification
- `auth` - Authentication/authorization
