# LangChain, LangGraph, LangFlow, LangSmith 비교

## 📌 Context

LangChain 생태계는 LLM 애플리케이션 개발을 위한 다양한 도구들로 구성되어 있다. 이름이 비슷해 혼동하기 쉽지만, 각 도구는 개발 워크플로우의 서로 다른 단계와 목적을 담당한다. 어떤 도구를 언제 사용해야 하는지 명확히 구분하는 것이 효율적인 LLM 애플리케이션 개발의 출발점이다.

## ⚙️ Core

| 도구 | 역할 | 주요 대상 |
|------|------|----------|
| LangChain | LLM 앱 개발 프레임워크 (기반) | 개발자 |
| LangGraph | 그래프 기반 멀티 에이전트 워크플로우 | 개발자 |
| LangFlow | 시각적 노코드/로코드 파이프라인 UI | 개발자 + 비개발자 |
| LangSmith | 모니터링 · 디버깅 · 평가 플랫폼 | 개발자 + ML Ops |

### LangChain

```python
from langchain_openai import ChatOpenAI
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.output_parsers import StrOutputParser

llm = ChatOpenAI(model="gpt-4o")
prompt = ChatPromptTemplate.from_template("Tell me a joke about {topic}")
chain = prompt | llm | StrOutputParser()

result = chain.invoke({"topic": "AI"})
print(result)
```

- LCEL(LangChain Expression Language)로 컴포넌트를 `|` 연산자로 연결
- Chain, Agent, Tool, Memory, Retriever 등의 핵심 추상화 제공
- RAG, 에이전트, 요약 등 대부분의 LLM 패턴 지원

### LangGraph

```python
from langgraph.graph import StateGraph, END
from typing import TypedDict

class AgentState(TypedDict):
    messages: list
    next: str

def agent_node(state: AgentState):
    # 에이전트 로직 처리
    return {"messages": state["messages"], "next": "tool"}

def tool_node(state: AgentState):
    # 도구 실행
    return {"messages": state["messages"], "next": END}

builder = StateGraph(AgentState)
builder.add_node("agent", agent_node)
builder.add_node("tool", tool_node)
builder.set_entry_point("agent")
builder.add_edge("agent", "tool")
builder.add_edge("tool", END)

graph = builder.compile()
result = graph.invoke({"messages": [], "next": ""})
```

- LangChain 위에 구축된 **사이클릭 그래프** 기반 워크플로우
- 상태(State)를 노드 간에 공유하며 조건부 분기와 루프 처리
- Human-in-the-loop, 체크포인트, 스트리밍 등 프로덕션급 기능 지원

### LangFlow

```bash
# 설치 및 실행
pip install langflow
python -m langflow run
# 브라우저에서 http://localhost:7860 접속
```

- 드래그 앤 드롭 UI로 LangChain 컴포넌트를 시각적으로 연결
- 완성된 파이프라인을 REST API로 즉시 노출 가능
- JSON으로 파이프라인 내보내기/불러오기 지원

### LangSmith

```python
import os
from langchain_openai import ChatOpenAI
from langchain_core.prompts import ChatPromptTemplate

# LangSmith 활성화
os.environ["LANGCHAIN_TRACING_V2"] = "true"
os.environ["LANGCHAIN_API_KEY"] = "<your-langsmith-api-key>"
os.environ["LANGCHAIN_PROJECT"] = "my-til-project"

llm = ChatOpenAI(model="gpt-4o")
prompt = ChatPromptTemplate.from_template("{input}")
chain = prompt | llm

# 이후 모든 실행이 LangSmith에 자동 추적됨
chain.invoke({"input": "Hello, LangSmith!"})
```

- 환경 변수 설정만으로 기존 LangChain 코드에 추적 기능 추가
- 각 실행의 입출력, 레이턴시, 토큰 사용량, 에러를 시각화
- 데이터셋 기반 자동 평가(Evaluation) 및 A/B 테스트 지원

## 💡 Insight

- **도구 선택 기준**: 단순 파이프라인 → LangChain, 복잡한 에이전트 루프 → LangGraph, 비개발자 협업·프로토타이핑 → LangFlow, 품질 모니터링 → LangSmith
- **조합 사용 권장**: 실무에서는 LangChain + LangGraph로 로직을 구현하고, LangSmith로 모니터링하는 조합이 일반적
- **LangFlow의 한계**: 시각적 도구인 만큼 복잡한 커스텀 로직 구현에는 한계가 있으며, 코드 기반 전환 시 재작업이 발생할 수 있음
- **LangGraph vs LangChain Agents**: 기존 AgentExecutor 방식보다 LangGraph가 더 세밀한 제어와 디버깅을 제공하므로, 신규 에이전트 프로젝트에는 LangGraph를 우선 검토할 것
- LangSmith는 무료 티어에서도 월 5,000건 트레이스를 제공하므로 개발 단계부터 활성화해두면 디버깅 비용을 크게 절약할 수 있음