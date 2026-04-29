# LangChain, LangGraph, LangFlow, LangSmith 비교

## 📌 Context

LLM 기반 애플리케이션을 개발할 때 LangChain 생태계의 여러 도구들이 혼재하여 각 도구의 역할과 사용 시나리오를 명확히 이해하기 어려운 경우가 있다. LangChain, LangGraph, LangFlow, LangSmith는 모두 LangChain 생태계에 속하지만, 각각 다른 레이어와 목적을 담당한다.

---

## ⚙️ Core

### LangChain
- LLM 애플리케이션의 기반 프레임워크
- Chain, Agent, Memory, Tools 등의 추상화 제공

```python
from langchain_openai import ChatOpenAI
from langchain_core.prompts import ChatPromptTemplate

llm = ChatOpenAI(model="gpt-4o")
prompt = ChatPromptTemplate.from_messages([
    ("system", "You are a helpful assistant."),
    ("human", "{input}")
])
chain = prompt | llm
response = chain.invoke({"input": "LangChain이란?"})
print(response.content)
```

---

### LangGraph
- 상태 기반 그래프(DAG/Cyclic)로 복잡한 멀티에이전트 워크플로우 구현
- 반복, 분기, 상태 관리가 필요한 에이전트에 적합
- LangChain 위에서 동작

```python
from langgraph.graph import StateGraph, END
from typing import TypedDict

class State(TypedDict):
    messages: list
    next_step: str

def agent_node(state: State) -> State:
    return {"messages": state["messages"], "next_step": "end"}

def should_continue(state: State) -> str:
    return state["next_step"]

graph = StateGraph(State)
graph.add_node("agent", agent_node)
graph.add_conditional_edges("agent", should_continue, {"end": END})
graph.set_entry_point("agent")
app = graph.compile()
result = app.invoke({"messages": [], "next_step": ""})
```

---

### LangFlow
- LangChain 기반의 시각적(노코드/로우코드) 워크플로우 빌더
- GUI 드래그앤드롭으로 체인과 에이전트 구성
- 프로토타이핑 및 비개발자 협업에 활용

```bash
# 설치 및 실행
pip install langflow
langflow run
# 브라우저에서 http://localhost:7860 접속
```

---

### LangSmith
- LLM 앱의 트레이싱, 디버깅, 평가, 모니터링 플랫폼
- 프로덕션 관측성(Observability) 도구
- 환경변수 설정만으로 LangChain 실행에 자동 연동

```python
import os
from langchain_openai import ChatOpenAI
from langchain_core.prompts import ChatPromptTemplate

os.environ["LANGCHAIN_TRACING_V2"] = "true"
os.environ["LANGCHAIN_API_KEY"] = "ls__your_api_key"
os.environ["LANGCHAIN_PROJECT"] = "my-project"

llm = ChatOpenAI(model="gpt-4o")
prompt = ChatPromptTemplate.from_messages([("human", "{input}")])
chain = prompt | llm
# 이 실행이 자동으로 LangSmith에 트레이싱됨
response = chain.invoke({"input": "LangSmith 테스트"})
```

---

## 💡 Insight

| 도구 | 역할 | 주요 사용자 |
|------|------|-------------|
| LangChain | LLM 앱 개발의 코어 SDK | 개발자 |
| LangGraph | 복잡한 상태 기반 에이전트 구현 | 시니어 개발자 |
| LangFlow | 노코드/로우코드 시각적 빌더 | 비개발자, 프로토타이퍼 |
| LangSmith | 관측성·디버깅·평가 플랫폼 | 개발자, ML 엔지니어 |

- 레이어 관계: `LangSmith(관측) ↔ LangFlow(GUI) → LangChain(코어) ← LangGraph(복잡 에이전트)`
- 실무에서는 **LangChain + LangSmith** 조합이 기본, 복잡한 에이전트 필요 시 **LangGraph** 추가
- LangFlow는 내부적으로 LangChain을 사용하므로, 코드 전환 시 LangChain 지식이 그대로 활용됨
- LangGraph는 단순 체인으로 해결 안 되는 루프·분기 에이전트 구현 시 진입 장벽이 있지만, 유연성이 높음
