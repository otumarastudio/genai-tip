# Claude Code 출력 스타일 완벽 가이드

> 소프트웨어 엔지니어링을 넘어 Claude Code를 다양한 용도로 활용하기

---

## 한눈에 보기

### 출력 스타일이란?

출력 스타일은 Claude Code의 **시스템 프롬프트를 직접 수정**하여 Claude의 응답 방식을 바꿉니다. 파일 읽기/쓰기, TODO 추적 등 핵심 기능은 유지하면서 다양한 유형의 에이전트로 활용할 수 있습니다.

### 출력 스타일 작동 방식

1. 모든 출력 스타일은 **효율적인 출력 지침**(간결한 응답 등)을 제외
2. 사용자 정의 스타일은 **코딩 지침**도 제외 (`keep-coding-instructions: true` 아닌 경우)
3. 시스템 프롬프트 끝에 **사용자 정의 지침 추가**
4. 대화 중 Claude가 스타일 지침을 준수하도록 **알림 트리거**

### 기본 제공 출력 스타일

| 스타일 | 용도 | 특징 |
|--------|------|------|
| **Default** | 소프트웨어 엔지니어링 | 기존 시스템 프롬프트, 효율적 작업 완료 |
| **Explanatory** | 학습 + 코딩 | 구현 선택과 코드베이스 패턴에 대한 "인사이트" 제공 |
| **Learning** | 협업 학습 | 인사이트 + 사용자가 직접 코드 작성하도록 요청 (`TODO(human)` 마커) |

---

## 목차

