---
name: auth
description: "인증 기능의 실장（Clerk / Supabase Auth 等）. 로그인 기능을 추가하고 싶은 경우 사용합니다."
allowed-tools: ["Read", "Write", "Edit", "Bash"]
---

# Auth Skill

Clerk 또는 Supabase Auth를 사용한 인증 기능을 구현하는 스킬.

---

##トリガーフレーズ

- 「로그인 기능을 붙여줘」
- 「인증을 추가해줘」
- 「Clerk로 인증을 구현해줘」
- 「Supabase Auth를 설정해줘」
- 「Google 로그인을 추가해줘」

---

## 기능

- 사인업/로그인
- 소셜 로그인（Google, GitHub）
- 메일 인증
- 비밀번호 리셋
- 사용자 프로필 관리

---

## 실행 흐름

1. 프로젝트 구성을 확인
2. Clerk 또는 Supabase Auth를 선택
3. 필요한 패키지를 설치
4. 인증 설정 파일을 생성
5. 로그인/사인업 UI를 작성
6. 미들웨어/보호 라우트를 설정
