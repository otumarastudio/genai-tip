# Claude Code 훅(Hooks) 완벽 가이드

> Claude Code의 라이프사이클에서 셸 명령을 자동 실행하여 동작 사용자 정의하기

---

## 한눈에 보기

### 훅이란?

훅은 Claude Code의 라이프사이클의 다양한 지점에서 **자동으로 실행되는 사용자 정의 셸 명령**입니다.

LLM이 실행하도록 "선택"하는 것이 아닌, **결정론적 제어**를 제공합니다 - 특정 작업이 **항상** 발생하도록 보장합니다.

### 훅 사용 사례

| 용도 | 예시 |
|------|------|
| **알림** | 입력 대기 시 사용자 정의 알림 |
| **자동 포맷팅** | 파일 편집 후 자동으로 `prettier`, `gofmt` 실행 |
| **로깅** | 실행된 모든 명령 추적 (규정 준수/디버깅) |
| **피드백** | 코드베이스 규칙 미준수 시 자동 피드백 |
| **사용자 정의 권한** | 프로덕션 파일이나 민감한 디렉토리 수정 차단 |

### 훅 이벤트 요약

| 이벤트 | 발생 시점 | 차단 가능 |
|--------|----------|----------|
| `PreToolUse` | 도구 호출 전 | ✅ |
| `PermissionRequest` | 권한 대화상자 표시 시 | ✅ |
| `PostToolUse` | 도구 호출 완료 후 | ❌ |
| `UserPromptSubmit` | 사용자 프롬프트 제출 시 | ❌ |
| `Notification` | 알림 발송 시 | ❌ |
| `Stop` | Claude가 응답 완료 시 | ❌ |
| `SubagentStop` | 서브에이전트 작업 완료 시 | ❌ |
| `PreCompact` | 컴팩트 작업 실행 전 | ❌ |
| `SessionStart` | 새 세션 시작/기존 세션 재개 시 | ❌ |
| `SessionEnd` | 세션 종료 시 | ❌ |

---

## 목차

1. [빠른 시작: 첫 번째 훅 만들기](#1-빠른-시작-첫-번째-훅-만들기)
2. [훅 구성 방법](#2-훅-구성-방법)
3. [훅 이벤트 상세](#3-훅-이벤트-상세)
4. [실전 예제](#4-실전-예제)
5. [보안 고려사항](#5-보안-고려사항)
6. [디버깅](#6-디버깅)

---

## 1. 빠른 시작: 첫 번째 훅 만들기

Claude Code가 실행하는 셸 명령을 로깅하는 훅을 만들어봅니다.

### 필수 조건

`jq` 설치 (명령줄 JSON 처리용):

```bash
# macOS
brew install jq

# Ubuntu/Debian
sudo apt-get install jq
```

### 단계 1: 훅 구성 열기

```
/hooks
```

`PreToolUse` 훅 이벤트 선택

### 단계 2: 매처 추가

`+ Add new matcher…` 선택

매처에 `Bash` 입력

> 💡 모든 도구를 일치시키려면 `*` 사용

### 단계 3: 훅 추가

`+ Add new hook…` 선택 후 명령 입력:

```bash
jq -r '"\(.tool_input.command) - \(.tool_input.description // "No description")"' >> ~/.claude/bash-command-log.txt
```

### 단계 4: 구성 저장

저장 위치로 `User settings` 선택 (모든 프로젝트에 적용)

`Esc`를 눌러 REPL로 돌아가기

### 단계 5: 훅 확인

```
/hooks
```

또는 `~/.claude/settings.json` 확인:

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "jq -r '\"\\(.tool_input.command) - \\(.tool_input.description // \"No description\")\"' >> ~/.claude/bash-command-log.txt"
          }
        ]
      }
    ]
  }
}
```

### 단계 6: 훅 테스트

Claude에게 `ls` 명령 실행 요청 후 로그 확인:

```bash
cat ~/.claude/bash-command-log.txt
```

출력:
```
ls - Lists files and directories
```

---

## 2. 훅 구성 방법

### 구성 형식

```json
{
  "hooks": {
    "<EventName>": [
      {
        "matcher": "<패턴>",
        "hooks": [
          {
            "type": "command",
            "command": "<실행할 명령>"
          }
        ]
      }
    ]
  }
}
```

### 매처 패턴

| 패턴 | 설명 |
|------|------|
| `Bash` | Bash 도구만 매칭 |
| `Edit\|Write` | Edit 또는 Write 도구 매칭 |
| `*` | 모든 도구 매칭 |
| `""` (빈 문자열) | 매칭 없이 항상 실행 (Notification 등) |

### 환경 변수

훅 명령에서 사용할 수 있는 변수:

| 변수 | 설명 |
|------|------|
| `$TOOL_INPUT` | 도구 입력 JSON |
| `$CLAUDE_PROJECT_DIR` | 프로젝트 디렉토리 경로 |
| `$FILE` | 파일 경로 (PostToolUse의 Edit/Write) |

### 저장 위치

| 범위 | 파일 |
|------|------|
| 사용자 수준 | `~/.claude/settings.json` |
| 프로젝트 수준 | `.claude/settings.json` |
| 로컬 | `.claude/settings.local.json` |

---

## 3. 훅 이벤트 상세

### PreToolUse

도구 호출 **전**에 실행됩니다. **차단 가능** (0이 아닌 종료 코드로 차단).

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "./scripts/validate-command.sh"
          }
        ]
      }
    ]
  }
}
```

