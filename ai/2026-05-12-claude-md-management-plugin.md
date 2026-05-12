# claude-md-management Plugin

## 📌 Context

Claude Code의 공식 플러그인 레지스트리인 `claude-plugins-official`에서 CLAUDE.md 파일을 관리하고 개선하기 위한 전용 플러그인 `claude-md-management`를 제공한다는 것을 알게 되었다. CLAUDE.md는 프로젝트 루트에 위치하는 Claude Code의 핵심 설정 파일로, AI 어시스턴트의 동작 지침, 코딩 컨벤션, 프로젝트 컨텍스트 등을 정의한다. 이 파일을 잘 관리하고 지속적으로 개선하는 것은 Claude Code의 활용도를 높이는 데 중요하다.

## ⚙️ Core

### 플러그인 설치
```bash
# claude-plugins-official 레지스트리에서 플러그인 추가
# Claude Code 설정을 통해 claude-md-management 플러그인 설치
```

### 제공 기능

#### 1. `claude-md-improver` 스킬
- 현재 CLAUDE.md 파일을 분석하고 개선점을 제안
- `/claude-md-improver` 형태로 호출

```bash
# 스킬 호출 예시
/claude-md-improver
```

#### 2. `revise-claude-md` 커맨드
- CLAUDE.md 파일을 직접 수정 및 개정하는 슬래시 커맨드
- 대화 맥락을 반영하여 CLAUDE.md를 업데이트

```bash
# 커맨드 호출 예시
/revise-claude-md
```

## 💡 Insight

- **스킬 vs 커맨드 구분**: `claude-md-improver`는 분석 및 제안 중심, `revise-claude-md`는 실제 파일 수정 중심으로 역할이 나뉘는 구조로 추정된다. 용도에 맞게 선택적으로 사용하는 것이 효율적이다.
- **CLAUDE.md 품질 관리**: 프로젝트가 성장하면서 CLAUDE.md가 낡거나 불완전해지는 경우가 많은데, 전용 플러그인으로 주기적으로 관리하면 Claude Code의 퍼포먼스를 일정 수준으로 유지할 수 있다.
- **공식 레지스트리 활용**: `claude-plugins-official`을 통해 검증된 플러그인을 사용하면 커스텀 훅이나 수동 설정 없이도 반복적인 작업을 자동화할 수 있다는 점에서 실무 효율이 높아진다.
- **한계**: 플러그인의 구체적인 동작 방식(LLM 호출 여부, 파일 diff 방식 등)은 공식 문서 확인이 필요하며, 자동 수정 기능 사용 시 변경 내용을 반드시 리뷰하는 습관이 필요하다.