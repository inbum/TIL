# LangGraph 활용사례

## 📌 Context
LangGraph는 LangChain 생태계에서 제공하는 상태 기반 멀티 에이전트 오케스트레이션 프레임워크입니다. 단순한 체인(Chain) 방식의 LLM 파이프라인을 넘어, 복잡한 워크플로우를 DAG(Directed Acyclic Graph) 또는 사이클이 포함된 그래프 형태로 설계할 수 있습니다. 특히 에이전트가 도구 호출 → 결과 분석 → 재시도와 같은 반복적 흐름이 필요한 경우, LangGraph는 상태(State)를 유지하면서 노드 간 전환을 제어할 수 있어 실용적인 AI 에이전트 구축에 적합합니다.

## ⚙️ Core

### 기본 구조 — ReAct 에이전트 구현

```python
from typing import TypedDict, Annotated
from langgraph.graph import StateGraph, END
from langgraph.prebuilt import ToolNode
from langchain_anthropic import ChatAnthropic
from langchain_core.tools import tool
from langchain_core.messages import HumanMessage, BaseMessage
import operator

# 상태 정의
class AgentState(TypedDict):
    messages: Annotated[list[BaseMessage], operator.add]

# 도구 정의
@tool
def search_web(query: str) -> str:
    """Search the web for information."""
    return f"Search results for: {query}"

@tool
def calculate(expression: str) -> str:
    """Evaluate a mathematical expression."""
    try:
        return str(eval(expression))
    except Exception as e:
        return f"Error: {e}"

tools = [search_web, calculate]

# LLM 설정
llm = ChatAnthropic(model="claude-sonnet-4-6").bind_tools(tools)

# 노드 정의
def agent_node(state: AgentState):
    response = llm.invoke(state["messages"])
    return {"messages": [response]}

def should_continue(state: AgentState):
    last_message = state["messages"][-1]
    if last_message.tool_calls:
        return "tools"
    return END

# 그래프 구성
tool_node = ToolNode(tools)

graph = StateGraph(AgentState)
graph.add_node("agent", agent_node)
graph.add_node("tools", tool_node)

graph.set_entry_point("agent")
graph.add_conditional_edges("agent", should_continue)
graph.add_edge("tools", "agent")

app = graph.compile()

# 실행
result = app.invoke({
    "messages": [HumanMessage(content="What is 42 * 7 and search for LangGraph")]
})
print(result["messages"][-1].content)
```

### 멀티 에이전트 패턴 — Supervisor 구조

```python
from langgraph.graph import StateGraph, END
from langchain_anthropic import ChatAnthropic
from langchain_core.messages import HumanMessage, SystemMessage
from typing import TypedDict, Literal

class SupervisorState(TypedDict):
    messages: list
    next_agent: str
    final_answer: str

llm = ChatAnthropic(model="claude-sonnet-4-6")

def supervisor_node(state: SupervisorState):
    """작업을 분석하여 적합한 서브 에이전트로 라우팅"""
    system = SystemMessage(content=
        "You are a supervisor. Route tasks to: 'researcher' for information gathering, "
        "'coder' for code generation, or 'finish' if complete. "
        "Respond with only one word: researcher, coder, or finish."
    )
    response = llm.invoke([system] + state["messages"])
    return {"next_agent": response.content.strip().lower()}

def researcher_node(state: SupervisorState):
    response = llm.invoke([
        SystemMessage(content="You are a research specialist."),
        *state["messages"]
    ])
    return {"messages": state["messages"] + [response]}

def coder_node(state: SupervisorState):
    response = llm.invoke([
        SystemMessage(content="You are a coding specialist. Provide clean, working code."),
        *state["messages"]
    ])
    return {"messages": state["messages"] + [response], "final_answer": response.content}

def route(state: SupervisorState) -> Literal["researcher", "coder", "__end__"]:
    next_agent = state.get("next_agent", "finish")
    if next_agent == "researcher":
        return "researcher"
    elif next_agent == "coder":
        return "coder"
    return END

workflow = StateGraph(SupervisorState)
workflow.add_node("supervisor", supervisor_node)
workflow.add_node("researcher", researcher_node)
workflow.add_node("coder", coder_node)

workflow.set_entry_point("supervisor")
workflow.add_conditional_edges("supervisor", route)
workflow.add_edge("researcher", "supervisor")
workflow.add_edge("coder", END)

app = workflow.compile()
```

## 💡 Insight
- **상태 관리의 강점**: LangGraph는 `TypedDict` 기반의 명시적 상태 관리로 복잡한 멀티턴 대화나 에이전트 협업 시 데이터 흐름 추적이 용이합니다.
- **사이클 지원**: LangChain의 단방향 체인과 달리 루프(agent → tools → agent)가 가능하여 ReAct 패턴 구현에 최적화되어 있습니다.
- **트레이드오프**: 단순한 Q&A나 단일 LLM 호출에는 오버헤드가 크므로, 복잡도가 낮은 태스크에는 일반 LangChain Chain 사용을 권장합니다.
- **실무 활용**: RAG 에이전트, 코드 리뷰 봇, 멀티 에이전트 리서치 시스템, 고객 지원 자동화 등에서 높은 활용도를 보입니다.
- **LangGraph Cloud**: Managed 환경으로 배포 시 checkpointing, persistence, streaming을 기본 지원하여 프로덕션 레디 에이전트 구축에 유리합니다.