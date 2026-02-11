# AGENTS.md - AI 작업 지침서

> 이 저장소에서 일하는 AI를 위한 규칙과 가이드

---

## 📌 프로젝트 소개

이 저장소는 **클로드(Claude)를 더 똑똑하게 만드는 도구 모음**이에요.

**구성 요소:**
- **스킬(Skills)** `.claude/skills/` - 클로드에게 가르치는 "교재"
- **에이전트(Agents)** `.claude/agents/` - 특정 일을 하는 "전문 AI 직원"
- **문서(Docs)** `docs/` - 사용 설명서

---

## 🗣️ 소통 규칙 (매우 중요!)

**모든 소통은 한국어로 합니다.**
- 사용자에게 답할 때: 한국어
- 깃 커밋 메시지: 한국어
- 코드 주석: 한국어
- 에러 메시지: 한국어
- 스크립트 출력: 한국어

---

## 🛠️ 명령어 모음

### 파이썬 스크립트 실행하기

```bash
# 기본 실행법
python3 .claude/skills/<스킬이름>/scripts/<스크립트>.py

# 새 스킬 만들기
python3 .claude/skills/skill-creator/scripts/init_skill.py 내스킬 --path .claude/skills

# 스킬 포장하기 (배포용)
python3 .claude/skills/skill-creator/scripts/package_skill.py .claude/skills/내스킬

# 새 슬래시 명령어 만들기
python3 .claude/skills/slash-command-creator/scripts/init_command.py 내커맨드
```

### 검사하기

```bash
# 스킬 구조가 올바른지 확인
python3 .claude/skills/skill-creator/scripts/quick_validate.py .claude/skills/<스킬이름>
```

---

## ✍️ 코드 작성 규칙

### 파이썬 코드

| 항목 | 규칙 |
|------|------|
| **시작 문구** | `#!/usr/bin/env python3` 필수 |
| **문서화** | 함수 위에 설명 달기 (Google 스타일) |
| **에러 처리** | try/except로 오류 잡기 |
| **출력** | ✅(성공) ❌(실패) 🚀(시작) 이모지 사용 |
| **파일 경로** | `os.path` 대신 `pathlib.Path` 사용 |

**예시:**
```python
#!/usr/bin/env python3

def create_file(name: str) -> bool:
    """파일을 생성합니다.
    
    Args:
        name: 만들 파일 이름
        
    Returns:
        성공하면 True, 실패하면 False
    """
    try:
        Path(name).touch()
        print(f"✅ {name} 생성 완료")
        return True
    except Exception as e:
        print(f"❌ 오류: {e}")
        return False
```

---

## 📄 파일 작성 규칙

### 1. 스킬 파일 (SKILL.md)

```markdown
---
name: 스킬-이름
description: 이 스킬이 뭔지, 언제 쓰는지 설명
---

# 스킬 제목

## 개요
한 두 문장으로 설명

## 사용법
구체적인 예시와 절차
```

### 2. 에이전트 파일

```markdown
---
name: 에이전트-이름
description: 이 에이전트가 뭔지, 언제 부르는지 ("use proactively" 넣으면 자동 호출)
tools: Read, Write  # 사용할 도구 (선택)
model: sonnet       # AI 모델 (선택)
---

당신은 ~를 하는 전문가입니다.

## 역할
- 할 일 1
- 할 일 2

## 출력 형식
결과물을 어떻게 보여줄지
```

### 3. 슬래시 명령어

```markdown
---
description: /help에 표시될 설명
---

명령어 실행 시 할 일을 적어요
```

---

## 🏷️ 이름 짓는 규칙

| 대상 | 규칙 | 예시 |
|------|------|------|
| **스킬 이름** | `하이픈-소문자` | `skill-creator`, `youtube-collector` |
| **에이전트 이름** | `하이픈-소문자` | `code-reviewer`, `bug-finder` |
| **파이썬 파일** | `snake_case.py` | `init_skill.py`, `package_skill.py` |
| **폰더 이름** | `하이픈-소문자` | `my-skill`, `new-command` |

---

## 📁 폰더 구조

```
.claude/
├── skills/                      ← 스킬 모음
│   └── 스킬이름/
│       ├── SKILL.md            # 필수! 메인 설명서
│       ├── scripts/            # 실행 코드 (선택)
│       ├── references/         # 추가 문서 (선택)
│       └── assets/             # 템플릿, 이미지 (선택)
│
├── agents/                      ← 에이전트 모음
│   └── 에이전트이름.md         # 에이전트 정의
│
└── commands/                    ← 슬래시 명령어
    └── 명령어이름.md
```

---

## ✨ 작성 팁

### 문서 스타일

- **명령조 사용**: "~하라", "~필요"
- **짧고 간결하게**: 문장마다 "클로드가 정말 이게 필요할까?" 생각하기
- **예시 중심**: 설명보다 예시 코드를 많이 넣기
- **500줄 제한**: SKILL.md는 500줄 이하로 (길면 references/로 분리)
- **TODO 표시**: `[TODO: 사용자가 채울 내용]` 형식으로 남기기

### 중요 원칙

1. **컨텍스트는 공유 자원** - 스킬은 서로 메모리를 공유하니 짧게 작성
2. **스킬에 README 금지** - AI 실행에 필요한 것만 넣기
3. **긴 내용은 분리** - 10k단어 이상은 references/로
4. **스크립트는 확실하게** - 자동화할 땐 스크립트 사용
5. **테스트 필수** - 파이썬 스크립트는 실행 테스트 후 커밋

---

## ❌ 하지 말 것

- 스킬 폰더에 `README.md`, `CHANGELOG.md` 만들지 말기
- SKILL.md와 references/에 내용 중복하지 말기
- references/를 깊게 중첩하지 말기 (1단계만)
- AI 실행에 필요 없는 문서 넣지 말기

---

## 🐛 에러 처리 방법

```python
# ✅ 좋은 예
print("❌ 파일을 찾을 수 없습니다: config.json")
sys.exit(1)

# ❌ 나쁜 예
print("Error")  # 무슨 에러인지 모름
```

**규칙:**
- ❌ 접두사로 오류 표시
- exit(0) = 성공, exit(1+) = 실패
- 파일 작업은 try/except로 감싸기
- 실패필도 대안 동작 제공하기
