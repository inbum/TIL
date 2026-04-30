# Andrej Karpathy의 AI 코딩 에이전트 작업지침 파일

## 📌 Context
AI 코딩 에이전트가 폭발적으로 확산되면서, 에이전트에게 작업 컨텍스트와 지침을 효과적으로 전달하는 방법이 중요한 화두로 떠올랐다. Andrej Karpathy는 AI 에이전트와 협업할 때 프로젝트 루트에 작업지침 파일(예: `AGENTS.md`, `CLAUDE.md`, `CODEX.md`)을 배치하는 패턴을 소개하며, 이를 통해 에이전트가 프로젝트의 구조·규칙·금기사항을 사전에 파악하고 더 정확하게 작동할 수 있다고 강조했다.

## ⚙️ Core

### 작업지침 파일의 일반적인 구조

```markdown
# Project Agent Instructions

## Overview
프로젝트의 목적과 기술 스택을 한 줄로 요약한다.

## Directory Structure
주요 폴더와 역할을 설명한다.

## Coding Conventions
- 언어별 스타일 가이드 (PEP8, ESLint 등)
- 네이밍 규칙 (snake_case, camelCase 등)
- 금지 패턴 (예: print 대신 logging 사용)

## Workflow
1. 기능 브랜치 생성 → 2. 구현 → 3. 테스트 → 4. PR

## Test Instructions
테스트 실행 명령어와 커버리지 기준을 명시한다.

## Do NOT
- 민감 정보를 코드에 하드코딩하지 말 것
- main 브랜치에 직접 푸시하지 말 것
```

### Claude Code에서의 실제 예시 (CLAUDE.md)

```markdown
# CLAUDE.md

## Commands
- `npm run dev` : 개발 서버 실행
- `npm run test` : 전체 테스트 수행
- `npm run lint` : ESLint + Prettier 검사

## Architecture
src/
  api/        # REST 엔드포인트
  services/   # 비즈니스 로직
  models/     # DB 스키마 (Prisma)

## Rules
- 모든 API 응답은 { data, error, meta } 구조를 따를 것
- 외부 라이브러리 추가 시 반드시 사전 승인 요청
- 테스트 커버리지 80% 미만 PR은 머지 금지
```

### 지원되는 파일명 (에이전트별)

| 에이전트 | 인식 파일명 |
|---|---|
| Claude Code | `CLAUDE.md` |
| OpenAI Codex | `CODEX.md` |
| Devin | `AGENTS.md` |
| 범용 | `AGENTS.md`, `.cursorrules` |

## 💡 Insight
- **에이전트는 지침 파일을 컨텍스트의 앵커로 활용한다.** 파일이 없으면 에이전트는 프로젝트 전체를 탐색하며 규칙을 추론해야 하지만, 명시적인 지침 파일이 있으면 불필요한 탐색을 줄이고 작업 정확도가 높아진다.
- **Karpathy의 핵심 주장:** 코드보다 _지침의 품질_ 이 에이전트 결과물의 품질을 결정한다. 좋은 지침 파일은 일종의 "시스템 프롬프트" 역할을 한다.
- **주의사항:** 지침 파일이 오래되면 오히려 에이전트를 잘못된 방향으로 유도할 수 있다. 코드 변경과 함께 지침 파일도 버전 관리하는 습관이 중요하다.
- **실무 팁:** `Do NOT` 섹션에 과거 에이전트 실수 사례를 누적 기록하면, 반복적인 실수를 구조적으로 방지할 수 있다.