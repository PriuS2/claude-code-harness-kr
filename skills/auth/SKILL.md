---
name: auth
description: "인증과 결제 기능을 구현합니다. Clerk, Supabase Auth, Stripeに対応。Use when user mentions login, authentication, payments, subscriptions, or Stripe. Do NOT load for: general UI work, database design, or non-auth features."
description-en: "Implements authentication and payment features using Clerk, Supabase Auth, or Stripe. Use when user mentions login, authentication, payments, subscriptions, or Stripe. Do NOT load for: general UI work, database design, or non-auth features."
description-ja: "인증과 결제 기능을 구현합니다. Clerk, Supabase Auth, Stripeに対応。Use when user mentions login, authentication, payments, subscriptions, or Stripe. Do NOT load for: general UI work, database design, or non-auth features."
allowed-tools: ["Read", "Write", "Edit", "Bash"]
user-invocable: false
---

# Auth Skills

인증과 결제 기능의 실장을 담당하는 스킬 그룹입니다.

## 기능 상세

| 기능 | 상세 |
|------|------|
| **인증 기능** | See [references/authentication.md](${CLAUDE_SKILL_DIR}/references/authentication.md) |
| **결제 기능** | See [references/payments.md](${CLAUDE_SKILL_DIR}/references/payments.md) |

## 실행 절차

1. **품질 판단 게이트** (Step 0)
2. 사용자의 요청을 분류 (인증 or 결제)
3. 위의 "기능 상세"에서 적절한 참조 파일을 읽음
4. 그 내용에 따라 실장

### Step 0: 품질 판단 게이트 (보안 체크리스트)

인증・결제 기능은 항상 보안 위험이 높으므로, 작업 시작전에 반드시以下を표시:

```markdown
🔐 보안 체크리스트

이 작업은 보안상 중요합니다. 이하를 확인해주세요:

### 인증 관련
- [ ] 비밀번호는 해시화 (bcrypt/argon2)
- [ ] 세션 관리는 안전한지 (HTTPOnly Cookie)
- [ ] CSRF 대응은 구현되어 있는지
- [ ] 레이트 리밋 (브루트포스 방지)

### 결제 관련
- [ ] 기밀 정보 (카드 번호 등) 를 서버에 저장하지 않음
- [ ] Stripe/결제 프로바이더의 SDK를 올바르게 사용
- [ ] Webhook의 서명 검증
- [ ] 금액 변조 방지 (서버 측에서 금액을 확정)

### 공통
- [ ] 에러 메시지가 너무 자세하지 않은지 (정보 유출 방지)
- [ ] 로그에 기밀 정보를 출력하지 않는지
```

### 보안 중요도 표시

```markdown
⚠️ 주의 레벨: 🔴 高

이 기능は以下のリスクがあります：
- 인증 정보의 유출
- 부정한 접근
- 결제의 부정한 조작

전문가による 검토를 권장합니다.
```

### VibeCoder 向け

```markdown
🔐安全にログイン・決済機能を作るために

1. **パスワードは「ハッシュ化」する**
   - 元のパスワードを復元できない形で保存
   - 万が一データが漏れても安全

2. **カード情報はサーバーに保存しない**
   - Stripe などの専用サービスに任せる
   - 自分のサーバーには一切保存しない

3. **エラーメッセージは曖昧に**
   - 「パスワードが違います」ではなく「認証に失敗しました」
   - 悪意ある人にヒントを与えない
```
