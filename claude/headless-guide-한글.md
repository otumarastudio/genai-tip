# Claude Code 헤드리스 모드 (프로그래밍 방식 실행) 완벽 가이드

> CLI, Python, TypeScript에서 Claude Code를 자동화하고 프로그래밍 방식으로 실행하기

---

## 한눈에 보기

### 헤드리스 모드란?

헤드리스 모드는 Claude Code를 **비대화형으로 실행**하는 방식입니다. `-p` 플래그를 사용하여 스크립트, CI/CD 파이프라인, 자동화 워크플로우에서 Claude Code를 활용할 수 있습니다.

### 대화형 모드 vs 헤드리스 모드

| 모드 | 특징 | 사용 시나리오 |
|------|------|--------------|
| **대화형** | 실시간 입력/출력, 슬래시 명령 사용 가능 | 일반 개발 작업 |
| **헤드리스** (`-p`) | 비대화형, 자동화 가능, 구조화된 출력 | CI/CD, 스크립트, 자동화 |

### 주요 기능

| 기능 | 설명 |
|------|------|
| **구조화된 출력** | JSON, 스트리밍 JSON 형식 지원 |
| **도구 자동 승인** | 특정 도구를 프롬프트 없이 사용 |
| **대화 연속** | 이전 대화를 이어서 진행 |
| **시스템 프롬프트 커스터마이징** | 기본 동작 유지하면서 지침 추가 |

---

## 목차