**입력 JSON 구조**:
```json
{
  "tool_name": "Bash",
  "tool_input": {
    "command": "ls -la",
    "description": "List files"
  }
}
```

### PermissionRequest

권한 대화상자가 표시될 때 실행됩니다. 허용 또는 거부 가능.

### PostToolUse

도구 호출 **완료 후** 실행됩니다.

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "npm run lint:fix $FILE"
          }
        ]
      }
    ]
  }
}
```

### UserPromptSubmit

사용자가 프롬프트를 제출할 때 Claude가 처리하기 **전**에 실행됩니다.

### Notification

Claude Code가 알림을 보낼 때 실행됩니다.

```json
{
  "hooks": {
    "Notification": [
      {
        "matcher": "",
        "hooks": [
          {
            "type": "command",
            "command": "notify-send 'Claude Code' 'Awaiting your input'"
          }
        ]
      }
    ]
  }
}
```

### Stop / SubagentStop

Claude가 응답을 마치거나 서브에이전트가 완료될 때 실행됩니다.

### SessionStart / SessionEnd

세션 시작/종료 시 실행됩니다.

### PreCompact

컴팩트 작업 실행 전에 실행됩니다.

---

## 4. 실전 예제

### 코드 포맷팅 훅

편집 후 TypeScript 파일 자동 포맷:

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "jq -r '.tool_input.file_path' | { read file_path; if echo \"$file_path\" | grep -q '\\.ts$'; then npx prettier --write \"$file_path\"; fi; }"
          }
        ]
      }
    ]
  }
}
```

### 마크다운 포맷팅 훅

마크다운 파일의 누락된 언어 태그와 포맷팅 문제 자동 수정:

**settings.json**:
```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "\"$CLAUDE_PROJECT_DIR\"/.claude/hooks/markdown_formatter.py"
          }
        ]
      }
    ]
  }
}
```

**.claude/hooks/markdown_formatter.py**:
```python
#!/usr/bin/env python3
"""
Markdown formatter for Claude Code output.
Fixes missing language tags and spacing issues.
"""
import json
import sys
import re
import os

def detect_language(code):
    """Best-effort language detection from code content."""
    s = code.strip()

    # JSON detection
    if re.search(r'^\s*[{\[]', s):
        try:
            json.loads(s)
            return 'json'
        except:
            pass

    # Python detection
    if re.search(r'^\s*def\s+\w+\s*\(', s, re.M) or \
       re.search(r'^\s*(import|from)\s+\w+', s, re.M):
        return 'python'

    # JavaScript detection
    if re.search(r'\b(function\s+\w+\s*\(|const\s+\w+\s*=)', s):
        return 'javascript'

    # Bash detection
    if re.search(r'^#!.*\b(bash|sh)\b', s, re.M):
        return 'bash'

    # SQL detection
    if re.search(r'\b(SELECT|INSERT|UPDATE|DELETE|CREATE)\s+', s, re.I):
        return 'sql'

    return 'text'

def format_markdown(content):
    """Format markdown content with language detection."""
    def add_lang_to_fence(match):
        indent, info, body, closing = match.groups()
        if not info.strip():
            lang = detect_language(body)
            return f"{indent}```{lang}\n{body}{closing}\n"
        return match.group(0)

    fence_pattern = r'(?ms)^([ \t]{0,3})```([^\n]*)\n(.*?)(\n\1```)\s*$'
    content = re.sub(fence_pattern, add_lang_to_fence, content)
    content = re.sub(r'\n{3,}', '\n\n', content)

    return content.rstrip() + '\n'

# Main execution
try:
    input_data = json.load(sys.stdin)
    file_path = input_data.get('tool_input', {}).get('file_path', '')

    if not file_path.endswith(('.md', '.mdx')):
        sys.exit(0)

    if os.path.exists(file_path):
        with open(file_path, 'r', encoding='utf-8') as f:
            content = f.read()

        formatted = format_markdown(content)

        if formatted != content:
            with open(file_path, 'w', encoding='utf-8') as f:
                f.write(formatted)
            print(f"✓ Fixed markdown formatting in {file_path}")