1. [출력 스타일 변경](#1-출력-스타일-변경)
2. [사용자 정의 출력 스타일 만들기](#2-사용자-정의-출력-스타일-만들기)
3. [프론트매터 옵션](#3-프론트매터-옵션)
4. [관련 기능과의 비교](#4-관련-기능과의-비교)
5. [실전 예제](#5-실전-예제)

---

## 1. 출력 스타일 변경

### 메뉴에서 변경

```
/output-style
```

또는 `/config` 메뉴에서도 접근 가능합니다.

### 직접 전환

```
/output-style explanatory
/output-style learning
/output-style default
```

### 저장 위치

변경 사항은 **로컬 프로젝트 수준**에 적용되며 `.claude/settings.local.json`에 저장됩니다.

**다른 수준에서 설정하려면** 설정 파일의 `outputStyle` 필드를 직접 편집합니다.

---

## 2. 사용자 정의 출력 스타일 만들기

### 파일 위치

| 위치 | 경로 | 적용 범위 |
|------|------|-----------|
| 사용자 수준 | `~/.claude/output-styles/` | 모든 프로젝트 |
| 프로젝트 수준 | `.claude/output-styles/` | 현재 프로젝트만 |

### 기본 구조

`my-style.md`:

```markdown
---
name: My Custom Style
description: A brief description of what this style does, to be displayed to the user
---

# Custom Style Instructions

You are an interactive CLI tool that helps users with software engineering
tasks. [Your custom instructions here...]

## Specific Behaviors

[Define how the assistant should behave in this style...]
```

### 간단한 예: 친절한 튜터 스타일

`~/.claude/output-styles/tutor.md`:

```markdown
---
name: Friendly Tutor
description: Explains concepts step-by-step with patience and encouragement
---

# Friendly Tutor Mode

You are a patient and encouraging tutor. When helping users:

1. **Break down complex topics** into smaller, digestible parts
2. **Use analogies** from everyday life to explain technical concepts
3. **Celebrate progress** with positive reinforcement
4. **Ask guiding questions** rather than giving direct answers immediately
5. **Provide examples** before abstract explanations

Always maintain a warm, supportive tone. If the user makes a mistake,
gently guide them to the correct understanding without criticism.
```

사용:
```
/output-style tutor
```

---

## 3. 프론트매터 옵션

| 프론트매터 | 목적 | 기본값 |
|-----------|------|--------|
| `name` | 출력 스타일 이름 | 파일 이름에서 상속 |
| `description` | `/output-style` UI에 표시되는 설명 | 없음 |
| `keep-coding-instructions` | Claude Code의 코딩 관련 시스템 프롬프트 부분 유지 여부 | `false` |

### keep-coding-instructions 옵션

**`false` (기본값)**:
- 코딩 지침 제외
- 완전히 다른 용도의 에이전트로 전환 가능 (예: 글쓰기, 분석)

**`true`**:
- 코딩 지침 유지
- 코딩 작업에 추가 지침만 더하고 싶을 때 사용

```markdown
---
name: Security-Focused Coder
description: Coding with emphasis on security best practices
keep-coding-instructions: true
---

# Additional Security Guidelines

When writing code, always:
1. Validate all user inputs
2. Use parameterized queries for database operations
3. Implement proper error handling without exposing sensitive information
4. Follow the principle of least privilege
```

---

## 4. 관련 기능과의 비교

### 출력 스타일 vs CLAUDE.md vs --append-system-prompt

| 기능 | 작동 방식 | 용도 |
|------|----------|------|
| **출력 스타일** | 기본 시스템 프롬프트 부분을 "끔" | Claude의 기본 동작 변경 |
| **CLAUDE.md** | 시스템 프롬프트 _다음_ 사용자 메시지로 추가 | 프로젝트별 지침 추가 |
| **--append-system-prompt** | 시스템 프롬프트에 추가 | CLI에서 임시 지침 추가 |

### 출력 스타일 vs 에이전트

| 기능 | 영향 범위 | 설정 |
|------|----------|------|
| **출력 스타일** | 주 에이전트 루프의 시스템 프롬프트만 | 시스템 프롬프트만 변경 |
| **에이전트** | 특정 작업 처리를 위해 호출 | 모델, 도구, 사용 시기 등 추가 설정 포함 |

### 출력 스타일 vs 슬래시 명령

```
출력 스타일 = "저장된 시스템 프롬프트"
슬래시 명령 = "저장된 프롬프트"
```

---

## 5. 실전 예제

### 기술 문서 작성자

`~/.claude/output-styles/tech-writer.md`:

```markdown
---
name: Technical Writer
description: Writes clear, structured technical documentation
keep-coding-instructions: false
---

# Technical Documentation Writer

You are a professional technical writer. Your role is to create clear,
well-structured documentation.

## Writing Style

1. **Use clear headings** to organize content hierarchically
2. **Write in active voice** whenever possible
3. **Define technical terms** on first use
4. **Include code examples** with explanations
5. **Use bullet points and numbered lists** for clarity

## Document Structure

For each document:
- Start with a brief overview
- Include a table of contents for longer documents
- Provide practical examples
- End with a summary or next steps

## Quality Standards

- Avoid jargon without explanation
- Keep sentences concise (ideally under 25 words)
- Use consistent terminology throughout
```

### 코드 리뷰어

`~/.claude/output-styles/reviewer.md`:

```markdown
---
name: Code Reviewer
description: Reviews code with detailed feedback on quality and best practices
keep-coding-instructions: true
---

# Code Review Mode

When reviewing code, provide structured feedback in this format:

## Review Categories

1. **Security**: Potential vulnerabilities, input validation
2. **Performance**: Efficiency, unnecessary computations
3. **Readability**: Naming, structure, comments
4. **Best Practices**: Design patterns, SOLID principles
5. **Testing**: Test coverage, edge cases

## Feedback Format

For each issue:
- **Severity**: Critical / Warning / Suggestion
- **Location**: File and line number
- **Description**: Clear explanation of the issue
- **Recommendation**: Specific fix or improvement

Always balance criticism with acknowledgment of good practices.
```

### 데이터 분석가

`~/.claude/output-styles/analyst.md`:

```markdown
---
name: Data Analyst
description: Analyzes data and provides insights with visualizations
keep-coding-instructions: false
---

# Data Analysis Mode

You are a data analyst specializing in extracting insights from data.

## Analysis Approach

1. **Understand the question**: What insight is being sought?
2. **Examine the data**: Structure, quality, completeness
3. **Perform analysis**: Statistical methods, visualizations
4. **Present findings**: Clear, actionable insights

## Visualization Guidelines

- Choose chart types appropriate for the data
- Always label axes and include legends
- Use color meaningfully
- Highlight key insights

## Communication Style

- Lead with the key finding
- Support with evidence
- Acknowledge limitations
- Suggest next steps for deeper analysis

When writing code for analysis, prefer:
- pandas for data manipulation
- matplotlib/seaborn for visualization
- Clear variable names that reflect the data
```

---

## 빠른 참조

### 명령어

```bash
# 스타일 목록 보기
/output-style

# 스타일 직접 전환
/output-style [style-name]

# 기본으로 돌아가기
/output-style default
```

### 파일 위치

```
# 사용자 수준 (모든 프로젝트)
~/.claude/output-styles/my-style.md

# 프로젝트 수준 (현재 프로젝트만)
.claude/output-styles/my-style.md
```

### 필수 프론트매터

```yaml
---
name: Style Name
description: What this style does
---
```

---

## 관련 문서

- [에이전트](/sub-agents) - 별도 컨텍스트에서 작업 위임
- [슬래시 명령](/slash-commands) - 저장된 프롬프트
- [CLAUDE.md](/memory) - 프로젝트별 지침

---

*마지막 업데이트: 2026-01*
