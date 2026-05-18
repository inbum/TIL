# 하네스 엔지니어링의 핵심 아키텍처 규칙

## 📌 Context
AI 에이전트 시스템을 운영하는 팀이 하네스(Harness) 엔지니어링 아키텍처를 설계할 때, 단순한 스크립트 모음이 아닌 확장 가능하고 유지보수 가능한 시스템을 구축하기 위한 핵심 원칙들이 있다. 이 규칙들은 에이전트 오케스트레이션 시스템이 커질수록 발생하는 문서 분산, 코드 품질 저하, 기술 부채 누적 문제를 예방한다.

## ⚙️ Core

### 1. 분산형 문서화 (AGENTS.md)
각 디렉토리 또는 모듈에 `AGENTS.md` 파일을 배치하여 해당 에이전트/모듈의 역할, 입출력, 의존성을 로컬에서 관리한다.

```
project/
├── AGENTS.md              # 프로젝트 전체 에이전트 개요
├── orchestrator/
│   ├── AGENTS.md          # 오케스트레이터 역할 정의
│   └── main.py
├── tools/
│   ├── AGENTS.md          # 각 툴의 인터페이스 명세
│   └── search_tool.py
```

```markdown
# orchestrator/AGENTS.md

## Role
사용자 요청을 분석하고 적절한 서브 에이전트에게 작업을 위임

## Inputs
- user_query: str

## Outputs
- task_result: TaskResult

## Dependencies
- tools/search_tool
- tools/code_executor
```

### 2. Git 중심의 레코드 관리

```bash
# 에이전트 실행 결과를 Git 추적 가능한 형태로 저장
git add agent_runs/$(date +%Y-%m-%d)/
git commit -m "agent-run: task_id=$(TASK_ID) status=$(STATUS)"

# 에이전트 출력을 JSON Lines 형식으로 누적
echo '{"timestamp": "2026-05-18T10:00:00Z", "agent": "orchestrator", "status": "success"}' \
  >> agent_runs/2026-05-18/log.jsonl
```

### 3. 정적 분석(Linter) 가드레일

```yaml
# .pre-commit-config.yaml
repos:
  - repo: https://github.com/astral-sh/ruff-pre-commit
    rev: v0.4.0
    hooks:
      - id: ruff
        args: [--fix]
      - id: ruff-format
  - repo: https://github.com/pre-commit/mirrors-mypy
    rev: v1.10.0
    hooks:
      - id: mypy
        args: [--strict]
```

```toml
# pyproject.toml
[tool.ruff]
line-length = 88
select = ["E", "F", "I", "N", "W", "UP"]

[tool.mypy]
strict = true
disallow_untyped_defs = true
```

### 4. 짧은 PR(Pull Request) 주기

```bash
# 에이전트 툴 하나 추가 → 즉시 PR
git checkout -b feat/add-search-tool
git add tools/search_tool.py tools/AGENTS.md
git commit -m "feat: add search tool with web crawling support"
gh pr create --title "feat: add search tool" --body "단일 툴 추가, 사이드 이펙트 없음"

# 다음 변경은 별도 브랜치에서 시작
git checkout -b feat/integrate-search-tool
```

### 5. 자동화된 품질 관리 (Garbage Collection)

```python
# gc_agent.py - 미사용 툴 자동 감지
import ast
from pathlib import Path

def find_unused_tools(tools_dir: str, agents_dir: str) -> list[str]:
    """어떤 에이전트에서도 참조되지 않는 툴 반환"""
    registered = {p.stem for p in Path(tools_dir).glob("*.py")}
    referenced: set[str] = set()

    for agent_file in Path(agents_dir).rglob("*.py"):
        tree = ast.parse(agent_file.read_text())
        for node in ast.walk(tree):
            if isinstance(node, ast.Import):
                for alias in node.names:
                    referenced.add(alias.name.split(".")[0])
            elif isinstance(node, ast.ImportFrom) and node.module:
                referenced.add(node.module.split(".")[0])

    return list(registered - referenced)

if __name__ == "__main__":
    unused = find_unused_tools("tools/", "agents/")
    print(f"미사용 툴 {len(unused)}개: {unused}")
```

## 💡 Insight
- **AGENTS.md 분산화**는 중앙 문서가 비대해지는 것을 막고, 모듈 담당자가 독립적으로 문서를 관리할 수 있게 한다.
- **Git 중심 레코드**는 에이전트 실행 이력을 코드와 동일하게 관리하므로 `git bisect`로 버그 발생 시점 추적이 가능하다.
- **Linter 가드레일**은 LLM이 생성한 코드라도 타입 안전성과 스타일 일관성을 강제할 수 있다.
- **짧은 PR 주기**는 에이전트 변경의 사이드 이펙트를 조기에 리뷰할 수 있어 통합 위험을 줄인다.
- **GC 자동화**는 에이전트 생태계가 성장하며 누적되는 dead code와 orphan tool 문제를 선제적으로 방지한다.
- 5가지 규칙은 개별 적용도 가능하지만, 함께 적용할 때 유지보수성이 기하급수적으로 향상된다.