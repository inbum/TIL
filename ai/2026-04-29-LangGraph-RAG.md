# LangGraph를 이용한 핵심 RAG 구현

## 📌 Context
RAG(Retrieval-Augmented Generation)는 LLM이 외부 지식 베이스에서 관련 문서를 검색한 뒤 이를 바탕으로 답변을 생성하는 패턴이다. LangGraph는 LangChain 생태계에서 복잡한 AI 워크플로우를 상태 기반 그래프로 표현할 수 있는 프레임워크로, RAG 파이프라인을 노드와 엣지로 명확하게 구조화할 수 있다.

기존 단순 체인(Chain) 방식과 달리 LangGraph는 상태(State)를 명시적으로 관리하고, 각 처리 단계를 독립적인 노드로 분리함으로써 조건부 분기, 루프, 병렬 처리 등 복잡한 제어 흐름을 구현할 수 있다.

## ⚙️ Core
```python
from typing import TypedDict, List
from langchain_core.documents import Document
from langchain_core.output_parsers import StrOutputParser
from langchain_core.prompts import ChatPromptTemplate
from langchain_openai import ChatOpenAI, OpenAIEmbeddings
from langchain_community.vectorstores import FAISS
from langgraph.graph import StateGraph, END

# 1. 상태(State) 정의 - 그래프 전체에서 공유되는 데이터 구조
class RAGState(TypedDict):
    question: str
    documents: List[Document]
    answer: str

# 벡터스토어 초기화
embeddings = OpenAIEmbeddings()
texts = [
    "LangGraph는 상태 기반 그래프 프레임워크입니다.",
    "RAG는 검색 증강 생성 기법입니다.",
]
vectorstore = FAISS.from_texts(texts, embeddings)

# 2. 검색(Retriever) 노드
def retrieve(state: RAGState) -> dict:
    retriever = vectorstore.as_retriever(search_kwargs={"k": 3})
    docs = retriever.invoke(state["question"])
    return {"documents": docs}

# 3. 생성(Generator) 노드
def generate(state: RAGState) -> dict:
    llm = ChatOpenAI(model="gpt-4o-mini", temperature=0)
    prompt = ChatPromptTemplate.from_template(
        "다음 컨텍스트를 바탕으로 질문에 정확하게 답하세요.\n\n"
        "컨텍스트:\n{context}\n\n"
        "질문: {question}\n\n"
        "답변:"
    )
    chain = prompt | llm | StrOutputParser()
    context = "\n\n".join(doc.page_content for doc in state["documents"])
    answer = chain.invoke({"context": context, "question": state["question"]})
    return {"answer": answer}

# 4. 그래프 구조 구성 및 연결
workflow = StateGraph(RAGState)
workflow.add_node("retrieve", retrieve)
workflow.add_node("generate", generate)
workflow.set_entry_point("retrieve")
workflow.add_edge("retrieve", "generate")
workflow.add_edge("generate", END)
graph = workflow.compile()

# 5. 실행
result = graph.invoke({"question": "LangGraph란 무엇인가요?"})
print(result["answer"])
```

## 💡 Insight
- **상태 중심 설계의 강점**: `TypedDict`로 상태를 명시하면 각 노드의 입출력 계약이 명확해지고, mypy 등 타입 검사 도구와 함께 사용 시 안전성이 높아진다.
- **노드 분리의 이점**: 검색과 생성을 별도 노드로 분리하면 각 노드를 독립적으로 테스트하거나 교체할 수 있다. 예를 들어 retriever만 BM25로 교체해도 generate 노드는 수정이 불필요하다.
- **확장 가능성**: 현재의 `retrieve → generate` 구조에 `grade_documents`(검색 품질 평가), `rewrite_query`(질문 재작성) 노드를 추가하면 Self-RAG, Corrective RAG 등 고급 패턴으로 확장 가능하다.
- **주의사항**: 각 노드 함수는 반드시 `dict`를 반환해야 하며, 반환 키는 `State` 필드와 일치해야 한다. 전체 상태를 반환할 필요 없이 변경된 필드만 반환하면 LangGraph가 자동으로 병합한다.