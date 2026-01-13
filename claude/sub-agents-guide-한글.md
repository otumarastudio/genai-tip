# Claude Code 서브에이전트 완벽 가이드

> 작업별 워크플로우와 컨텍스트 관리를 위한 특화된 AI 서브에이전트 활용법

---

## 한눈에 보기

### 서브에이전트란?

서브에이전트는 특정 유형의 작업을 처리하는 **특화된 AI 어시스턴트**입니다. 각 서브에이전트는:
- 자신만의 컨텍스트 윈도우에서 실행
- 사용자 정의 시스템 프롬프트 보유
- 특정 도구에만 액세스 가능
- 독립적인 권한 설정 가능

### 서브에이전트의 장점

| 장점 | 설명 |
|------|------|
| **컨텍스트 보존** | 탐색과 구현을 주 대화에서 분리하여 관리 |
| **제약 조건 적용** | 서브에이전트가 사용할 수 있는 도구 제한 |
| **구성 재사용** | 사용자 수준 서브에이전트로 프로젝트 간 재사용 |
| **동작 특화** | 특정 도메인을 위한 집중된 시스템 프롬프트 |
| **비용 제어** | Haiku 같은 빠르고 저렴한 모델로 작업 라우팅 |

### 기본 제공 서브에이전트

| 에이전트 | 모델 | 도구 | 용도 |
|----------|------|------|------|
| **Explore** | Haiku | 읽기 전용 | 코드베이스 검색 및 분석 |
| **Plan** | 상속 | 읽기 전용 | 계획 모드에서 컨텍스트 수집 |
| **general-purpose** | 상속 | 모든 도구 | 복잡한 다단계 작업 |
| **Bash** | 상속 | Bash | 터미널 명령 실행 |
| **Claude Code Guide** | Haiku | 검색/읽기 | Claude Code 기능 질문 응답 |

---

## 목차

