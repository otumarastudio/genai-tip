# Claude Code MCP (Model Context Protocol) 완벽 가이드

> MCP를 사용하여 Claude Code를 외부 도구, 데이터베이스, API에 연결하기

---

## 한눈에 보기

### MCP란?

MCP(Model Context Protocol)는 AI 도구 통합을 위한 **오픈 소스 표준**입니다. MCP 서버를 통해 Claude Code에 외부 도구, 데이터베이스, API 액세스를 제공합니다.

### MCP로 할 수 있는 것

| 카테고리 | 예시 |
|----------|------|
| **이슈 추적기 연동** | "JIRA 이슈 ENG-4521에 설명된 기능을 구현하세요" |
| **모니터링 데이터** | "Sentry와 Statsig에서 기능 사용 현황을 확인하세요" |
| **데이터베이스 쿼리** | "PostgreSQL에서 기능을 사용한 사용자 10명을 찾으세요" |
| **디자인 통합** | "Slack의 새 Figma 디자인으로 템플릿을 업데이트하세요" |
| **워크플로우 자동화** | "이 사용자들에게 피드백 세션 초대 Gmail 초안을 생성하세요" |

### 전송 유형

| 유형 | 설명 | 사용 시나리오 |
|------|------|--------------|
| **HTTP** | 원격 서버 연결 (권장) | 클라우드 기반 서비스 |
| **SSE** | Server-Sent Events (레거시) | 이전 버전 호환 |
| **stdio** | 로컬 프로세스 실행 | 로컬 도구, 커스텀 스크립트 |

---

## 목차

