---
name: payments
description: "결제 기능의 실장（Stripe）. 구독이나 구매 결제을 추가하고 싶은 경우 사용합니다."
allowed-tools: ["Read", "Write", "Edit", "Bash"]
---

# Payments Skill

Stripe를 사용한 결제 기능을 구현하는 스킬.

---

##トリガーフレーズ

- 「결제를 붙이고 싶다」
- 「Stripe를 도입해줘」
- 「구독을 구현해줘」
- 「구매 결제를 추가해줘」

---

## 기능

- 구독（월액/연액）
- 일회 결제
- Webhook（결제 완료 알림）
- 고객 포털（플랜 변경, 취소）

---

## 실행 흐름

1. 프로젝트 구성을 확인
2. 구독 or 일회 결제를 선택
3. Stripe SDK를 설치
4. 결제 페이지를 작성
5. Webhook 엔드포인트를 설정
6. 환경 변수의 설정을 가이드
