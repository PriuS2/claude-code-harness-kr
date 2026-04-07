# Dual Review (--dual)

Claude Reviewer와 Codex Reviewer를 병렬 실행하여, 다른 모델 관점에서 리뷰 품질을 향상시킵니다.

## 전제 조건

- Codex CLI가 설치済み（`scripts/codex-companion.sh setup --json`로 확인）
- Codex가 利用不可인 경우, Claude 단독 리뷰로 폴백

## 실행 플로우

1. Codex의 利用可否를 확인

   ```bash
   CODEX_AVAILABLE="$(bash scripts/codex-companion.sh setup --json 2>/dev/null | jq -r '.ready // false')"
   ```

2. Claude Reviewer를 Task tool로起動（일반적인 review 플로우）

3. Codex가 利用가능하면 `scripts/codex-companion.sh review`를 병렬起動

   ```bash
   # BASE_REF가 전달된 경우는 --base를 지정. --json으로 구조화된 출력을 취득
   bash scripts/codex-companion.sh review --base "${BASE_REF:-HEAD~1}" --json
   ```

4. 양方の 결과를 대기

5. Verdict 머지 규칙（以下の順に 평가）:
   - 둘 다 APPROVE → `APPROVE`
   - 하나라도 REQUEST_CHANGES → `REQUEST_CHANGES`（엄격한方を採用）
   - `critical_issues`는 양方の 목록을 통합（중복 제거 없음）
   - `major_issues`는 양方の 목록을 통합（중복 제거 없음）
   - `recommendations`는 중복 제거하여 통합

## 출력 형식

일반적인 `review-result.v1` 스키마에 `dual_review` 필드를 추가합니다:

```json
{
  "schema_version": "review-result.v1",
  "verdict": "APPROVE | REQUEST_CHANGES",
  "dual_review": {
    "claude_verdict": "APPROVE | REQUEST_CHANGES",
    "codex_verdict": "APPROVE | REQUEST_CHANGES | unavailable | timeout",
    "merged_verdict": "APPROVE | REQUEST_CHANGES",
    "divergence_notes": "판단이 달랐을 경우의 이유. 예: Claude는 Performance에서 major 검출, Codex는 문제없음으로 판단"
  },
  "critical_issues": [],
  "major_issues": [],
  "observations": [],
  "recommendations": []
}
```

### `codex_verdict`의 특수 값

| 값 | 의미 |
|----|------|
| `"unavailable"` | Codex CLI가 설치되어 있지 않거나 利用不可 |
| `"timeout"` | Codex 리뷰가 타임아웃（120초 이내에 응답 없음） |

## 폴백

- **Codex가 利用不可**: Claude 단독으로 실행하고, `codex_verdict: "unavailable"`을 기록
- **Codex가 타임아웃**: Claude의 verdict를 그대로 채택하고, `codex_verdict: "timeout"`을 기록
- **Codex의 리뷰 출력이不正**: 파싱 실패로 취급하고, `codex_verdict: "unavailable"`을 기록

いずれのフォールバックでも、Claude 단독 리뷰의 결과가 최종 verdict가 됩니다.

## Divergence Notes의写法

판단이 일치한 경우（`claude_verdict == codex_verdict`）는 `divergence_notes`를 빈 문자열로 합니다.

판단이 달린 경우는以下の 형식으로 기록합니다:

```
Claude: REQUEST_CHANGES（Security - SQLイン젝션のリスク）
Codex: APPROVE（同箇所を問題なしと判定）
採用: REQUEST_CHANGES（엄격한方を 우선）
```
