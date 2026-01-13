# Claude Code Agent Skills 완벽 가이드

> Claude에게 특정 작업을 수행하는 방법을 가르치는 스킬 생성 및 활용법

---

## 한눈에 보기

### 스킬이란?

스킬(Skill)은 Claude에게 **특정 작업을 수행하는 방법**을 가르치는 마크다운 파일입니다.

- 팀 표준에 맞춰 PR 검토
- 선호하는 형식으로 커밋 메시지 생성
- 회사 데이터베이스 스키마 쿼리

### 스킬 vs 다른 기능

| 기능 | 용도 | 실행 시기 |
|------|------|-----------|
| **Skills** | Claude에게 전문 지식 제공 | Claude가 관련성 있을 때 자동 선택 |
| **슬래시 명령** | 재사용 가능한 프롬프트 | `/command` 입력 시 실행 |
| **CLAUDE.md** | 프로젝트 전체 지침 | 모든 대화에 자동 로드 |
| **Subagents** | 별도 컨텍스트에서 작업 위임 | Claude가 위임하거나 명시적 호출 |
| **Hooks** | 이벤트에서 스크립트 실행 | 특정 도구 이벤트에서 발생 |
| **MCP** | 외부 도구/데이터 소스 연결 | Claude가 필요시 호출 |

### 핵심 포인트

- **모델 호출**: 사용자가 명시적으로 호출하지 않아도 Claude가 요청에 따라 자동으로 적용
- **점진적 공개**: 필수 정보만 SKILL.md에, 상세 참조는 별도 파일로 분리
- **도구 제한**: `allowed-tools`로 스킬 활성화 시 사용 가능한 도구 제한 가능

---

## 목차

