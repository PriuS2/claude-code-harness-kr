---
name: merge-plans
description: "Plans.md의 머지 업데이트를 수행하는 스킬（사용자 작업 유지）.複数のPlans.mdを統合する必要がある場合に使用します."
allowed-tools: ["Read", "Write", "Edit"]
---

# Merge Plans Skill

기존의 Plans.md를 업데이트할 때, 사용자 작업 데이터를 유지하면서
템플릿의 구조를 적용하는 스킬입니다.

---

## 목적

- 사용자의 작업（🔴🟡🟢📦섹션）을 유지
- 템플릿의 구조·마커 정의를 업데이트
- 최종 업데이트 정보를 업데이트

---

## Plans.md의 구조

```markdown
# Plans.md - 작업 관리

> **프로젝트**: {{PROJECT_NAME}}
> **최종 업데이트**: {{DATE}}
> **업데이트자**: Claude Code

---

## 🔴 진행 중인 작업        ← 사용자 데이터（유지）

## 🟡 미착수 작업        ← 사용자 데이터（유지）

## 🟢 완료 작업            ← 사용자 데이터（유지）

## 📦 아카이브            ← 사용자 데이터（유지）

## 마커 凡例             ← 템플릿에서 업데이트

## 최종 업데이트 정보             ← 날짜를 업데이트
```

---

## 머지 알고리즘

### Step 1: 섹션 분할

```
기존 Plans.md를 다음 섹션으로 분할:

1. 헤더 부분（# Plans.md ... ---）
2. 🔴 진행 중인 작업（다음 섹션까지）
3. 🟡 미착수 작업（다음 섹션까지）
4. 🟢 완료 작업（다음 섹션까지）
5. 📦 아카이브（다음 섹션까지）
6. 마커 凡例（다음 섹션까지）
7. 최종 업데이트 정보（파일 끝까지）
```

### Step 2: 작업 섹션의 추출

```bash
extract_section() {
  local file="$1"
  local start_marker="$2"
  local end_markers="$3"  # 파이프 区切りの終了マーカー

  awk -v start="$start_marker" -v ends="$end_markers" '
    BEGIN { in_section = 0; split(ends, end_arr, "|") }
    $0 ~ start { in_section = 1; next }
    in_section {
      for (i in end_arr) {
        if ($0 ~ end_arr[i]) { in_section = 0; exit }
      }
      if (in_section) print
    }
  ' "$file"
}

# 각 섹션을 추출
TASKS_WIP=$(extract_section "$PLANS_FILE" "## 🔴" "## 🟡|## 🟢|## 📦|## 마커|---")
TASKS_TODO=$(extract_section "$PLANS_FILE" "## 🟡" "## 🔴|## 🟢|## 📦|## 마커|---")
TASKS_DONE=$(extract_section "$PLANS_FILE" "## 🟢" "## 🔴|## 🟡|## 📦|## 마커|---")
TASKS_ARCHIVE=$(extract_section "$PLANS_FILE" "## 📦" "## 🔴|## 🟡|## 🟢|## 마커|---")
```

### Step 3: 작업의 검증

```bash
# 空でないことを確認
count_tasks() {
  echo "$1" | grep -c "^\s*- \[" || echo "0"
}

WIP_COUNT=$(count_tasks "$TASKS_WIP")
TODO_COUNT=$(count_tasks "$TASKS_TODO")
DONE_COUNT=$(count_tasks "$TASKS_DONE")
ARCHIVE_COUNT=$(count_tasks "$TASKS_ARCHIVE")

echo "유지되는 작업:"
echo "  진행중: $WIP_COUNT"
echo "  미착수: $TODO_COUNT"
echo "  완료: $DONE_COUNT"
echo "  아카이브: $ARCHIVE_COUNT"
```

### Step 4: 새 Plans.md의 생성

```markdown
# Plans.md - 작업 관리

> **프로젝트**: {{PROJECT_NAME}}
> **최종 업데이트**: {{DATE}}
> **업데이트자**: Claude Code

---

## 🔴 진행 중인 작업

<!-- cc:WIP의 작업을 여기에 기재 -->

{{TASKS_WIP}}

---

## 🟡 미착수 작업

<!-- cc:TODO, pm:依頼中（호환: cursor:依頼中） 의 작업을 여기에 기재 -->

{{TASKS_TODO}}

---

## 🟢 완료 작업

<!-- cc:완료, pm:확인済（호환: cursor:확인済） 의 작업을 여기에 기재 -->

{{TASKS_DONE}}

---

## 📦 아카이브

<!--古い完了 작업はここに移動 -->

{{TASKS_ARCHIVE}}

---

## 마커 凡例

| 마커 | 의미 |
|---------|-------|
| `pm:依頼中` | PM으로부터 의뢰된 작업（호환: cursor:依頼中） |
| `cc:TODO` | Claude Code 미착수 |
| `cc:WIP` | Claude Code 작업중 |
| `cc:완료` | Claude Code 완료（확인 대기） |
| `pm:확인済` | PM 확인 완료（호환: cursor:확인済） |
| `cursor:依頼中` | （호환）pm:依頼中와 同義 |
| `cursor:確認済` | （호환）pm:확인済와 同義 |
| `blocked` | 블록중（사유를 並記） |

---

## 최종 업데이트 정보

- **更新日時**: {{DATE}}
- **最終セッション担当**: Claude Code
- **브랜치**: main
- **更新種別**: 플러그인 업데이트
```

---

## 빈 섹션의 처리

작업이 빈 경우, 기본 텍스트를 삽입합니다:

```markdown
## 🔴 진행 중인 작업

<!-- cc:WIP의 작업을 여기에 기재 -->

（현재 없음）
```

---

## 에러 처리

### Plans.md가 파싱 불가능한 경우

```bash
if ! validate_plans_structure "$PLANS_FILE"; then
  echo "⚠️ Plans.md의 구조를 파싱할 수 없었습니다"
  echo "백업을 유지하고, 새 템플릿을 사용합니다"

  # 백업
  cp "$PLANS_FILE" "${PLANS_FILE}.bak.$(date +%Y%m%d%H%M%S)"

  # 템플릿 사용
  use_template_instead=true
fi
```

### 필수 섹션이 없는 경우

부족한 섹션은 템플릿의 기본값으로 보충합니다.

---

## 출력

| 항목 | 설명 |
|------|------|
| `merge_successful` | 머지 성공 플래그 |
| `tasks_wip_count` | 진행중 작업 수 |
| `tasks_todo_count` | 미착수 작업 수 |
| `tasks_done_count` | 완료 작업 수 |
| `tasks_archive_count` | 아카이브 작업 수 |
| `backup_created` | 백업 생성 유무 |

---

## 사용 예

```bash
# 스킬의 호출
merge_plans \
  --existing "./Plans.md" \
  --template "$PLUGIN_PATH/templates/Plans.md.template" \
  --output "./Plans.md" \
  --project-name "my-project" \
  --date "$(date +%Y-%m-%d)"
```

---

## 관련 스킬

- `update-2agent-files` - 업데이트 플로우 전체
- `generate-workflow-files` - 신규 생성
