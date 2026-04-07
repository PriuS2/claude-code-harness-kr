---
name: health-check
description: "환경 진단（의존/설정/이용 가능 기능의 확인）. 환경이 올바르게 설정되어 있는지 확인하고 싶은 경우 사용합니다."
allowed-tools: ["Read", "Bash"]
---

# Health Check Skill

플러그인을 사용하기 전에, 환경이 올바르게 설정되어 있는지를 진단하는 스킬.

---

##トリガーフレーズ

- 「이 환경에서 작동하는지 체크해줘」
- 「뭐가 빠져 있어?」
- 「환경을 진단해줘」
- 「사용 가능한 기능을 알려줘」

---

## 체크 항목

### 필수 도구
- Git
- Node.js / npm（해당하는 경우）
- GitHub CLI（옵션）

### 설정 파일
- `claude-code-harness.config.json` 의 존재와 타당성
- `.claude/settings.json` 의 존재

### 워크플로 파일
- `Plans.md` 의 존재
- `AGENTS.md` 의 존재
- `CLAUDE.md` 의 존재

---

## 출력 형식

```
## 환경 진단 리포트

### 필수 도구
✅ git (2.40.0)
✅ node (v20.10.0)
⚠️ gh (미설치 - CI자동 수정이 필요)

### 설정 파일
✅ claude-code-harness.config.json
✅ .claude/settings.json

### 이용 가능한 기능
✅ /work, /plan-with-agent, /sync-status
⚠️ CI자동 수정 (gh 필요)
```
