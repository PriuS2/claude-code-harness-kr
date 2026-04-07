---
name: gogcli-ops
description: "gogcli로 Google Workspace 조작（Drive/Sheets/Docs/Slides）. 사용자가 Google 파일의 확인・검색・エクスポート・읽기・更新을 gogcli로依頼할 때 사용. Trigger when a user asks to check, list, search, export, read, or update Google files via gogcli; when a Google URL/ID needs parsing; when auth/account selection or safe read-only workflows are needed; or when troubleshooting gogcli access/errors. Do NOT load for: general file operations, non-Google cloud storage, or standard shell commands."
description-en: "Use gogcli for Google Workspace CLI operations (Drive/Sheets/Docs/Slides). Trigger when a user asks to check, list, search, export, read, or update Google files via gogcli; when a Google URL/ID needs parsing; when auth/account selection or safe read-only workflows are needed; or when troubleshooting gogcli access/errors. Do NOT load for: general file operations, non-Google cloud storage, or standard shell commands."
description-ja: "gogcli로 Google Workspace 조작（Drive/Sheets/Docs/Slides）. 사용자가 Google 파일의 확인・검색・エクスポート・읽기・更新을 gogcli로依頼할 때 사용. Trigger when a user asks to check, list, search, export, read, or update Google files via gogcli; when a Google URL/ID needs parsing; when auth/account selection or safe read-only workflows are needed; or when troubleshooting gogcli access/errors. Do NOT load for: general file operations, non-Google cloud storage, or standard shell commands."
allowed-tools: ["Read", "Bash", "Grep", "Glob"]
---

# Gogcli Ops

## 개요
gogcli 사용의 표준화: 인증 확인, URL에서 ID 해결, 읽기 전용 체크가 기본, 그 후 필요한 최소 커맨드만 실행.

## 빠른 시작
- gogcli 이용 가능 확인: `gog --version`
- 저장된 계정 목록 및 다수인 경우 명시적으로 선택: `gog auth list`
- URL을 ID로 해결: `python3 scripts/gog_parse_url.py "<url-or-id>"`
- 먼저 읽기 전용 메타데이터 커맨드 실행 (Drive/Sheets/Docs/Slides)

## 워크플로 결정 트리
1. 대상 타입 특정: `sheet | doc | slide | file | folder | id | unknown` via `scripts/gog_parse_url.py`.
2. 액세스 확인을 위해 가장 작은 읽기 전용 커맨드를 선택:
   - Sheets: `gog sheets metadata <spreadsheetId>`
   - Docs: `gog docs info <docId>`
   - Slides: `gog slides info <presentationId>`
   - Drive file/folder: `gog drive get <fileId>` 또는 `gog drive permissions <fileId>`
3. 명시적인 사용자 확인 후에만 쓰기 작업으로 진행 (update/append/move/share/delete).

## 핵심 태스크

### 인증과 계정 선택
- 저장된 계정 표시: `gog auth list`
- 인증 설정 표시: `gog auth status`
- 계정 추가/허가: `gog auth add <email>`
- 다수 계정이 있는 경우 항상 `--account <email>` 사용.

### URL에서 ID 해결
- URL 또는 ID 파싱:
  - `python3 scripts/gog_parse_url.py "<url-or-id>"`
- 출력 타입이 `unknown`인 경우, 직접 ID 또는 다른 URL을 요청.

### Drive (files/folders)
- 폴더 목록 (기본: root): `gog drive ls`
- 쿼리로 검색: `gog drive search "<query>"`
- 메타데이터: `gog drive get <fileId>`
- 다운로드/エクスポート: `gog drive download <fileId>`
- 권한 확인: `gog drive permissions <fileId>`

### Sheets
- 스프레드시트 메타데이터: `gog sheets metadata <spreadsheetId>`
- 값 읽기: `gog sheets get <spreadsheetId> <range>`
- エクスポート: `gog sheets export <spreadsheetId>`
- 쓰기 작업 (update/append/clear/format): 명시적인 확인과 정확한 range가 필요.

### Docs
- 메타데이터: `gog docs info <docId>`
- 텍스트 읽기: `gog docs cat <docId>`
- エクスポート: `gog docs export <docId>`

### Slides
- 메타데이터: `gog slides info <presentationId>`
- エクスポート: `gog slides export <presentationId>`

### 출력 모드
- 안정적인 TSV 출력을 위해 `--plain` 사용.
- 호출자가 구조화된 출력을 원하는 경우 `--json` 사용.
- non-interactive 흐름에서 멈추지 않으려면 `--no-input` 사용.

## 에러 처리
- 403/404: 계정 확인 (`gog auth list`), 권한 확인 (`gog drive permissions <fileId>`), ID 확인.
- 액세스가 실패한 경우, 사용자에게 선택한 계정에 파일을 공유하거나 올바른 계정을 제공하도록 요청.

## 리소스
- 압축된 커맨드 목록은 `${CLAUDE_SKILL_DIR}/references/gogcli-cheatsheet.md` 참조.
- 커맨드 실행 전에 `scripts/gog_parse_url.py`를 사용하여 URL을 ID로 정규화.
