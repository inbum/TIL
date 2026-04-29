# LangChain Vector Store & Retriever with Chroma

## 📌 Context

RAG(Retrieval-Augmented Generation) 파이프라인을 구성할 때 핵심은 문서를 벡터로 변환하여 저장하고, 질의(query)와 유사한 문서를 효율적으로 검색하는 것입니다. LangChain은 이 과정을 추상화된 인터페이스로 제공하며, 벡터 스토어(Vector Store)와 검색기(Retriever)는 그 핵심 컴포넌트입니다.

- **Vector Store**: 텍스트 청크를 임베딩 벡터로 변환하여 저장하고, 유사도 검색을 수행하는 저장소
- **Retriever**: Vector Store를 래핑하여 LangChain의 표준 검색 인터페이스를 제공하는 컴포넌트
- **Chroma**: SQLite 기반의 경량 오픈소스 벡터 DB로, 로컬 개발 및 소규모 프로덕션에 적합
- **OpenAI Embedding**: `text-embedding-3-small` / `text-embedding-3-large` 모델을 통해 고품질 벡터 생성

## ⚙️ Core

### 설치

```bash
pip install langchain langchain-openai langchain-chroma chromadb
```

### 기본 구성 코드

```python
import os
from langchain_openai import OpenAIEmbeddings
from langchain_chroma import Chroma
from langchain_core.documents import Document

# 1. 임베딩 모델 초기화
embeddings = OpenAIEmbeddings(
    model="text-embedding-3-small",
    openai_api_key=os.environ["OPENAI_API_KEY"]
)

# 2. 문서 준비
docs = [
    Document(page_content="LangChain은 LLM 애플리케이션 개발 프레임워크입니다.", metadata={"source": "intro"}),
    Document(page_content="Chroma는 오픈소스 벡터 데이터베이스입니다.", metadata={"source": "db"}),
    Document(page_content="RAG는 검색 증강 생성 기법입니다.", metadata={"source": "rag"}),
]

# 3. Vector Store 생성 및 문서 저장
vectorstore = Chroma.from_documents(
    documents=docs,
    embedding=embeddings,
    persist_directory="./chroma_db",
    collection_name="til_collection"
)

# 4. Retriever 생성
retriever = vectorstore.as_retriever(
    search_type="similarity",
    search_kwargs={"k": 3}
)

# 5. 검색 수행
query = "벡터 데이터베이스란?"
results = retriever.invoke(query)

for doc in results:
    print(f"[{doc.metadata['source']}] {doc.page_content}")
```

### MMR(Maximal Marginal Relevance) 검색

```python
retriever_mmr = vectorstore.as_retriever(
    search_type="mmr",
    search_kwargs={
        "k": 3,
        "fetch_k": 10,
        "lambda_mult": 0.5
    }
)
```

### 기존 Chroma DB 로드

```python
vectorstore = Chroma(
    persist_directory="./chroma_db",
    embedding_function=embeddings,
    collection_name="til_collection"
)
```

## 💡 Insight

- **`as_retriever()`는 LCEL과 완전 호환**: `retriever | prompt | llm` 형태의 체인 구성이 가능하여 RAG 파이프라인 구축이 직관적입니다.
- **Chroma의 영속화**: `persist_directory`를 지정하면 SQLite 파일로 저장되어 재실행 시에도 데이터가 유지됩니다. 미지정 시 인메모리로 동작합니다.
- **임베딩 비용 관리**: `text-embedding-3-small`은 `ada-002` 대비 약 5배 저렴하며 성능은 더 우수하므로, 특별한 이유가 없으면 `small` 모델을 우선 선택하세요.
- **`search_type` 선택 기준**: 단순 유사도 검색은 `similarity`, 결과 다양성이 필요하면 `mmr`, 임계값 이상만 필요하면 `similarity_score_threshold`를 사용하세요.
- **프로덕션 전환 고려**: Chroma는 로컬/소규모에 적합하며, 대규모 서비스에서는 Pinecone, Weaviate, pgvector 등으로의 전환을 검토해야 합니다. LangChain의 추상화 덕분에 인터페이스 변경 없이 교체 가능합니다.