1. [빠른 시작: 첫 번째 스킬 만들기](#1-빠른-시작-첫-번째-스킬-만들기)
2. [스킬 작동 방식](#2-스킬-작동-방식)
3. [SKILL.md 작성](#3-skillmd-작성)
4. [점진적 공개와 지원 파일](#4-점진적-공개와-지원-파일)
5. [도구 액세스 제한](#5-도구-액세스-제한)
6. [고급 설정](#6-고급-설정)
7. [실전 예제](#7-실전-예제)
8. [문제 해결](#8-문제-해결)

---

## 1. 빠른 시작: 첫 번째 스킬 만들기

시각적 다이어그램과 유추로 코드를 설명하는 스킬을 만들어봅니다.

### 단계 1: 사용 가능한 스킬 확인

```
What Skills are available?
```

### 단계 2: 스킬 디렉토리 생성

```bash
mkdir -p ~/.claude/skills/explaining-code
```

### 단계 3: SKILL.md 작성

`~/.claude/skills/explaining-code/SKILL.md`:

```yaml
---
name: explaining-code
description: Explains code with visual diagrams and analogies. Use when explaining how code works, teaching about a codebase, or when the user asks "how does this work?"
---

When explaining code, always include:

1. **Start with an analogy**: Compare the code to something from everyday life
2. **Draw a diagram**: Use ASCII art to show the flow, structure, or relationships
3. **Walk through the code**: Explain step-by-step what happens
4. **Highlight a gotcha**: What's a common mistake or misconception?

Keep explanations conversational. For complex concepts, use multiple analogies.
```

### 단계 4: 스킬 확인

```
What Skills are available?
```

`explaining-code`가 목록에 나타나야 합니다.

### 단계 5: 스킬 테스트

파일을 열고 질문:

```
How does this code work?
```

Claude가 유추와 ASCII 다이어그램을 포함한 설명을 제공합니다.

---

## 2. 스킬 작동 방식

### 라이프사이클

```
1. 발견 → 시작 시 이름과 설명만 로드 (빠른 시작)
2. 활성화 → 요청이 설명과 일치하면 전체 SKILL.md 로드
3. 실행 → 지침 따르기, 필요시 참조 파일 로드/스크립트 실행
```

### 스킬 저장 위치

| 위치 | 경로 | 적용 대상 | 우선순위 |
|------|------|-----------|----------|
| Enterprise | 관리 설정 참조 | 조직 전체 | 1 (최고) |
| Personal | `~/.claude/skills/` | 모든 프로젝트 | 2 |
| Project | `.claude/skills/` | 이 저장소의 모든 사람 | 3 |
| Plugin | 플러그인과 함께 번들 | 플러그인 설치된 곳 | 4 (최저) |

> 💡 같은 이름의 스킬은 우선순위가 높은 위치가 우선합니다.

---

## 3. SKILL.md 작성

### 기본 구조

```yaml
---
name: your-skill-name
description: Brief description of what this Skill does and when to use it
---

# Your Skill Name

## Instructions
Provide clear, step-by-step guidance for Claude.

## Examples
Show concrete examples of using this Skill.
```

### 메타데이터 필드

| 필드 | 필수 | 설명 |
|------|------|------|
| `name` | ✅ | 스킬 이름 (소문자, 숫자, 하이픈, 최대 64자) |
| `description` | ✅ | 스킬 용도와 사용 시기 (최대 1024자) |
| `allowed-tools` | ❌ | 스킬 활성화 시 사용 가능한 도구 목록 |
| `model` | ❌ | 사용할 모델 (예: `claude-sonnet-4-20250514`) |
| `context` | ❌ | `fork`: 별도 서브에이전트 컨텍스트에서 실행 |
| `agent` | ❌ | `context: fork` 시 사용할 에이전트 유형 |
| `hooks` | ❌ | 스킬 라이프사이클 훅 정의 |
| `user-invocable` | ❌ | `false`: 슬래시 메뉴에서 숨김 (기본: `true`) |
| `disable-model-invocation` | ❌ | `true`: Claude의 프로그래밍 방식 호출 차단 |

### description 작성 팁

`description`은 Claude가 스킬을 사용할지 결정하는 핵심입니다.

**좋은 예**:
```yaml
description: Extract text and tables from PDF files, fill forms, merge documents. Use when working with PDF files or when the user mentions PDFs, forms, or document extraction.
```

**나쁜 예**:
```yaml
description: Helps with documents
```

**좋은 설명이 답해야 할 질문**:
1. 이 스킬은 무엇을 합니까? (특정 기능 나열)
2. Claude는 언제 사용해야 합니까? (트리거 용어 포함)

---

## 4. 점진적 공개와 지원 파일

### 왜 필요한가?

스킬은 컨텍스트 윈도우를 공유합니다. **점진적 공개**를 사용하면:
- SKILL.md는 간결하게 유지 (500줄 이하 권장)
- 상세한 참조는 별도 파일로 분리
- Claude가 필요할 때만 추가 파일 로드

### 다중 파일 구조 예시

```
my-skill/
├── SKILL.md              # 필수 - 개요와 네비게이션
├── reference.md          # 상세 API 문서 - 필요시 로드
├── examples.md           # 사용 예제 - 필요시 로드
└── scripts/
    └── helper.py         # 유틸리티 스크립트 - 읽지 않고 실행
```

### SKILL.md에서 참조하기

````markdown
## Overview

[Essential instructions here]

## Additional resources

- For complete API details, see [reference.md](reference.md)
- For usage examples, see [examples.md](examples.md)

## Utility scripts

To validate input files, run the helper script:
```bash
python scripts/helper.py input.txt
```
````

### 유틸리티 스크립트 활용

스크립트는 **읽지 않고 실행**하여 컨텍스트를 절약합니다:

```markdown
Run the validation script to check the form:
python scripts/validate_form.py input.pdf
```

**스크립트 활용이 좋은 경우**:
- 복잡한 검증 로직
- 데이터 처리
- 일관성이 필요한 반복 작업

> 💡 참조를 한 수준 깊이로 유지하세요. 깊은 중첩은 Claude가 파일을 부분적으로만 읽을 수 있습니다.

---

## 5. 도구 액세스 제한

### allowed-tools 사용

스킬 활성화 시 Claude가 사용할 수 있는 도구를 제한합니다.

**쉼표 구분 방식**:
```yaml
---
name: reading-files-safely
description: Read files without making changes. Use when you need read-only file access.
allowed-tools: Read, Grep, Glob
---
```

**YAML 목록 방식**:
```yaml
---
name: reading-files-safely
description: Read files without making changes. Use when you need read-only file access.
allowed-tools:
  - Read
  - Grep
  - Glob
---
```

### 활용 사례

| 용도 | 허용 도구 |
|------|-----------|
| 읽기 전용 분석 | `Read, Grep, Glob` |
| 데이터 분석만 | `Bash(python:*)` |
| 파일 수정 포함 | `Read, Edit, Write` |

---

## 6. 고급 설정

### 포크된 컨텍스트에서 실행

복잡한 다단계 작업을 주 대화와 분리하여 실행:

```yaml
---
name: code-analysis
description: Analyze code quality and generate detailed reports
context: fork
agent: general-purpose
---
```

### 스킬 훅 정의

```yaml
---
name: secure-operations
description: Perform operations with additional security checks
hooks:
  PreToolUse:
    - matcher: "Bash"
      hooks:
        - type: command
          command: "./scripts/security-check.sh $TOOL_INPUT"
          once: true
---
```

**지원 훅 이벤트**:
- `PreToolUse`: 도구 사용 전
- `PostToolUse`: 도구 사용 후
- `Stop`: 스킬 완료 시

> `once: true`는 세션당 한 번만 실행합니다.

### 스킬 가시성 제어

| 설정 | 슬래시 메뉴 | Skill 도구 | 자동 발견 |
|------|------------|-----------|----------|
| `user-invocable: true` (기본) | 표시 | 허용 | 예 |
| `user-invocable: false` | 숨김 | 허용 | 예 |
| `disable-model-invocation: true` | 표시 | 차단 | 예 |

```yaml
---
name: internal-review-standards
description: Apply internal code review standards when reviewing pull requests
user-invocable: false
---
```

### 서브에이전트에 스킬 제공

서브에이전트는 스킬을 자동 상속하지 않습니다. 명시적으로 지정해야 합니다:

```yaml
# .claude/agents/code-reviewer.md
---
name: code-reviewer
description: Review code for quality and best practices
skills: pr-review, security-check
---
```

> 기본 제공 에이전트(Explore, Plan, general-purpose)는 스킬에 액세스할 수 없습니다.

---

## 7. 실전 예제

### 단일 파일 스킬: 커밋 메시지 생성

```
commit-helper/
└── SKILL.md
```

```yaml
---
name: generating-commit-messages
description: Generates clear commit messages from git diffs. Use when writing commit messages or reviewing staged changes.
---

# Generating Commit Messages

## Instructions

1. Run `git diff --staged` to see changes
2. I'll suggest a commit message with:
   - Summary under 50 characters
   - Detailed description
   - Affected components

## Best practices

- Use present tense
- Explain what and why, not how
```

### 다중 파일 스킬: PDF 처리

```
pdf-processing/
├── SKILL.md              # 개요와 빠른 시작
├── FORMS.md              # 양식 필드 매핑
├── REFERENCE.md          # pypdf/pdfplumber API 상세
└── scripts/
    ├── fill_form.py      # 양식 채우기 유틸리티
    └── validate.py       # PDF 유효성 검사
```

**SKILL.md**:

````yaml
---
name: pdf-processing
description: Extract text, fill forms, merge PDFs. Use when working with PDF files, forms, or document extraction. Requires pypdf and pdfplumber packages.
allowed-tools: Read, Bash(python:*)
---

# PDF Processing

## Quick start

Extract text:
```python
import pdfplumber
with pdfplumber.open("doc.pdf") as pdf:
    text = pdf.pages[0].extract_text()
```

For form filling, see [FORMS.md](FORMS.md).
For detailed API reference, see [REFERENCE.md](REFERENCE.md).

## Requirements

Packages must be installed in your environment:
```bash
pip install pypdf pdfplumber
```
````

---

## 8. 문제 해결

### 스킬 확인 방법

```
What Skills are available?
```

### 스킬이 트리거되지 않음

**description 개선**:
```yaml
# 나쁜 예
description: 문서에 도움이 됨

# 좋은 예
description: Extract text and tables from PDF files, fill forms, merge documents. Use when working with PDF files or when the user mentions PDFs, forms, or document extraction.
```

### 스킬이 로드되지 않음

**파일 경로 확인**:

| 유형 | 경로 |
|------|------|
| Personal | `~/.claude/skills/my-skill/SKILL.md` |
| Project | `.claude/skills/my-skill/SKILL.md` |
| Plugin | 플러그인 디렉토리 내 `skills/my-skill/SKILL.md` |

**YAML 구문 확인**:
- 프론트매터는 1줄에서 `---`로 시작
- 마크다운 콘텐츠 앞에 `---`로 끝
- 들여쓰기에 공백 사용 (탭 아님)

**디버그 모드**:
```bash
claude --debug
```

### 스킬에 오류가 있음

1. **의존성 설치 확인**: 외부 패키지가 환경에 설치되어 있는지
2. **스크립트 권한 확인**: `chmod +x scripts/*.py`
3. **파일 경로 확인**: 슬래시(Unix 스타일) 사용

### 여러 스킬 충돌

설명이 너무 유사할 때 발생합니다. 구별되는 트리거 용어 사용:

```yaml
# 스킬 1
description: Excel 파일 및 CRM 내보내기의 판매 데이터 분석

# 스킬 2
description: 로그 파일 및 시스템 메트릭 분석
```

### 플러그인 스킬이 나타나지 않음

```bash
# 캐시 지우기
rm -rf ~/.claude/plugins/cache

# Claude Code 재시작

# 플러그인 재설치
/plugin install plugin-name@marketplace-name
```

---

## 스킬 배포 방법

| 방법 | 설명 |
|------|------|
| **프로젝트 스킬** | `.claude/skills/`를 버전 제어에 커밋 |
| **플러그인** | `skills/` 디렉토리 포함하여 마켓플레이스 배포 |
| **관리** | 관리 설정을 통해 조직 전체 배포 |

---

## 관련 문서

- [작성 모범 사례](https://docs.claude.com/docs/agents-and-tools/agent-skills/best-practices)
- [Agent Skills 개요](https://docs.claude.com/docs/agents-and-tools/agent-skills/overview)
- [Agent SDK에서 스킬 사용](https://docs.claude.com/docs/agent-sdk/skills)

---

*마지막 업데이트: 2026-01*
