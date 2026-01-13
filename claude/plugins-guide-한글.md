# Claude Code 플러그인 만들기 완벽 가이드

> 슬래시 명령어, 에이전트, 훅, 스킬, MCP 서버를 사용하여 Claude Code 확장하기

---

## 한눈에 보기

### 플러그인이란?

플러그인은 Claude Code의 기능을 확장하는 **패키지화된 확장 프로그램**입니다. 팀과 커뮤니티 전체에서 공유할 수 있는 사용자 정의 기능을 제공합니다.

### 플러그인 vs 독립 실행형 구성

| 접근 방식 | 슬래시 명령어 이름 | 최적 용도 |
|-----------|-------------------|-----------|
| **독립 실행형** (`.claude/` 디렉토리) | `/hello` | 개인 워크플로우, 프로젝트별 사용자 정의, 빠른 실험 |
| **플러그인** (`.claude-plugin/plugin.json`) | `/plugin-name:hello` | 팀 공유, 커뮤니티 배포, 버전 관리, 프로젝트 간 재사용 |

### 플러그인 구조

```
my-plugin/
├── .claude-plugin/    # 플러그인 매니페스트만 포함 (필수)
│   └── plugin.json
├── commands/          # 슬래시 명령어
├── agents/            # 사용자 정의 에이전트
├── skills/            # 에이전트 스킬
├── hooks/             # 이벤트 핸들러 (hooks.json)
├── .mcp.json          # MCP 서버 구성
└── .lsp.json          # LSP 서버 구성
```

> ⚠️ **주의**: `commands/`, `agents/`, `skills/` 등을 `.claude-plugin/` 내부에 넣지 마세요!

---

## 목차