1. [기본 사용법](#1-기본-사용법)
2. [구조화된 출력 얻기](#2-구조화된-출력-얻기)
3. [도구 자동 승인](#3-도구-자동-승인)
4. [대화 연속하기](#4-대화-연속하기)
5. [시스템 프롬프트 커스터마이징](#5-시스템-프롬프트-커스터마이징)
6. [실전 예제](#6-실전-예제)
7. [CI/CD 통합](#7-cicd-통합)

---

## 1. 기본 사용법

### `-p` 플래그로 실행

```bash
# 기본 사용법
claude -p "질문 또는 작업 지시"

# 예: 코드베이스에 대한 질문
claude -p "What does the auth module do?"

# 예: 버그 수정 요청
claude -p "Find and fix the bug in auth.py" --allowedTools "Read,Edit,Bash"
```

### 주요 CLI 옵션

| 옵션 | 설명 |
|------|------|
| `-p`, `--print` | 비대화형 모드로 실행 |
| `--continue` | 가장 최근 대화 계속 |
| `--resume <session-id>` | 특정 세션 계속 |
| `--allowedTools <tools>` | 자동 승인할 도구 지정 |
| `--output-format <format>` | 출력 형식 지정 (text, json, stream-json) |
| `--json-schema <schema>` | 출력 스키마 지정 |
| `--append-system-prompt` | 시스템 프롬프트에 지침 추가 |

---

## 2. 구조화된 출력 얻기

### 출력 형식 옵션

| 형식 | 설명 |
|------|------|
| `text` (기본값) | 일반 텍스트 출력 |
| `json` | 결과, 세션 ID, 메타데이터 포함 JSON |
| `stream-json` | 실시간 스트리밍용 줄 구분 JSON |

### JSON 출력

```bash
# JSON 형식으로 프로젝트 요약
claude -p "Summarize this project" --output-format json
```

응답 예시:
```json
{
  "result": "이 프로젝트는...",
  "session_id": "abc123",
  "usage": {...}
}
```

### JSON 스키마로 구조화된 출력

```bash
# 함수 이름을 배열로 추출
claude -p "Extract the main function names from auth.py" \
  --output-format json \
  --json-schema '{"type":"object","properties":{"functions":{"type":"array","items":{"type":"string"}}},"required":["functions"]}'
```

응답의 `structured_output` 필드에 스키마에 맞는 데이터가 포함됩니다.

### jq로 결과 추출

```bash
# 텍스트 결과만 추출
claude -p "Summarize this project" --output-format json | jq -r '.result'

# 구조화된 출력 추출
claude -p "Extract function names from auth.py" \
  --output-format json \
  --json-schema '...' \
  | jq '.structured_output'
```

---

## 3. 도구 자동 승인

### `--allowedTools` 사용

```bash
# Read, Edit, Bash 도구 자동 승인
claude -p "Run the test suite and fix any failures" \
  --allowedTools "Read,Edit,Bash"
```

### 특정 명령만 허용

```bash
# 특정 git 명령만 허용
claude -p "Look at my staged changes and create an appropriate commit" \
  --allowedTools "Bash(git diff:*),Bash(git log:*),Bash(git status:*),Bash(git commit:*)"
```

### 일반적인 도구 조합

| 작업 | 권장 도구 |
|------|----------|
| 코드 분석 | `Read` |
| 코드 수정 | `Read,Edit` |
| 테스트 실행 | `Bash,Read` |
| 전체 개발 | `Bash,Read,Edit` |

---

## 4. 대화 연속하기

### 가장 최근 대화 계속

```bash
# 첫 번째 요청
claude -p "Review this codebase for performance issues"

# 이전 대화 계속
claude -p "Now focus on the database queries" --continue

# 계속 이어서
claude -p "Generate a summary of all issues found" --continue
```

### 특정 세션 재개

```bash
# 세션 ID 캡처
session_id=$(claude -p "Start a review" --output-format json | jq -r '.session_id')

# 해당 세션 재개
claude -p "Continue that review" --resume "$session_id"
```

---

## 5. 시스템 프롬프트 커스터마이징

### 지침 추가 (`--append-system-prompt`)

기본 Claude Code 동작을 유지하면서 추가 지침을 제공합니다.

```bash
# PR diff를 보안 관점에서 검토
gh pr diff "$1" | claude -p \
  --append-system-prompt "You are a security engineer. Review for vulnerabilities." \
  --output-format json
```

### 시스템 프롬프트 완전 대체 (`--system-prompt`)

```bash
# 기본 프롬프트를 완전히 대체
claude -p "Analyze this code" \
  --system-prompt "You are a code review expert..."
```

---

## 6. 실전 예제

### 예제 1: 자동 코드 리뷰

```bash
#!/bin/bash
# review.sh - 변경된 파일 자동 리뷰

files=$(git diff --name-only HEAD~1)

for file in $files; do
  echo "Reviewing: $file"
  claude -p "Review $file for bugs, security issues, and best practices" \
    --allowedTools "Read" \
    --output-format json | jq -r '.result'
done
```

### 예제 2: 자동 커밋 메시지 생성

```bash
#!/bin/bash
# commit.sh - 스테이징된 변경사항으로 커밋 생성

claude -p "Look at my staged changes and create an appropriate commit" \
  --allowedTools "Bash(git diff:*),Bash(git log:*),Bash(git status:*),Bash(git commit:*)"
```

### 예제 3: 테스트 자동 수정

```bash
#!/bin/bash
# fix-tests.sh - 실패한 테스트 자동 수정

claude -p "Run the test suite, identify failures, and fix them" \
  --allowedTools "Bash,Read,Edit"
```

### 예제 4: 문서 생성

```bash
#!/bin/bash
# generate-docs.sh - API 문서 자동 생성

claude -p "Generate API documentation for all public functions in src/" \
  --allowedTools "Read" \
  --output-format json \
  --json-schema '{"type":"object","properties":{"documentation":{"type":"string"}},"required":["documentation"]}' \
  | jq -r '.structured_output.documentation' > API.md
```

### 예제 5: 보안 감사

```bash
#!/bin/bash
# security-audit.sh - 보안 취약점 스캔

claude -p "Scan the codebase for security vulnerabilities and create a report" \
  --append-system-prompt "You are a security expert. Focus on OWASP Top 10 vulnerabilities." \
  --allowedTools "Read,Bash(grep:*)" \
  --output-format json | jq -r '.result' > security-report.md
```

---

## 7. CI/CD 통합

### GitHub Actions

```yaml
# .github/workflows/code-review.yml
name: AI Code Review

on:
  pull_request:
    types: [opened, synchronize]

jobs:
  review:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Install Claude Code
        run: npm install -g @anthropic-ai/claude-code

      - name: Run Code Review
        env:
          ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
        run: |
          claude -p "Review the changes in this PR for bugs and improvements" \
            --allowedTools "Read" \
            --output-format json | jq -r '.result' > review.md

      - name: Post Review Comment
        uses: actions/github-script@v7
        with:
          script: |
            const fs = require('fs');
            const review = fs.readFileSync('review.md', 'utf8');
            github.rest.issues.createComment({
              owner: context.repo.owner,
              repo: context.repo.repo,
              issue_number: context.issue.number,
              body: review
            });
```

### GitLab CI/CD

```yaml
# .gitlab-ci.yml
code-review:
  stage: test
  script:
    - npm install -g @anthropic-ai/claude-code
    - claude -p "Review the changes for bugs and security issues" \
        --allowedTools "Read" \
        --output-format text > review.txt
  artifacts:
    paths:
      - review.txt
```

---

## 빠른 참조

### 자주 사용하는 명령

```bash
# 기본 질문
claude -p "What does this code do?"

# JSON 출력
claude -p "Summarize" --output-format json

# 도구 허용
claude -p "Fix tests" --allowedTools "Bash,Read,Edit"

# 대화 계속
claude -p "Continue" --continue

# 특정 세션 재개
claude -p "Continue" --resume <session-id>
```

### 주요 플래그

```bash
-p, --print              # 비대화형 모드
--continue               # 최근 대화 계속
--resume <id>            # 특정 세션 재개
--allowedTools <tools>   # 도구 자동 승인
--output-format <fmt>    # 출력 형식 (text/json/stream-json)
--json-schema <schema>   # JSON 스키마 지정
--append-system-prompt   # 시스템 프롬프트 추가
--system-prompt          # 시스템 프롬프트 대체
```

---

## 관련 문서

- [CLI 참조](/cli-reference) - 모든 CLI 플래그 및 옵션
- [Agent SDK](https://platform.claude.com/docs/en/agent-sdk/overview) - Python/TypeScript SDK
- [GitHub Actions](/github-actions) - GitHub 워크플로우 통합
- [GitLab CI/CD](/gitlab-ci-cd) - GitLab 파이프라인 통합

---

*마지막 업데이트: 2026-01*
