# Opencode 최적화 방법론 가이드

> CC-System(Claude Code 중심)을 Opencode 중심으로 마이그레이션

---

## 🔑 핵심 차이점: Claude Code vs Opencode

| 기능 | Claude Code | Opencode |
|------|-------------|----------|
| **라이선스** | 상용 (Anthropic) | 오픈소스 (100K+ GitHub Stars) |
| **설정 폰더** | `.claude/` | `.opencode/` 또는 프로젝트 루트 |
| **스킬 시스템** | SKILL.md (자동 로드) | AGENTS.md + 문서 기반 |
| **에이전트** | subagents (자동 위임) | session/agent 기반 |
| **훅** | hooks (이벤트 기반) | 미지원 또는 확장 필요 |
| **슬래시 커맨드** | `.claude/commands/` | 기본 제공 + 커스텀 가능 |
| **LSP 지원** | 제한적 | 자동 LSP 로드 |
| **멀티 세션** | 제한적 | 다중 세션 병렬 실행 가능 |
| **모델 선택** | Claude만 | 75+ 제공자 (Claude, GPT, Gemini, 로컬) |

---

## 📁 권장 디렉토리 구조 (Opencode 버전)

```
cc-system-opencode/              ← 프로젝트 루트
│
├── .opencode/                   ← Opencode 전용 설정
│   ├── agents/                  ← AI 에이전트 정의
│   │   ├── code-reviewer.md
│   │   ├── bug-finder.md
│   │   └── doc-writer.md
│   ├── skills/                  ← 스킬 문서 (컨텍스트용)
│   │   ├── python-best-practices.md
│   │   ├── api-integration.md
│   │   └── testing-guide.md
│   ├── prompts/                 ← 재사용 가능한 프롬프트
│   │   ├── commit-message.txt
│   │   ├── code-review.txt
│   │   └── refactor.txt
│   └── sessions/                ← 세션 템플릿
│       ├── web-development.json
│       └── data-analysis.json
│
├── docs/                        ← 문서
│   ├── opencode/               ← Opencode 특화 문서
│   │   ├── agents.md
│   │   ├── skills.md
│   │   └── workflows.md
│   └── guides/                 ← 사용자 가이드
│       ├── getting-started.md
│       └── best-practices.md
│
├── scripts/                     ← 유틸리티 스크립트
│   ├── setup.sh
│   ├── validate.py
│   └── init-agent.py
│
├── AGENTS.md                    ← Opencode 표준 (필수!)
├── opencode.config.js          ← Opencode 설정 (선택)
├── package.json                ← Node.js 프로젝트 (있는 경우)
└── README.md
```

---

## 🔄 마이그레이션 변환표

### 1. Skill → Skill 문서

**Claude Code 방식:**
```
.claude/skills/skill-name/
├── SKILL.md
├── scripts/
└── references/
```

**Opencode 방식:**
```
.opencode/skills/
├── skill-name.md          ← 통합된 문서
└── scripts/               ← (별도 폰더로 분리 가능)
    └── skill-name/
        └── helper.py
```

**변환 예시:**

*AS-IS (Claude)*:
```markdown
---
name: python-helper
description: Python 개발 도우미
---

# Python Helper

## 파일 생성하기

```python
# 코드 예시
```
```

*TO-BE (Opencode)*:
```markdown
# Python 개발 스킬

## 역할
Python 코드 작성, 리팩토링, 디버깅을 도와줍니다.

## 사용 시나리오
- 새로운 Python 파일을 생성할 때
- 코드 리팩토링이 필요할 때
- 버그를 찾고 수정할 때

## 가이드라인
1. 타입 힌트를 항상 사용하세요
2. Docstring은 Google 스타일을 따르세요
3. pathlib를 사용하여 경로를 처리하세요

## 예시

### 파일 생성
```python
from pathlib import Path

def create_module(name: str) -> None:
    \"\"\"새로운 Python 모듈을 생성합니다.\"\"\"
    Path(f"{name}.py").touch()
```

### 리팩토링
- 함수가 20줄 이상이면 분리하세요
- 중복 코드는 헬퍼 함수로 추출하세요
```
```

---

### 2. Agent → Agent 문서

**Claude Code 방식:**
```markdown
---
name: code-reviewer
description: 코드 리뷰어
tools: Read, Grep, Bash
model: sonnet
---

System prompt here...
```

**Opencode 방식:**
```markdown
# 코드 리뷰어 에이전트

## 역할
당신은 시니어 코드 리뷰어입니다. 코드 품질과 보안을 검사합니다.

## 사용 시점
- 코드 작성이 완료되었을 때
- PR을 생성하기 전
- 리팩토링 후 검증이 필요할 때