1. [빠른 시작: MCP 서버 추가](#1-빠른-시작-mcp-서버-추가)
2. [MCP 서버 설치 방법](#2-mcp-서버-설치-방법)
3. [서버 관리](#3-서버-관리)
4. [설치 범위](#4-설치-범위)
5. [원격 서버 인증](#5-원격-서버-인증)
6. [MCP 리소스 사용](#6-mcp-리소스-사용)
7. [MCP 프롬프트를 슬래시 명령으로](#7-mcp-프롬프트를-슬래시-명령으로)
8. [실전 예제](#8-실전-예제)
9. [고급 설정](#9-고급-설정)
10. [문제 해결](#10-문제-해결)

---

## 1. 빠른 시작: MCP 서버 추가

### GitHub 연결 예제

```bash
# 1. GitHub MCP 서버 추가
claude mcp add --transport http github https://api.githubcopilot.com/mcp/

# 2. Claude Code에서 인증 (필요한 경우)
> /mcp
# GitHub에 대해 "인증" 선택

# 3. GitHub 작업 요청
> "PR #456을 검토하고 개선 사항을 제안하세요"
> "나에게 할당된 모든 열린 PR을 보여주세요"
```

### Sentry 연결 예제

```bash
# 1. Sentry MCP 서버 추가
claude mcp add --transport http sentry https://mcp.sentry.dev/mcp

# 2. 인증
> /mcp

# 3. 오류 분석
> "지난 24시간 동안 가장 일반적인 오류는 무엇입니까?"
```

---

## 2. MCP 서버 설치 방법

### 옵션 1: 원격 HTTP 서버 (권장)

```bash
# 기본 구문
claude mcp add --transport http <name> <url>

# 예: Notion 연결
claude mcp add --transport http notion https://mcp.notion.com/mcp

# Bearer 토큰 사용
claude mcp add --transport http secure-api https://api.example.com/mcp \
  --header "Authorization: Bearer your-token"
```

### 옵션 2: 원격 SSE 서버 (레거시)

```bash
# 기본 구문
claude mcp add --transport sse <name> <url>

# 예: Asana 연결
claude mcp add --transport sse asana https://mcp.asana.com/sse

# 인증 헤더 사용
claude mcp add --transport sse private-api https://api.company.com/sse \
  --header "X-API-Key: your-key-here"
```

> ⚠️ **참고**: SSE 전송은 더 이상 사용되지 않습니다. 가능하면 HTTP를 사용하세요.

### 옵션 3: 로컬 stdio 서버

```bash
# 기본 구문
claude mcp add [options] <name> -- <command> [args...]

# 예: Airtable 서버 추가
claude mcp add --transport stdio --env AIRTABLE_API_KEY=YOUR_KEY airtable \
  -- npx -y airtable-mcp-server

# 예: 환경 변수 포함
claude mcp add --transport stdio --env KEY=value myserver \
  -- python server.py --port 8080
```

> 💡 **중요**: 모든 옵션(`--transport`, `--env`, `--scope`, `--header`)은 서버 이름 **앞에** 와야 합니다. `--`는 서버 이름과 명령을 구분합니다.

---

## 3. 서버 관리

### 기본 관리 명령

```bash
# 구성된 모든 서버 나열
claude mcp list

# 특정 서버 세부 정보
claude mcp get github

# 서버 제거
claude mcp remove github

# (Claude Code 내에서) 서버 상태 확인
/mcp
```

### JSON 구성으로 추가

```bash
# HTTP 서버
claude mcp add-json weather-api '{"type":"http","url":"https://api.weather.com/mcp","headers":{"Authorization":"Bearer token"}}'

# stdio 서버
claude mcp add-json local-weather '{"type":"stdio","command":"/path/to/cli","args":["--api-key","abc123"]}'
```

### Claude Desktop에서 가져오기

```bash
# Claude Desktop 서버 가져오기
claude mcp add-from-claude-desktop

# 가져온 서버 확인
claude mcp list
```

---

## 4. 설치 범위

### 범위 유형

| 범위 | 저장 위치 | 용도 |
|------|----------|------|
| **local** (기본값) | `~/.claude.json` (프로젝트 경로 아래) | 개인 개발, 실험적 구성 |
| **project** | `.mcp.json` (프로젝트 루트) | 팀 공유, 버전 관리 |
| **user** | `~/.claude.json` | 모든 프로젝트에서 사용 |

### 범위 지정하여 추가

```bash
# 로컬 범위 (기본값)
claude mcp add --transport http stripe https://mcp.stripe.com

# 명시적 로컬 범위
claude mcp add --transport http stripe --scope local https://mcp.stripe.com

# 프로젝트 범위 (팀 공유)
claude mcp add --transport http paypal --scope project https://mcp.paypal.com/mcp

# 사용자 범위 (모든 프로젝트)
claude mcp add --transport http hubspot --scope user https://mcp.hubspot.com/anthropic
```

### `.mcp.json` 파일 형식

```json
{
  "mcpServers": {
    "shared-server": {
      "command": "/path/to/server",
      "args": [],
      "env": {}
    }
  }
}
```

### 환경 변수 확장

`.mcp.json`에서 환경 변수를 사용할 수 있습니다:

```json
{
  "mcpServers": {
    "api-server": {
      "type": "http",
      "url": "${API_BASE_URL:-https://api.example.com}/mcp",
      "headers": {
        "Authorization": "Bearer ${API_KEY}"
      }
    }
  }
}
```

지원 구문:
- `${VAR}` - 환경 변수 값으로 확장
- `${VAR:-default}` - 설정되지 않으면 기본값 사용

---

## 5. 원격 서버 인증

### OAuth 2.0 인증

```bash
# 1. 인증이 필요한 서버 추가
claude mcp add --transport http sentry https://mcp.sentry.dev/mcp

# 2. Claude Code에서 /mcp 명령 실행
> /mcp

# 3. 브라우저에서 로그인 단계 완료
```

### 인증 관리

- 토큰은 안전하게 저장되고 자동 갱신
- `/mcp` 메뉴에서 "Clear authentication"으로 액세스 취소
- 브라우저가 자동으로 열리지 않으면 제공된 URL 복사

---

## 6. MCP 리소스 사용

MCP 서버는 `@` 멘션으로 참조할 수 있는 리소스를 노출합니다.

### 리소스 참조

```bash
# 사용 가능한 리소스 확인 (@ 입력)
> @

# 특정 리소스 참조
> @github:issue://123을 분석하고 수정 사항을 제안할 수 있나요?

# 여러 리소스 참조
> @postgres:schema://users와 @docs:file://database/user-model을 비교하세요
```

### 리소스 형식

```
@server:protocol://resource/path
```

---

## 7. MCP 프롬프트를 슬래시 명령으로

MCP 서버의 프롬프트는 슬래시 명령으로 사용 가능합니다.

### 프롬프트 실행

```bash
# 사용 가능한 명령 확인 (/ 입력)
> /

# 인수 없이 실행
> /mcp__github__list_prs

# 인수와 함께 실행
> /mcp__github__pr_review 456
> /mcp__jira__create_issue "로그인 버그" high
```

---

## 8. 실전 예제

### 예제 1: GitHub 코드 리뷰

```bash
# 1. GitHub MCP 서버 추가
claude mcp add --transport http github https://api.githubcopilot.com/mcp/

# 2. 인증
> /mcp

# 3. PR 리뷰
> "PR #456을 검토하고 개선 사항을 제안하세요"
> "방금 발견한 버그에 대한 새 이슈를 생성하세요"
```

### 예제 2: Sentry 오류 모니터링

```bash
# 1. Sentry MCP 서버 추가
claude mcp add --transport http sentry https://mcp.sentry.dev/mcp

# 2. 인증
> /mcp

# 3. 오류 디버깅
> "지난 24시간 동안 가장 일반적인 오류는 무엇입니까?"
> "오류 ID abc123의 스택 추적을 보여주세요"
```

### 예제 3: PostgreSQL 데이터베이스 쿼리

```bash
# 1. 데이터베이스 서버 추가
claude mcp add --transport stdio db -- npx -y @bytebase/dbhub \
  --dsn "postgresql://readonly:pass@prod.db.com:5432/analytics"

# 2. 자연어로 쿼리
> "이번 달 총 수익은 얼마입니까?"
> "주문 테이블의 스키마를 보여주세요"
> "지난 90일 동안 구매하지 않은 고객을 찾으세요"
```

### 예제 4: Claude Code를 MCP 서버로 사용

```bash
# Claude를 stdio MCP 서버로 시작
claude mcp serve
```

Claude Desktop에 추가:

```json
{
  "mcpServers": {
    "claude-code": {
      "type": "stdio",
      "command": "claude",
      "args": ["mcp", "serve"],
      "env": {}
    }
  }
}
```

---

## 9. 고급 설정

### 출력 토큰 제한 조정

```bash
# 기본 최대값: 25,000 토큰
# 큰 출력이 필요할 때 제한 증가
export MAX_MCP_OUTPUT_TOKENS=50000
claude
```

### 서버 시작 시간 초과 설정

```bash
# 10초 시간 초과 설정
MCP_TIMEOUT=10000 claude
```

### Windows에서 npx 사용

Windows에서는 `cmd /c` 래퍼가 필요합니다:

```bash
claude mcp add --transport stdio my-server -- cmd /c npx -y @some/package
```

### 플러그인 MCP 서버

플러그인이 MCP 서버를 번들로 제공할 수 있습니다.

플러그인 루트의 `.mcp.json`:

```json
{
  "database-tools": {
    "command": "${CLAUDE_PLUGIN_ROOT}/servers/db-server",
    "args": ["--config", "${CLAUDE_PLUGIN_ROOT}/config.json"],
    "env": {
      "DB_URL": "${DB_URL}"
    }
  }
}
```

또는 `plugin.json`에 인라인:

```json
{
  "name": "my-plugin",
  "mcpServers": {
    "plugin-api": {
      "command": "${CLAUDE_PLUGIN_ROOT}/servers/api-server",
      "args": ["--port", "8080"]
    }
  }
}
```

---

## 10. 문제 해결

### 일반적인 문제

| 문제 | 해결 방법 |
|------|----------|
| 연결 실패 | URL 확인, 네트워크 연결 확인 |
| 인증 오류 | `/mcp`에서 재인증 |
| 출력 경고 | `MAX_MCP_OUTPUT_TOKENS` 환경 변수 조정 |
| Windows npx 오류 | `cmd /c` 래퍼 사용 |

### 프로젝트 승인 재설정

```bash
claude mcp reset-project-choices
```

### 서버 상태 확인

```bash
# CLI에서
claude mcp list

# Claude Code 내에서
/mcp
```

---

## 인기 있는 MCP 서버

### 외부 통합

| 플러그인 | 용도 |
|----------|------|
| `github` | GitHub 이슈, PR, 저장소 작업 |
| `gitlab` | GitLab 연동 |
| `atlassian` | Jira/Confluence 연동 |
| `asana` | Asana 프로젝트 관리 |
| `linear` | Linear 이슈 트래킹 |
| `notion` | Notion 문서 연동 |
| `figma` | Figma 디자인 연동 |
| `vercel` | Vercel 배포 |
| `firebase` | Firebase 프로젝트 |
| `supabase` | Supabase 연동 |
| `slack` | Slack 메시징 |
| `sentry` | Sentry 오류 모니터링 |

> ⚠️ **주의**: 타사 MCP 서버는 자신의 책임 하에 사용하세요. Anthropic은 보안을 검증하지 않았습니다.

---

## 빠른 참조

### 자주 사용하는 명령

```bash
# 서버 추가
claude mcp add --transport http <name> <url>
claude mcp add --transport stdio <name> -- <command>

# 서버 관리
claude mcp list                    # 모든 서버 나열
claude mcp get <name>              # 서버 세부 정보
claude mcp remove <name>           # 서버 제거

# JSON으로 추가
claude mcp add-json <name> '<json>'

# Claude Desktop에서 가져오기
claude mcp add-from-claude-desktop

# Claude Code 내에서
/mcp                               # 서버 상태 및 인증
```

### 범위 옵션

```bash
--scope local    # 현재 프로젝트에서만 나만 사용 (기본값)
--scope project  # 팀과 공유 (.mcp.json)
--scope user     # 모든 프로젝트에서 사용
```

---

## 관련 문서

- [플러그인](/plugins) - 플러그인에서 MCP 서버 번들링
- [플러그인 참조](/plugins-reference#mcp-servers) - MCP 서버 구성 상세
- [MCP 공식 문서](https://modelcontextprotocol.io/introduction) - Model Context Protocol 상세
- [MCP 서버 목록](https://github.com/modelcontextprotocol/servers) - GitHub에서 더 많은 서버 찾기

---

*마지막 업데이트: 2026-01*