except Exception as e:
    print(f"Error formatting markdown: {e}", file=sys.stderr)
    sys.exit(1)
```

스크립트 실행 권한 부여:
```bash
chmod +x .claude/hooks/markdown_formatter.py
```

### 사용자 정의 알림 훅

Claude가 입력을 필요로 할 때 데스크톱 알림:

```json
{
  "hooks": {
    "Notification": [
      {
        "matcher": "",
        "hooks": [
          {
            "type": "command",
            "command": "notify-send 'Claude Code' 'Awaiting your input'"
          }
        ]
      }
    ]
  }
}
```

### 파일 보호 훅

민감한 파일 편집 차단:

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "python3 -c \"import json, sys; data=json.load(sys.stdin); path=data.get('tool_input',{}).get('file_path',''); sys.exit(2 if any(p in path for p in ['.env', 'package-lock.json', '.git/']) else 0)\""
          }
        ]
      }
    ]
  }
}
```

### 명령 검증 훅

위험한 Bash 명령 차단:

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "./scripts/validate-bash-command.sh"
          }
        ]
      }
    ]
  }
}
```

**./scripts/validate-bash-command.sh**:
```bash
#!/bin/bash
# Read tool input from stdin
input=$(cat)
command=$(echo "$input" | jq -r '.tool_input.command')

# Block dangerous commands
dangerous_patterns=("rm -rf /" "dd if=" ":(){:|:&};:" "mkfs.")

for pattern in "${dangerous_patterns[@]}"; do
    if [[ "$command" == *"$pattern"* ]]; then
        echo "Blocked dangerous command: $command" >&2
        exit 2  # Non-zero exit code blocks the tool
    fi
done

exit 0
```

---

## 5. 보안 고려사항

> ⚠️ **중요**: 훅은 현재 환경의 자격 증명으로 에이전트 루프 중 **자동으로 실행**됩니다.

### 주요 위험

1. **데이터 유출**: 악의적인 훅이 데이터를 외부로 전송
2. **시스템 손상**: 위험한 명령 실행
3. **권한 상승**: 의도치 않은 권한 획득

### 베스트 프랙티스

1. **훅 검토**: 등록 전 항상 훅 구현을 검토
2. **최소 권한**: 필요한 최소한의 권한만 부여
3. **입력 검증**: 외부 입력을 신뢰하지 않음
4. **로깅**: 훅 실행을 기록하여 감사 추적

### 신뢰할 수 있는 소스만 사용

- 직접 작성한 훅만 사용
- 검증된 팀/조직의 훅 사용
- 타사 훅은 코드 검토 후 사용

---

## 6. 디버깅

### 훅 확인

```
/hooks
```

### 설정 파일 직접 확인

```bash
# 사용자 설정
cat ~/.claude/settings.json | jq '.hooks'

# 프로젝트 설정
cat .claude/settings.json | jq '.hooks'
```

### 훅 출력 테스트

훅 명령을 직접 테스트:

```bash
# 테스트 입력 생성
echo '{"tool_name":"Bash","tool_input":{"command":"ls"}}' | ./your-hook-script.sh
```

### 종료 코드

| 종료 코드 | 의미 |
|----------|------|
| 0 | 성공, 도구 실행 진행 |
| 1 | 오류, 경고만 표시 |
| 2 | 차단, 도구 실행 중단 |

### 일반적인 문제

| 문제 | 해결 방법 |
|------|----------|
| 훅이 실행되지 않음 | 매처 패턴 확인, 파일 경로 확인 |
| JSON 파싱 오류 | `jq` 명령 구문 확인 |
| 권한 오류 | 스크립트 실행 권한 확인 (`chmod +x`) |
| 경로 문제 | 절대 경로 또는 `$CLAUDE_PROJECT_DIR` 사용 |

---

## 빠른 참조

### 명령어

```bash
# 훅 관리 UI
/hooks

# 설정 확인
cat ~/.claude/settings.json | jq '.hooks'
```

### 기본 구조

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          { "type": "command", "command": "your-command" }
        ]
      }
    ]
  }
}
```

### 종료 코드

- `0` = 성공/진행
- `1` = 오류/경고
- `2` = 차단

---

## 관련 문서

- [훅 참조](/hooks) - 완전한 참조 문서
- [보안 고려사항](/hooks#security-considerations) - 상세 보안 가이드
- [디버깅](/hooks#debugging) - 문제 해결 단계

---

*마지막 업데이트: 2026-01*