1. [빠른 시작: 첫 번째 서브에이전트 만들기](#1-빠른-시작-첫-번째-서브에이전트-만들기)
2. [서브에이전트 구성 방법](#2-서브에이전트-구성-방법)
3. [서브에이전트 파일 작성](#3-서브에이전트-파일-작성)
4. [기능 제어: 도구와 권한](#4-기능-제어-도구와-권한)
5. [서브에이전트 훅 정의](#5-서브에이전트-훅-정의)
6. [서브에이전트 작업 패턴](#6-서브에이전트-작업-패턴)
7. [실전 예제](#7-실전-예제)
8. [문제 해결 및 팁](#8-문제-해결-및-팁)

---

## 1. 빠른 시작: 첫 번째 서브에이전트 만들기

코드를 검토하고 개선 사항을 제안하는 서브에이전트를 만들어봅니다.

### 단계 1: 서브에이전트 인터페이스 열기

```
/agents
```

### 단계 2: 새 사용자 수준 에이전트 만들기

**Create new agent** → **User-level** 선택

> 💡 사용자 수준으로 저장하면 `~/.claude/agents/`에 저장되어 모든 프로젝트에서 사용 가능합니다.

### 단계 3: Claude로 생성

**Generate with Claude** 선택 후 설명 입력:

```
파일을 스캔하고 가독성, 성능 및 모범 사례에 대한 개선 사항을 제안하는
코드 개선 에이전트입니다. 각 문제를 설명하고, 현재 코드를 표시하고,
개선된 버전을 제공해야 합니다.
```

### 단계 4: 도구 선택

읽기 전용 검토자의 경우 **Read-only tools**만 선택합니다.

### 단계 5: 모델 선택

코드 패턴 분석을 위해 **Sonnet** 선택 (기능과 속도의 균형)

### 단계 6: 저장 및 테스트

저장 후 바로 사용 가능:

```
code-improver 에이전트를 사용하여 이 프로젝트의 개선 사항을 제안합니다
```

---

## 2. 서브에이전트 구성 방법

### 범위와 우선순위

| 위치 | 범위 | 우선순위 | 용도 |
|------|------|----------|------|
| `--agents` CLI 플래그 | 현재 세션 | 1 (최고) | 빠른 테스트, 자동화 스크립트 |
| `.claude/agents/` | 현재 프로젝트 | 2 | 팀 공유, 버전 제어 |
| `~/.claude/agents/` | 모든 프로젝트 | 3 | 개인 유틸리티 |
| 플러그인의 `agents/` | 플러그인 활성화 위치 | 4 (최저) | 배포된 플러그인 |

### `/agents` 명령 활용

```
/agents
```

이 명령으로:
- 사용 가능한 모든 서브에이전트 보기
- 새 서브에이전트 만들기 (안내 설정 또는 Claude 생성)
- 기존 서브에이전트 편집/삭제
- 중복 시 활성 서브에이전트 확인

### CLI로 서브에이전트 정의

세션에만 존재하는 빠른 테스트용:

```bash
claude --agents '{
  "code-reviewer": {
    "description": "Expert code reviewer. Use proactively after code changes.",
    "prompt": "You are a senior code reviewer. Focus on code quality, security, and best practices.",
    "tools": ["Read", "Grep", "Glob", "Bash"],
    "model": "sonnet"
  }
}'
```

---

## 3. 서브에이전트 파일 작성

서브에이전트는 YAML 프론트매터가 있는 Markdown 파일로 정의됩니다.

### 기본 구조

```markdown
---
name: code-reviewer
description: Reviews code for quality and best practices
tools: Read, Glob, Grep
model: sonnet
---

You are a code reviewer. When invoked, analyze the code and provide
specific, actionable feedback on quality, security, and best practices.
```

### 지원되는 프론트매터 필드

| 필드 | 필수 | 설명 |
|------|------|------|
| `name` | ✅ | 고유 식별자 (소문자, 하이픈 사용) |
| `description` | ✅ | Claude가 위임 시기를 결정하는 데 사용 |
| `tools` | ❌ | 사용 가능한 도구 목록 (생략 시 모든 도구 상속) |
| `disallowedTools` | ❌ | 거부할 도구 목록 |
| `model` | ❌ | `sonnet`, `opus`, `haiku`, `inherit` (기본: `sonnet`) |
| `permissionMode` | ❌ | 권한 모드 설정 |
| `skills` | ❌ | 컨텍스트에 로드할 스킬 |
| `hooks` | ❌ | 라이프사이클 훅 정의 |

### 모델 선택 가이드

| 모델 | 용도 |
|------|------|
| **haiku** | 빠른 검색, 간단한 분석 |
| **sonnet** | 균형 잡힌 코드 분석, 리뷰 |
| **opus** | 복잡한 추론, 상세한 분석 |
| **inherit** | 주 대화와 동일한 모델 사용 |

---

## 4. 기능 제어: 도구와 권한

### 도구 액세스 제한

**허용 목록 방식**:
```yaml
---
name: safe-researcher
description: Research agent with restricted capabilities
tools: Read, Grep, Glob, Bash
---
```

**거부 목록 방식**:
```yaml
---
name: safe-researcher
description: Research agent with restricted capabilities
tools: Read, Grep, Glob, Bash
disallowedTools: Write, Edit
---
```

### 권한 모드

| 모드 | 동작 |
|------|------|
| `default` | 프롬프트를 사용한 표준 권한 확인 |
| `acceptEdits` | 파일 편집 자동 수락 |
| `dontAsk` | 권한 프롬프트 자동 거부 |
| `bypassPermissions` | 모든 권한 확인 건너뛰기 ⚠️ |
| `plan` | 계획 모드 (읽기 전용 탐색) |

```yaml
---
name: auto-fixer
description: Automatically fixes code issues
permissionMode: acceptEdits
---
```

> ⚠️ **주의**: `bypassPermissions`는 모든 권한 확인을 건너뛰어 위험할 수 있습니다.

### 특정 서브에이전트 비활성화

**settings.json에서**:
```json
{
  "permissions": {
    "deny": ["Task(Explore)", "Task(my-custom-agent)"]
  }
}
```

**CLI 플래그로**:
```bash
claude --disallowedTools "Task(Explore)"
```

---

## 5. 서브에이전트 훅 정의

### 프론트매터에서 훅 정의

서브에이전트가 활성화된 동안만 실행되는 훅:

```yaml
---
name: code-reviewer
description: Review code changes with automatic linting
hooks:
  PreToolUse:
    - matcher: "Bash"
      hooks:
        - type: command
          command: "./scripts/validate-command.sh $TOOL_INPUT"
  PostToolUse:
    - matcher: "Edit|Write"
      hooks:
        - type: command
          command: "./scripts/run-linter.sh"
---
```

### 지원되는 훅 이벤트

| 이벤트 | 매처 입력 | 발생 시점 |
|--------|-----------|-----------|
| `PreToolUse` | 도구 이름 | 서브에이전트가 도구 사용 전 |
| `PostToolUse` | 도구 이름 | 서브에이전트가 도구 사용 후 |
| `Stop` | (없음) | 서브에이전트 완료 시 |

### settings.json에서 서브에이전트 이벤트 훅

```json
{
  "hooks": {
    "SubagentStart": [
      {
        "matcher": "db-agent",
        "hooks": [
          { "type": "command", "command": "./scripts/setup-db-connection.sh" }
        ]
      }
    ],
    "SubagentStop": [
      {
        "matcher": "db-agent",
        "hooks": [
          { "type": "command", "command": "./scripts/cleanup-db-connection.sh" }
        ]
      }
    ]
  }
}
```

---

## 6. 서브에이전트 작업 패턴

### 자동 위임 이해

Claude는 요청의 작업 설명과 서브에이전트의 `description` 필드를 기반으로 자동 위임합니다.

**명시적 요청도 가능**:
```
Use the test-runner subagent to fix failing tests
Have the code-reviewer subagent look at my recent changes
```

### 포그라운드 vs 백그라운드 실행

| 모드 | 특징 |
|------|------|
| **포그라운드** | 완료까지 주 대화 차단, 권한 프롬프트가 사용자에게 전달됨 |
| **백그라운드** | 동시 실행, 부모 권한 상속, 권한 없으면 자동 거부 |

**백그라운드로 이동**:
- Claude에게 "run this in the background" 요청
- 실행 중 **Ctrl+B** 누르기

### 일반적인 패턴

#### 1. 대량 작업 격리

```
Use a subagent to run the test suite and report only the failing tests with their error messages
```

테스트 실행, 로그 처리 등 많은 출력을 생성하는 작업을 격리하여 요약만 반환받습니다.

#### 2. 병렬 연구 실행

```
Research the authentication, database, and API modules in parallel using separate subagents
```

독립적인 조사를 동시에 수행하고 결과를 종합합니다.

#### 3. 서브에이전트 체인

```
Use the code-reviewer subagent to find performance issues, then use the optimizer subagent to fix them
```

다단계 워크플로우를 순차적으로 실행합니다.

### 언제 서브에이전트를 사용해야 하나?

**주 대화 사용**:
- 빈번한 왕복/반복 개선이 필요한 경우
- 여러 단계가 상당한 컨텍스트를 공유하는 경우
- 빠르고 대상이 지정된 변경을 할 때
- 지연 시간이 중요한 경우

**서브에이전트 사용**:
- 주 컨텍스트에 필요하지 않은 자세한 출력 생성 시
- 특정 도구 제한/권한 적용이 필요한 경우
- 자체 포함된 작업으로 요약만 반환 가능한 경우

> 💡 서브에이전트는 다른 서브에이전트를 생성할 수 없습니다.

### 서브에이전트 재개

```
Use the code-reviewer subagent to review the authentication module
[Agent completes]

Continue that code review and now analyze the authorization logic
[Claude resumes the subagent with full context from previous conversation]
```

재개된 서브에이전트는 이전 도구 호출, 결과, 추론을 포함한 전체 대화 기록을 유지합니다.

---

## 7. 실전 예제

### 코드 검토자 (읽기 전용)

```markdown
---
name: code-reviewer
description: Expert code review specialist. Proactively reviews code for quality, security, and maintainability. Use immediately after writing or modifying code.
tools: Read, Grep, Glob, Bash
model: inherit
---

You are a senior code reviewer ensuring high standards of code quality and security.

When invoked:
1. Run git diff to see recent changes
2. Focus on modified files
3. Begin review immediately

Review checklist:
- Code is clear and readable
- Functions and variables are well-named
- No duplicated code
- Proper error handling
- No exposed secrets or API keys
- Input validation implemented
- Good test coverage
- Performance considerations addressed

Provide feedback organized by priority:
- Critical issues (must fix)
- Warnings (should fix)
- Suggestions (consider improving)

Include specific examples of how to fix issues.
```

### 디버거 (편집 권한 포함)

```markdown
---
name: debugger
description: Debugging specialist for errors, test failures, and unexpected behavior. Use proactively when encountering any issues.
tools: Read, Edit, Bash, Grep, Glob
---

You are an expert debugger specializing in root cause analysis.

When invoked:
1. Capture error message and stack trace
2. Identify reproduction steps
3. Isolate the failure location
4. Implement minimal fix
5. Verify solution works

Debugging process:
- Analyze error messages and logs
- Check recent code changes
- Form and test hypotheses
- Add strategic debug logging
- Inspect variable states

For each issue, provide:
- Root cause explanation
- Evidence supporting the diagnosis
- Specific code fix
- Testing approach
- Prevention recommendations

Focus on fixing the underlying issue, not the symptoms.
```

### 데이터 과학자

```markdown
---
name: data-scientist
description: Data analysis expert for SQL queries, BigQuery operations, and data insights. Use proactively for data analysis tasks and queries.
tools: Bash, Read, Write
model: sonnet
---

You are a data scientist specializing in SQL and BigQuery analysis.

When invoked:
1. Understand the data analysis requirement
2. Write efficient SQL queries
3. Use BigQuery command line tools (bq) when appropriate
4. Analyze and summarize results
5. Present findings clearly

Key practices:
- Write optimized SQL queries with proper filters
- Use appropriate aggregations and joins
- Include comments explaining complex logic
- Format results for readability
- Provide data-driven recommendations

For each analysis:
- Explain the query approach
- Document any assumptions
- Highlight key findings
- Suggest next steps based on data

Always ensure queries are efficient and cost-effective.
```

### 읽기 전용 DB 쿼리 에이전트 (훅 활용)

```yaml
---
name: db-reader
description: Execute read-only database queries
tools: Bash
hooks:
  PreToolUse:
    - matcher: "Bash"
      hooks:
        - type: command
          command: "./scripts/validate-readonly-query.sh"
---

You are a database query agent. Execute only SELECT queries.
Never run INSERT, UPDATE, DELETE, or any DDL commands.
```

---

## 8. 문제 해결 및 팁

### 서브에이전트가 로드되지 않을 때

1. **파일 위치 확인**: 올바른 디렉토리에 있는지 확인
2. **YAML 구문 검사**: 프론트매터 형식이 올바른지 확인
3. **세션 재시작**: 수동으로 파일을 추가한 경우 `/agents`로 즉시 로드

### 서브에이전트가 자동 호출되지 않을 때

**description 필드 개선**:
- "use proactively" 문구 포함
- 구체적인 트리거 조건 명시

```yaml
description: Expert code reviewer. Use proactively after code changes.
```

### 컨텍스트 관리

- **트랜스크립트 위치**: `~/.claude/projects/{project}/{sessionId}/subagents/`
- **압축**: 서브에이전트도 자동 압축 지원
- **세션 지속성**: 세션 내에서 유지, 30일 후 자동 정리

### 베스트 프랙티스

1. **집중된 서브에이전트 설계**: 각 서브에이전트는 특정 작업에 탁월해야 함
2. **상세한 설명 작성**: Claude가 위임 시기를 결정하는 핵심
3. **도구 액세스 제한**: 보안과 집중을 위해 필요한 권한만 부여
4. **버전 제어에 체크인**: 팀과 프로젝트 서브에이전트 공유

---

## 관련 문서

- [플러그인으로 서브에이전트 배포](/plugins) - 팀/프로젝트 간 공유
- [Agent SDK로 프로그래밍 방식 실행](/headless) - CI/CD 및 자동화
- [MCP 서버 사용](/mcp) - 외부 도구 및 데이터 액세스 제공
- [스킬 활용](/skills) - 서브에이전트에 전문 지식 추가

---

*마지막 업데이트: 2026-01*