1. [빠른 시작: 첫 번째 플러그인 만들기](#1-빠른-시작-첫-번째-플러그인-만들기)
2. [플러그인 매니페스트](#2-플러그인-매니페스트)
3. [슬래시 명령어 추가](#3-슬래시-명령어-추가)
4. [스킬 추가](#4-스킬-추가)
5. [LSP 서버 추가](#5-lsp-서버-추가)
6. [플러그인 테스트 및 디버깅](#6-플러그인-테스트-및-디버깅)
7. [기존 구성을 플러그인으로 변환](#7-기존-구성을-플러그인으로-변환)
8. [플러그인 공유 및 배포](#8-플러그인-공유-및-배포)

---

## 1. 빠른 시작: 첫 번째 플러그인 만들기

### 필수 조건

- Claude Code 버전 1.0.33 이상 (`claude --version`으로 확인)

### 단계 1: 플러그인 디렉토리 만들기

```bash
mkdir my-first-plugin
```

### 단계 2: 플러그인 매니페스트 만들기

```bash
mkdir my-first-plugin/.claude-plugin
```

`my-first-plugin/.claude-plugin/plugin.json` 생성:

```json
{
  "name": "my-first-plugin",
  "description": "A greeting plugin to learn the basics",
  "version": "1.0.0",
  "author": {
    "name": "Your Name"
  }
}
```

### 단계 3: 슬래시 명령어 추가

```bash
mkdir my-first-plugin/commands
```

`my-first-plugin/commands/hello.md` 생성:

```markdown
---
description: Greet the user with a friendly message
---

# Hello Command

Greet the user warmly and ask how you can help them today.
```

### 단계 4: 플러그인 테스트

```bash
claude --plugin-dir ./my-first-plugin
```

Claude Code에서 테스트:

```
/my-first-plugin:hello
```

### 단계 5: 인수 추가

`hello.md` 업데이트:

```markdown
---
description: Greet the user with a personalized message
---

# Hello Command

Greet the user named "$ARGUMENTS" warmly and ask how you can help them today.
Make the greeting personal and encouraging.
```

재시작 후 테스트:

```
/my-first-plugin:hello Alex
```

---

## 2. 플러그인 매니페스트

### 기본 필드

```json
{
  "name": "my-plugin",
  "description": "플러그인 설명",
  "version": "1.0.0",
  "author": {
    "name": "작성자 이름"
  }
}
```

| 필드 | 목적 |
|------|------|
| `name` | 고유 식별자, 슬래시 명령어 네임스페이스 (예: `/my-plugin:hello`) |
| `description` | 플러그인 관리자에 표시되는 설명 |
| `version` | 의미 있는 버전 관리 (예: `1.0.0`) |
| `author` | 저작권 표시용 (선택) |

### 추가 필드

```json
{
  "name": "my-plugin",
  "description": "플러그인 설명",
  "version": "1.0.0",
  "homepage": "https://github.com/user/plugin",
  "repository": "https://github.com/user/plugin.git",
  "license": "MIT",
  "author": {
    "name": "작성자",
    "email": "email@example.com"
  }
}
```

---

## 3. 슬래시 명령어 추가

### 기본 명령어

`commands/` 디렉토리에 Markdown 파일로 생성합니다.

파일 이름 = 명령어 이름 (예: `deploy.md` → `/plugin-name:deploy`)

```markdown
---
description: Deploy the application to production
---

# Deploy Command

Deploy the current application to production environment.
Ensure all tests pass before deploying.
```

### 인수 사용

| 자리 표시자 | 설명 |
|-------------|------|
| `$ARGUMENTS` | 명령어 뒤의 모든 텍스트 |
| `$1`, `$2`, ... | 개별 매개변수 |

```markdown
---
description: Create a new component
---

# Create Component

Create a new React component named "$1" in the "$2" directory.
If no directory is specified, use "src/components".
```

사용:
```
/my-plugin:create Button src/ui
```

---

## 4. 스킬 추가

### 스킬 디렉토리 구조

```
my-plugin/
├── .claude-plugin/
│   └── plugin.json
└── skills/
    └── code-review/
        └── SKILL.md
```

### SKILL.md 작성

```yaml
---
name: code-review
description: Reviews code for best practices and potential issues. Use when reviewing code, checking PRs, or analyzing code quality.
---

When reviewing code, check for:
1. Code organization and structure
2. Error handling
3. Security concerns
4. Test coverage
```

> 💡 플러그인 설치 후 Claude Code를 재시작하여 스킬을 로드합니다.

---

## 5. LSP 서버 추가

LSP (Language Server Protocol) 플러그인은 Claude에 실시간 코드 인텔리전스를 제공합니다.

### .lsp.json 작성

```json
{
  "go": {
    "command": "gopls",
    "args": ["serve"],
    "extensionToLanguage": {
      ".go": "go"
    }
  }
}
```

> 💡 사용자는 자신의 머신에 언어 서버 바이너리를 설치해야 합니다.

### 일반적인 언어 서버

| 언어 | 명령어 | 필요한 바이너리 |
|------|--------|----------------|
| Go | `gopls` | `gopls` |
| Python | `pyright-langserver` | `pyright` |
| TypeScript | `typescript-language-server` | `typescript-language-server` |
| Rust | `rust-analyzer` | `rust-analyzer` |

---

## 6. 플러그인 테스트 및 디버깅

### 로컬 테스트

```bash
# 단일 플러그인 로드
claude --plugin-dir ./my-plugin

# 여러 플러그인 로드
claude --plugin-dir ./plugin-one --plugin-dir ./plugin-two
```

> 💡 플러그인 변경 시마다 Claude Code를 재시작하여 업데이트를 적용합니다.

### 테스트 항목

1. **명령어**: `/command-name`으로 시도
2. **에이전트**: `/agents`에 나타나는지 확인
3. **훅**: 예상대로 작동하는지 확인

### 문제 해결

| 문제 | 해결 방법 |
|------|----------|
| 플러그인이 로드되지 않음 | 디렉토리 구조 확인, `.claude-plugin/` 내부에 파일이 없는지 확인 |
| 명령어가 작동하지 않음 | 파일 이름과 프론트매터 확인 |
| 스킬이 나타나지 않음 | 캐시 지우기: `rm -rf ~/.claude/plugins/cache` |

---

## 7. 기존 구성을 플러그인으로 변환

### 단계 1: 플러그인 구조 만들기

```bash
mkdir -p my-plugin/.claude-plugin
```

`my-plugin/.claude-plugin/plugin.json`:

```json
{
  "name": "my-plugin",
  "description": "Migrated from standalone configuration",
  "version": "1.0.0"
}
```

### 단계 2: 기존 파일 복사

```bash
# 명령어 복사
cp -r .claude/commands my-plugin/

# 에이전트 복사 (있는 경우)
cp -r .claude/agents my-plugin/

# 스킬 복사 (있는 경우)
cp -r .claude/skills my-plugin/
```

### 단계 3: 훅 마이그레이션

`my-plugin/hooks/hooks.json` 생성:

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [{ "type": "command", "command": "npm run lint:fix $FILE" }]
      }
    ]
  }
}
```

### 단계 4: 테스트

```bash
claude --plugin-dir ./my-plugin
```

### 마이그레이션 변경 사항

| 독립 실행형 (`.claude/`) | 플러그인 |
|-------------------------|---------|
| 한 프로젝트에서만 사용 | 마켓플레이스를 통해 공유 가능 |
| `.claude/commands/` | `plugin-name/commands/` |
| `settings.json`의 훅 | `hooks/hooks.json`의 훅 |
| 수동으로 복사해서 공유 | `/plugin install`로 설치 |

---

## 8. 플러그인 공유 및 배포

### 배포 준비

1. **문서 추가**: 설치/사용 지침이 포함된 `README.md`
2. **버전 관리**: `plugin.json`에서 의미 있는 버전 관리
3. **테스트**: 팀 구성원과 함께 테스트

### 배포 방법

- **프로젝트 범위**: `.claude/plugins/`에 저장하고 버전 제어에 체크인
- **마켓플레이스**: [플러그인 마켓플레이스](/plugin-marketplaces)를 통해 배포
- **직접 공유**: Git 저장소 URL 공유

### 마켓플레이스 배포

마켓플레이스에 플러그인을 등록하면 다른 사용자가 쉽게 설치할 수 있습니다:

```bash
/plugin install my-plugin@marketplace-name
```

---

## 실전 팁

### 1. 네임스페이싱 이해

플러그인 명령어는 항상 네임스페이스가 붙습니다:
- `/my-plugin:hello` ✅
- `/hello` ❌ (독립 실행형만 가능)

이는 여러 플러그인 간 충돌을 방지합니다.

### 2. 디렉토리 구조 주의

```
# 올바른 구조 ✅
my-plugin/
├── .claude-plugin/
│   └── plugin.json
├── commands/
└── skills/

# 잘못된 구조 ❌
my-plugin/
└── .claude-plugin/
    ├── plugin.json
    ├── commands/      # 여기 넣지 마세요!
    └── skills/        # 여기 넣지 마세요!
```

### 3. 빠른 반복 개발

독립 실행형 (`.claude/`)에서 먼저 개발하고, 완성되면 플러그인으로 변환하세요.

---

## 관련 문서

- [플러그인 발견 및 설치](/discover-plugins) - 마켓플레이스 검색 및 설치
- [슬래시 명령어](/slash-commands) - 명령어 개발 상세
- [서브에이전트](/sub-agents) - 에이전트 구성
- [에이전트 스킬](/skills) - Claude 기능 확장
- [훅](/hooks) - 이벤트 처리 및 자동화
- [MCP](/mcp) - 외부 도구 통합

---

*마지막 업데이트: 2026-01*