## 접근 방식
1. 변경된 파일을 확인하세요 (`git diff`)
2. 파일을 읽고 분석하세요
3. 다음 항목을 체크하세요:
   - [ ] 코드 가독성
   - [ ] 보안 취약점
   - [ ] 에러 처리
   - [ ] 테스트 커버리지

## 출력 형식
### 🔴 Critical
- 문제점과 수정 방법

### 🟡 Warning
- 개선 권장사항

### 🟢 Suggestion
- 선택적 개선사항
```

---

### 3. Slash Command → 프롬프트 템플릿

**Claude Code 방식:**
```markdown
---
description: 코드 리뷰
---

Review this code for bugs and suggest improvements.
```

**Opencode 방식:**
사용자가 직접 `/review`를 타이핑하거나, `AGENTS.md`에 단축키를 정의:

```markdown
## 단축 명령어

| 명령어 | 동작 |
|--------|------|
| `/review` | 코드 리뷰 요청: "이 코드를 리뷰하고 버그와 개선사항을 알려줘" |
| `/refactor` | 리팩토링 요청: "이 코드를 더 깔끔하게 리팩토링해줘" |
| `/test` | 테스트 생성: "이 함수에 대한 단위 테스트를 작성해줘" |
```

---

### 4. Hooks → Git Hooks + 스크립트

**Claude Code:** 내장 훅 시스템 (PreToolUse, PostToolUse 등)

**Opencode:** 직접 Git hooks + 유틸리티 스크립트로 구현

```
├── .git/hooks/              ← Git 훅
│   ├── pre-commit          ← 커밋 전 실행
│   └── post-checkout       ← 브랜치 전환 후 실행
│
└── scripts/hooks/          ← Opencode 연동 훅
    ├── pre-edit.py         ← 파일 수정 전
    ├── post-edit.py        ← 파일 수정 후
    └── auto-format.sh      ← 자동 포맷팅
```

**예시: 자동 포맷팅 훅 (scripts/hooks/post-edit.py)**
```python
#!/usr/bin/env python3
"""파일 수정 후 자동으로 포맷팅을 실행합니다."""

import sys
import subprocess
from pathlib import Path

def main():
    file_path = sys.argv[1] if len(sys.argv) > 1 else None
    if not file_path:
        return
    
    path = Path(file_path)
    
    # Python 파일 자동 포맷팅
    if path.suffix == '.py':
        subprocess.run(['black', str(path)], capture_output=True)
        print(f"✅ Formatted {path.name} with black")
    
    # JavaScript/TypeScript 파일
    elif path.suffix in ['.js', '.ts', '.tsx']:
        subprocess.run(['prettier', '--write', str(path)], capture_output=True)
        print(f"✅ Formatted {path.name} with prettier")

if __name__ == "__main__":
    main()
```

---

## 🛠️ Opencode용 AGENTS.md 작성법

Opencode는 Claude Code와 동일하게 `AGENTS.md` 파일을 사용합니다:

```markdown
# AGENTS.md - Opencode Project Guidelines

## Project Overview

이 프로젝트는 [설명]을 위한 [기술 스택] 프로젝트입니다.

## Build/Test/Lint Commands

```bash
# 개발 서버 실행
npm run dev

# 테스트 실행
npm test

# 단일 테스트 실행
npm test -- --testNamePattern="테스트이름"

# 린트 검사
npm run lint

# 타입 체크
npm run typecheck
```

## Code Style Guidelines

### Python
- **Formatter**: black (line-length: 88)
- **Linter**: ruff
- **Type hints**: 필수
- **Docstring**: Google style

### JavaScript/TypeScript
- **Formatter**: prettier
- **Linter**: eslint
- **Semicolons**: required
- **Quotes**: single

### Naming Conventions
- **Files**: kebab-case (e.g., `my-component.tsx`)
- **Functions**: camelCase (e.g., `getUserData`)
- **Classes**: PascalCase (e.g., `UserManager`)
- **Constants**: UPPER_SNAKE_CASE

## Opencode Workflow

### 1. 세션 시작
```bash
# 새 세션 시작
opencode

# 특정 작업을 위한 세션
opencode --session feature-login
```

### 2. 에이전트 사용
- 코드 리뷰가 필요하면 `.opencode/agents/code-reviewer.md` 참조
- 버그 수정이 필요하면 `.opencode/agents/bug-finder.md` 참조
- 문서 작성이 필요하면 `.opencode/agents/doc-writer.md` 참조

### 3. 스킬 활용
작업 전에 관련 `.opencode/skills/` 문서를 읽어 컨텍스트를 확보하세요.

## Communication Guidelines

