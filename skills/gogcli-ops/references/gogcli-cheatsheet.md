# Gogcli quick commands (v0.9.0)

## 인증과 계정
- 저장된 계정 목록:
  - `gog auth list`
- 인증 상태와 키링 백엔드 표시:
  - `gog auth status`
- 계정 추가/허가:
  - `gog auth add <email>`
- 어느 커맨드든 특정 계정 사용:
  - `gog <area> <cmd> --account <email>`

## 출력 모드
- 기계 판독 가능한 TSV:
  - `--plain`
- JSON 출력:
  - `--json`
- 논인터랙티브:
  - `--no-input` (프롬프트 대신 실패)

## Drive
- 폴더内の 파일 목록 (기본: root):
  - `gog drive ls`
- 검색:
  - `gog drive search "<query>"`
- 메타데이터:
  - `gog drive get <fileId>`
- 다운로드 (Google Docs 포맷으로エクスポート):
  - `gog drive download <fileId>`
- 권한:
  - `gog drive permissions <fileId>`

## Sheets
- 스프레드시트 메타데이터:
  - `gog sheets metadata <spreadsheetId>`
- 범위 읽기:
  - `gog sheets get <spreadsheetId> <range>`
- エクスポート:
  - `gog sheets export <spreadsheetId>`

## Docs
- 메타데이터:
  - `gog docs info <docId>`
- エクスポート:
  - `gog docs export <docId>`
- 플레인 텍스트:
  - `gog docs cat <docId>`

## Slides
- 메타데이터:
  - `gog slides info <presentationId>`
- エクスポート:
  - `gog slides export <presentationId>`

## URL 파싱 헬퍼 (이 스킬)
- 타입과 ID 추출:
  - `python3 scripts/gog_parse_url.py "<url-or-id>"`
  - 출력 포맷: `<type>\t<id>` where type is `sheet|doc|slide|file|folder|id|unknown`
