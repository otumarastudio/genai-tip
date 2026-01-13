# Claude Code 플러그인 발견 및 설치 완벽 가이드

> 마켓플레이스에서 플러그인을 찾아 설치하여 Claude Code 확장하기

---

## 한눈에 보기

### 플러그인 마켓플레이스란?

마켓플레이스는 다른 사람이 만들어 공유한 플러그인의 **카탈로그**입니다. 앱 스토어처럼 카탈로그를 등록하고, 원하는 플러그인을 개별적으로 설치합니다.

### 설치 프로세스

```
1. 마켓플레이스 추가 → 카탈로그 등록 (플러그인 미설치)
2. 플러그인 설치 → 원하는 플러그인 선택 설치
```

### 공식 마켓플레이스 플러그인 카테고리

| 카테고리 | 플러그인 예시 |
|----------|--------------|
| **코드 인텔리전스** | TypeScript, Python, Rust LSP |
| **외부 통합** | GitHub, GitLab, Jira, Slack, Figma |
| **개발 워크플로우** | commit-commands, pr-review-toolkit |
| **출력 스타일** | explanatory, learning |

---

## 목차

1. [빠른 시작: 플러그인 설치하기](#1-빠른-시작-플러그인-설치하기)
2. [마켓플레이스 추가 방법](#2-마켓플레이스-추가-방법)
3. [플러그인 설치](#3-플러그인-설치)
4. [설치된 플러그인 관리](#4-설치된-플러그인-관리)
5. [마켓플레이스 관리](#5-마켓플레이스-관리)
6. [팀 마켓플레이스 구성](#6-팀-마켓플레이스-구성)
7. [코드 인텔리전스 플러그인](#7-코드-인텔리전스-플러그인)
8. [문제 해결](#8-문제-해결)

---

## 1. 빠른 시작: 플러그인 설치하기

### 공식 마켓플레이스에서 설치

공식 Anthropic 마켓플레이스(`claude-plugins-official`)는 자동으로 사용 가능합니다.

```bash
# 플러그인 관리자 열기
/plugin

# 직접 설치
/plugin install plugin-name@claude-plugins-official
```

### 데모 마켓플레이스 추가 및 플러그인 설치

```bash
# 1. 마켓플레이스 추가
/plugin marketplace add anthropics/claude-code

# 2. 플러그인 관리자 열기
/plugin

# 3. Discover 탭에서 플러그인 찾아보기
# Tab으로 탭 이동

# 4. 플러그인 설치
/plugin install commit-commands@anthropics-claude-code

# 5. 사용하기
/commit-commands:commit
```

---

## 2. 마켓플레이스 추가 방법

### GitHub 저장소에서 추가

```bash
/plugin marketplace add owner/repo

# 예시
/plugin marketplace add anthropics/claude-code
```

### 다른 Git 호스트에서 추가

```bash
# HTTPS
/plugin marketplace add https://gitlab.com/company/plugins.git

# SSH
/plugin marketplace add git@gitlab.com:company/plugins.git

# 특정 분기/태그
/plugin marketplace add https://gitlab.com/company/plugins.git#v1.0.0
```

### 로컬 경로에서 추가

```bash
# 디렉토리
/plugin marketplace add ./my-marketplace

# 직접 파일 경로
/plugin marketplace add ./path/to/marketplace.json

# 원격 URL
/plugin marketplace add https://example.com/marketplace.json
```

> 💡 **단축키**: `/plugin marketplace` 대신 `/plugin market` 사용 가능

---

## 3. 플러그인 설치

### 기본 설치 (사용자 범위)

```bash
/plugin install plugin-name@marketplace-name
```

### 범위 선택 설치

| 범위 | 설명 | 저장 위치 |
|------|------|-----------|
| **User** | 모든 프로젝트에서 나만 사용 | 사용자 설정 |
| **Project** | 이 저장소의 모든 협력자 사용 | `.claude/settings.json` |
| **Local** | 이 저장소에서 나만 사용 | 로컬 설정 |
| **Managed** | 관리자가 설치 (수정 불가) | 관리 설정 |

**대화형 UI로 범위 선택**:
1. `/plugin` 실행
2. **Discover** 탭으로 이동
3. 플러그인 선택 후 **Enter**
4. 범위 선택

**CLI로 범위 지정**:
```bash
claude plugin install formatter@your-org --scope project
```

> ⚠️ **주의**: 플러그인 설치 전 신뢰할 수 있는지 확인하세요. Anthropic은 타사 플러그인의 보안을 보장하지 않습니다.

---

## 4. 설치된 플러그인 관리

### 플러그인 관리자 UI

```bash
/plugin
```

**탭 구성**:
- **Discover**: 사용 가능한 플러그인 찾아보기
- **Installed**: 설치된 플러그인 관리
- **Marketplaces**: 마켓플레이스 관리
- **Errors**: 로딩 오류 확인

### CLI 명령어

```bash
# 플러그인 비활성화 (제거하지 않음)
/plugin disable plugin-name@marketplace-name

# 플러그인 다시 활성화
/plugin enable plugin-name@marketplace-name

# 플러그인 완전히 제거
/plugin uninstall plugin-name@marketplace-name

# 특정 범위에서 제거
claude plugin uninstall formatter@your-org --scope project
```

---

## 5. 마켓플레이스 관리

### 대화형 인터페이스

```bash
/plugin
# Marketplaces 탭으로 이동
```

### CLI 명령어

```bash
# 모든 마켓플레이스 나열
/plugin marketplace list

# 플러그인 목록 새로 고침
/plugin marketplace update marketplace-name

# 마켓플레이스 제거
/plugin marketplace remove marketplace-name
```

> ⚠️ 마켓플레이스를 제거하면 해당 마켓플레이스에서 설치한 모든 플러그인이 제거됩니다.

### 자동 업데이트 구성

**UI에서 설정**:
1. `/plugin` → **Marketplaces**
2. 마켓플레이스 선택
3. **Enable/Disable auto-update** 선택

**기본 설정**:
- 공식 Anthropic 마켓플레이스: 자동 업데이트 활성화
- 타사/로컬 마켓플레이스: 자동 업데이트 비활성화

**환경 변수로 제어**:
```bash
# 모든 자동 업데이트 비활성화
export DISABLE_AUTOUPDATER=true

# Claude Code 업데이트만 비활성화, 플러그인 업데이트는 유지
export DISABLE_AUTOUPDATER=true
export FORCE_AUTOUPDATE_PLUGINS=true
```

---

## 6. 팀 마켓플레이스 구성

프로젝트의 `.claude/settings.json`에 마켓플레이스 구성을 추가하면 팀 전체에서 사용할 수 있습니다.

```json
{
  "extraKnownMarketplaces": ["company/internal-plugins"],
  "enabledPlugins": ["formatter@company-internal-plugins"]
}
```

팀 멤버가 저장소 폴더를 신뢰하면 Claude Code가 자동으로 설치를 요청합니다.

---

## 7. 코드 인텔리전스 플러그인

코드 인텔리전스 플러그인은 LSP(Language Server Protocol)를 사용하여 Claude에 실시간 코드 이해 기능을 제공합니다.

### 사용 가능한 플러그인

| 언어 | 플러그인 | 필요한 바이너리 |
|------|----------|----------------|
| C/C++ | `clangd-lsp` | `clangd` |
| C# | `csharp-lsp` | `csharp-ls` |
| Go | `gopls-lsp` | `gopls` |
| Java | `jdtls-lsp` | `jdtls` |
| Lua | `lua-lsp` | `lua-language-server` |
| PHP | `php-lsp` | `intelephense` |
| Python | `pyright-lsp` | `pyright-langserver` |
| Rust | `rust-analyzer-lsp` | `rust-analyzer` |
| Swift | `swift-lsp` | `sourcekit-lsp` |
| TypeScript | `typescript-lsp` | `typescript-language-server` |

### 설치 방법

```bash
# 1. 플러그인 설치
/plugin install typescript-lsp@claude-plugins-official

# 2. 바이너리 설치 (시스템에 따라 다름)
npm install -g typescript-language-server typescript
```

> 💡 "Executable not found in $PATH" 오류가 발생하면 해당 바이너리를 설치하세요.

---

## 8. 문제 해결

### /plugin 명령이 인식되지 않음

1. **버전 확인**: `claude --version` (1.0.33 이상 필요)
2. **업데이트**:
   - Homebrew: `brew upgrade claude-code`
   - npm: `npm update -g @anthropic-ai/claude-code`
3. **재시작**: 터미널 재시작 후 `claude` 다시 실행

### 일반적인 문제

| 문제 | 해결 방법 |
|------|----------|
| 마켓플레이스가 로드되지 않음 | URL 접근 가능 여부 확인, `.claude-plugin/marketplace.json` 경로 확인 |
| 플러그인 설치 실패 | 저장소 URL 접근 권한 확인 |
| 설치 후 파일 찾을 수 없음 | 플러그인은 캐시에 복사됨, 외부 파일 참조 불가 |
| 스킬이 나타나지 않음 | 캐시 지우기: `rm -rf ~/.claude/plugins/cache` |

### 스킬이 나타나지 않을 때

```bash
# 1. 캐시 지우기
rm -rf ~/.claude/plugins/cache

# 2. Claude Code 재시작

# 3. 플러그인 재설치
/plugin install plugin-name@marketplace-name
```

---

## 공식 마켓플레이스 플러그인 상세

### 외부 통합 플러그인

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

### 개발 워크플로우 플러그인

| 플러그인 | 용도 |
|----------|------|
| `commit-commands` | Git 커밋, 푸시, PR 생성 |
| `pr-review-toolkit` | PR 검토 전문 에이전트 |
| `agent-sdk-dev` | Claude Agent SDK 개발 도구 |
| `plugin-dev` | 플러그인 개발 도구 |

### 출력 스타일 플러그인

| 플러그인 | 용도 |
|----------|------|
| `explanatory-output-style` | 구현 선택에 대한 교육적 인사이트 |
| `learning-output-style` | 대화형 학습 모드 |

---

## 관련 문서

- [플러그인 만들기](/plugins) - 사용자 정의 플러그인 개발
- [플러그인 마켓플레이스 만들기](/plugin-marketplaces) - 마켓플레이스 배포
- [플러그인 참조](/plugins-reference) - 완전한 기술 사양

---

*마지막 업데이트: 2026-01*