모든 소통은 한국어로 진행합니다.
```

---

## 🚀 Opencode 특화 기능 활용법

### 1. 멀티 세션 활용

Opencode는 동시에 여러 세션을 실행할 수 있습니다:

```bash
# 터미널 1: 백엔드 작업
opencode --session backend-api

# 터미널 2: 프론트엔드 작업
opencode --session frontend-ui

# 터미널 3: 데이터베이스 작업
opencode --session db-migration
```

**세션 공유 링크 생성:**
```
Opencode 내부에서:
> /share

링크가 생성됩니다: https://opencode.ai/s/abc123
```

---

### 2. LSP (Language Server Protocol) 활용

Opencode는 자동으로 LSP를 로드하여 더 정확한 코드 분석을 제공합니다:

```markdown
## LSP 활용 팁

Opencode는 다음 LSP들을 자동으로 사용합니다:
- Python: pylsp 또는 pyright
- JavaScript/TypeScript: typescript-language-server
- Rust: rust-analyzer
- Go: gopls

코드 작업 시 LSP의 진단 메시지를 참고하세요.
```

---

### 3. 모델 선택 전략

Opencode는 여러 모델을 지원합니다. 작업별 최적 모델:

```markdown
## 모델 선택 가이드

| 작업 유형 | 권장 모델 | 이유 |
|-----------|-----------|------|
| 코드 작성 | Claude Sonnet | 뛰어난 코드 이해와 생성 |
| 빠른 검색 | GPT-4o-mini | 빠르고 효율적 |
| 복잡한 설계 | Claude Opus | 심층적인 추론 |
| 간단한 수정 | 로컬 모델 | 프라이버시와 비용 |

모델 변경:
> /model
```

---

### 4. GitHub Copilot 통합

Opencode는 GitHub Copilot과도 연동됩니다:

```bash
# Copilot 로그인
opencode --login copilot

# Copilot과 함께 사용
opencode --model copilot
```

---

## 📋 마이그레이션 체크리스트

### Phase 1: 구조 변환
- [ ] `.claude/` → `.opencode/` 폰더명 변경
- [ ] `SKILL.md` 파일들을 통합된 `.md` 형식으로 변환
- [ ] `agents/` 파일들을 프롬프트 형식으로 변환
- [ ] `commands/` → `prompts/` 또는 템플릿으로 변환

### Phase 2: AGENTS.md 업데이트
- [ ] Opencode용 명령어로 업데이트
- [ ] 사용 가능한 모델 목록 추가
- [ ] LSP 관련 가이드라인 추가
- [ ] 멀티 세션 활용법 추가

### Phase 3: Hooks 마이그레이션
- [ ] Git hooks로 핵심 자동화 마이그레이션
- [ ] Opencode와 연동할 스크립트 작성
- [ ] CI/CD 연동 검토

### Phase 4: 테스트 및 검증
- [ ] 주요 워크플로우 테스트
- [ ] 에이전트 응답 품질 검증
- [ ] 문서 최신화

---

## 💡 모범 사례 (Best Practices)

### 1. 문서 중심 접근
Opencode는 Claude Code와 달리 SKILL.md 같은 특별한 형식이 없습니다. 대신 **일반 Markdown 문서**를 컨텍스트로 활용합니다:

```
작업 시작 전:
1. 관련 skill 문서를 Read
2. 필요한 agent 문서를 Read
3. 작업 수행
```

### 2. 명시적 컨텍스트 관리
Opencode는 Claude Code만큼 자동으로 컨텍스트를 관리하지 않을 수 있습니다. 명시적으로 파일을 참조하세요:

```
❌ "이 파일을 수정해줘"
✅ "@src/components/Button.tsx 파일을 수정해줘"
```

### 3. 세션 기반 작업 분리
Opencode의 멀티 세션 기능을 활용하여 작업을 분리하세요:

```
세션 1: API 설계 및 구현
세션 2: UI 컴포넌트 개발
세션 3: 테스트 작성
```

### 4. 프롬프트 라이브러리 구축
자주 사용하는 프롬프트를 `.opencode/prompts/`에 저장하고 재사용하세요.

---

## 🔗 참고 자료

- [Opencode 공식 문서](https://opencode.ai/docs)
- [Opencode GitHub](https://github.com/anomalyco/opencode)
- [Claude Code → Opencode 마이그레이션 가이드](https://opencode.ai/docs/migration)

---

**요약:** Opencode는 Claude Code의 스킬/에이전트 개념을 **문서 중심**으로 단순화하고, **멀티 세션**과 **다양한 모델** 지원으로 더 유연한 워크플로우를 제공합니다. 마이그레이션의 핵심은 복잡한 구조를 단순한 Markdown 문서 중심으로 변환하는 것입니다.
