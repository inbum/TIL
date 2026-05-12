# Andrej Karpathy 스타일 CLAUDE.md 스킬 설정

## 📌 Context

Andrej Karpathy는 AI 도구를 활용한 개발 워크플로우를 적극적으로 공유하는 것으로 알려져 있다. 그의 접근 방식에서 영감을 받아, Claude Code의 `CLAUDE.md` 파일에 커스텀 슬래시 커맨드(스킬)를 정의함으로써 반복적인 작업을 자동화하고 AI와의 협업 효율을 극대화하는 방법을 학습했다.

CLAUDE.md는 Claude Code가 프로젝트 시작 시 자동으로 읽는 설정 파일로, 프로젝트별 컨텍스트, 지시사항, 그리고 커스텀 스킬(슬래시 커맨드)을 정의할 수 있다.

## ⚙️ Core

### CLAUDE.md 기본 구조

```markdown
# Project Context
프로젝트에 대한 설명 및 주요 기술 스택

# Instructions
- 코드 스타일 지침
- 응답 형식 지정
- 금지 행동 목록

# Custom Skills
<!-- 슬래시 커맨드 정의 -->
```

### 스킬(슬래시 커맨드) 정의 예시

스킬은 `.claude/commands/` 디렉토리에 마크다운 파일로 정의한다.

```bash
# 프로젝트 루트에서 실행
mkdir -p .claude/commands
```

```markdown
<!-- .claude/commands/til.md -->
당신은 TIL 문서 생성 전문가입니다.
사용자의 입력을 분석하여 GitHub TIL 마크다운 문서를 JSON 형식으로 생성하세요.

## 출력 형식
```json
{
  "folder": "<카테고리>",
  "filename": "<YYYY-MM-DD-제목.md>",
  "content": "<마크다운 전체 내용>"
}
```
```

### 글로벌 vs 프로젝트 레벨 스킬

```bash
# 글로벌 스킬 (모든 프로젝트에서 사용)
~/.claude/commands/

# 프로젝트 스킬 (해당 프로젝트에서만 사용)
./.claude/commands/
```

### CLAUDE.md에서 스킬 참조

```markdown
# Available Skills
- `/til` - TIL 문서 자동 생성
- `/review` - 코드 리뷰 수행
- `/refactor` - 리팩토링 제안

# Usage
`/til <학습한 내용>` 형식으로 입력하면 구조화된 TIL 문서를 생성합니다.
```

## 💡 Insight

- **재사용성**: 한 번 정의한 스킬은 모든 세션에서 동일하게 동작하므로 프롬프트를 매번 반복 입력할 필요가 없다.
- **컨텍스트 일관성**: CLAUDE.md를 git으로 관리하면 팀원 모두가 동일한 AI 워크플로우를 공유할 수 있다.
- **Karpathy 방식의 핵심**: AI를 단순 도구가 아닌 "프로그래밍 가능한 에이전트"로 보고, 반복 작업을 스킬로 추상화하는 것이 생산성의 핵심이다.
- **주의사항**: 스킬 파일이 너무 길거나 복잡하면 컨텍스트 윈도우를 불필요하게 소모할 수 있으므로, 지시사항은 간결하게 유지하는 것이 중요하다.
- **활용 가능성**: TIL 자동화 외에도 코드 리뷰, 커밋 메시지 생성, API 문서화 등 반복적인 개발 작업 전반에 적용 가능하다.