---
name: deploy
description: "Vercel이나 Netlify로 출정합니다. 프로덕션 환경への 片道切符을 준비합니다. Use when user mentions deployment, Vercel, Netlify, analytics, or health checks. Do NOT load for: implementation work, local development, reviews, or setup."
description-en: "Deploy to Vercel/Netlify. One-way ticket to production arranged. Use when user mentions deployment, Vercel, Netlify, analytics, or health checks. Do NOT load for: implementation work, local development, reviews, or setup."
description-ja: "Vercel이나 Netlify로 출정합니다. 프로덕션 환경への 片道切符을 준비합니다. Use when user mentions deployment, Vercel, Netlify, analytics, or health checks. Do NOT load for: implementation work, local development, reviews, or setup."
allowed-tools: ["Read", "Write", "Edit", "Bash"]
disable-model-invocation: true
argument-hint: "[vercel|netlify|health]"
context: fork
---

# Deploy Skills

배포와 모니터링의 설정을 담당하는 스킬 그룹입니다.

## 기능 상세

| 기능 | 상세 |
|------|------|
| **배포 설정** | See [references/deployment-setup.md](${CLAUDE_SKILL_DIR}/references/deployment-setup.md) |
| **아날리틱스** | See [references/analytics.md](${CLAUDE_SKILL_DIR}/references/analytics.md) |
| **환경 진단** | See [references/health-checking.md](${CLAUDE_SKILL_DIR}/references/health-checking.md) |

## 실행 절차

1. 사용자의 요청을 분류
2. 위의 "기능 상세"에서 적절한 참조 파일을 읽음
3. 그 내용에 따라 설정
