# LangGraph 활용사례

## 📌 Context
LangGraph는 LangChain 팀이 개발한 라이브러리로, LLM 기반 애플리케이션에서 **상태(state)를 유지하는 멀티 액터 워크플로우**를 구축할 수 있게 해준다. 기존 LangChain의 선형적 체인 구조와 달리, LangGraph는 **순환 그래프(Cyclic Graph)** 구조를 지원하여 복잡한 에이전트 흐름, 조건 분기, 반복 처리가 가능하다. 멀티 에이전트 오케스트레이션, 인간 개입(Human-in-the-Loop), 장기 기억 등이 필요한 실무 시나리오에서 특히 강력하다.

## ⚙️ Core

### 1. 기본 StateGraph 구조

```python
from langgraph.graph import StateGraph, END
from typing import TypedDict, Annotated
import operator

class AgentState(TypedDict):
    messages: Annotated[list, operator.add]
    next: str

def node_a(state: AgentState) -> AgentState:
    return {"messages": ["Node A 처리 완료"], "next": "B"}

def node_b(state: AgentState) -> AgentState:
    return {"messages": ["Node B 처리 완료"], "next": END}

def router(state: AgentState) -> str:
    return state["next"]

graph = StateGraph(AgentState)
graph.add_node("A", node_a)
graph.add_node("B", node_b)
graph.set_entry_point("A")
graph.add_conditional_edges("A", router, {"B": "B", END: END})
graph.add_edge("B", END)

app = graph.compile()
result = app.invoke({"messages": [], "next": ""})
print(result)
```

### 2. 멀티 에이전트 오케스트레이션 (Supervisor 패턴)

```python
from langchain_anthropic import ChatAnthropic
from langgraph.graph import StateGraph, END
from typing import TypedDict, Literal

llm = ChatAnthropic(model="claude-sonnet-4-6")

class SupervisorState(TypedDict):
    task: str
    result: str
    next_agent: str

def supervisor(state: SupervisorState) -> SupervisorState:
    """어떤 에이전트에게 작업을 위임할지 결정"""
    response = llm.invoke(
        f"다음 작업을 분석하고 'researcher' 또는 'coder' 중 적합한 에이전트를 답하라.\n작업: {state['task']}"
    )
    return {"next_agent": response.content.strip()}

def researcher(state: SupervisorState) -> SupervisorState:
    response = llm.invoke(f"다음을 조사하라: {state['task']}")
    return {"result": response.content}

def coder(state: SupervisorState) -> SupervisorState:
    response = llm.invoke(f"다음을 코드로 구현하라: {state['task']}")
    return {"result": response.content}

def route(state: SupervisorState) -> Literal["researcher", "coder"]:
    return state["next_agent"]

graph = StateGraph(SupervisorState)
graph.add_node("supervisor", supervisor)
graph.add_node("researcher", researcher)
graph.add_node("coder", coder)
graph.set_entry_point("supervisor")
graph.add_conditional_edges("supervisor", route)
graph.add_edge("researcher", END)
graph.add_edge("coder", END)

app = graph.compile()
result = app.invoke({"task": "파이썬으로 소수 판별 알고리즘 구현", "result": "", "next_agent": ""})
print(result["result"])
```

### 3. Human-in-the-Loop (중단점 활용)

```python
from langgraph.checkpoint.memory import MemorySaver

memory = MemorySaver()

graph = StateGraph(AgentState)
# ... 노드 추가 ...
app = graph.compile(
    checkpointer=memory,
    interrupt_before=["sensitive_node"]  # 이 노드 전에 중단
)

config = {"configurable": {"thread_id": "session-001"}}

# 1단계: 실행 → sensitive_node 직전에 중단
state = app.invoke({"messages": ["작업 시작"]}, config=config)

# 2단계: 인간 검토 후 재개
app.invoke(None, config=config)  # None 전달 시 중단점에서 재개
```

### 4. 반복 RAG 워크플로우 (Iterative Retrieval)

```python
from langgraph.graph import StateGraph, END
from typing import TypedDict

class RAGState(TypedDict):
    question: str
    documents: list
    answer: str
    grade: str
    iterations: int

def retrieve(state: RAGState) -> RAGState:
    # 벡터 DB에서 문서 검색
    docs = retriever.invoke(state["question"])
    return {"documents": docs, "iterations": state.get("iterations", 0) + 1}

def generate(state: RAGState) -> RAGState:
    answer = llm.invoke(f"문서: {state['documents']}\n질문: {state['question']}")
    return {"answer": answer.content}

def grade_answer(state: RAGState) -> RAGState:
    grade = llm.invoke(f"답변이 질문에 충분한가? yes/no만 답하라.\n답변: {state['answer']}")
    return {"grade": grade.content.strip().lower()}

def should_retry(state: RAGState) -> str:
    if state["grade"] == "no" and state["iterations"] < 3:
        return "retrieve"  # 재검색
    return END

graph = StateGraph(RAGState)
graph.add_node("retrieve", retrieve)
graph.add_node("generate", generate)
graph.add_node("grade", grade_answer)
graph.set_entry_point("retrieve")
graph.add_edge("retrieve", "generate")
graph.add_edge("generate", "grade")
graph.add_conditional_edges("grade", should_retry)

app = graph.compile()
```

## 💡 Insight
- **StateGraph vs MessageGraph**: 복잡한 상태 관리가 필요하면 `StateGraph`, 단순 채팅 흐름이면 `MessageGraph`를 사용한다. 실무에서는 대부분 `StateGraph`가 적합하다.
- **순환 그래프의 위험**: `should_retry` 같은 종료 조건 없이 순환 엣지를 추가하면 무한 루프에 빠진다. 반드시 `iterations` 카운터나 조건 분기로 탈출 경로를 명시해야 한다.
- **Checkpointer 활용**: `MemorySaver`(인메모리) 외에 `PostgresSaver`, `SqliteSaver`를 사용하면 서버 재시작 후에도 대화 상태를 복원할 수 있어 프로덕션 환경에서 필수적이다.
- **LangGraph Studio**: 로컬에서 그래프를 시각화하고 디버깅할 수 있는 GUI 도구로, 복잡한 워크플로우 설계 시 병목 노드 파악에 효과적이다.
- **n8n과의 비교**: LangGraph는 코드 레벨의 세밀한 제어가 필요할 때, n8n은 노코드 자동화에 적합하다. LLM 로직이 복잡할수록 LangGraph가 유리하다.