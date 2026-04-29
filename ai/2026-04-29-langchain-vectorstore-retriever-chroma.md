# LangChain Vector Store & Retriever with Chroma + OpenAI Embedding

## 📌 Context
LangChain 기반 RAG(Retrieval-Augmented Generation) 파이프라인을 구축할 때, 문서를 벡터로 변환하고 유사도 기반 검색을 수행하기 위해 Vector Store와 Retriever를 구성해야 한다. 가장 널리 쓰이는 오픈소스 벡터 DB인 Chroma와 OpenAI의 text-embedding 모델을 연동하여, 로컬 환경에서도 빠르게 시맨틱 검색 파이프라인을 구성하는 방법을 정리한다.

## ⚙️ Core

### 설치
```bash
pip install langchain langchain-openai langchain-chroma chromadb
```

### 기본 구성
```python
import os
from langchain_openai import OpenAIEmbeddings
from langchain_chroma import Chroma
from langchain.text_splitter import RecursiveCharacterTextSplitter
from langchain_core.documents import Document

# 1. 임베딩 모델 초기화
embeddings = OpenAIEmbeddings(
    model="text-embedding-3-small",  # 또는 text-embedding-3-large
    openai_api_key=os.environ["OPENAI_API_KEY"]
)

# 2. 샘플 문서 준비
documents = [
    Document(page_content="LangChain은 LLM 기반 애플리케이션을 구축하기 위한 프레임워크입니다."),
    Document(page_content="Chroma는 AI 네이티브 오픈소스 임베딩 데이터베이스입니다."),
    Document(page_content="Vector Store는 임베딩 벡터를 저장하고 유사도 검색을 지원합니다."),
]

# 3. 텍스트 분할
splitter = RecursiveCharacterTextSplitter(chunk_size=500, chunk_overlap=50)
split_docs = splitter.split_documents(documents)

# 4. Chroma Vector Store 생성 및 문서 저장
vectorstore = Chroma.from_documents(
    documents=split_docs,
    embedding=embeddings,
    persist_directory="./chroma_db",
    collection_name="til_collection"
)

# 5. Retriever 구성
retriever = vectorstore.as_retriever(
    search_type="similarity",
    search_kwargs={"k": 3}
)

# 6. 검색 실행
query = "임베딩 데이터베이스란 무엇인가?"
results = retriever.invoke(query)
for doc in results:
    print(doc.page_content)
```

### MMR(Maximal Marginal Relevance) 검색 방식
```python
retriever_mmr = vectorstore.as_retriever(
    search_type="mmr",
    search_kwargs={"k": 3, "fetch_k": 10, "lambda_mult": 0.5}
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
- `text-embedding-3-small`은 비용 대비 성능이 우수하며, 대부분의 RAG 파이프라인에서 충분한 품질을 제공한다. 고정밀이 필요한 경우 `text-embedding-3-large` 사용을 권장한다.
- Chroma의 `persist_directory` 옵션으로 벡터를 로컬 디스크에 영속화할 수 있어, 반복 실행 시 임베딩 API 호출 비용을 절약할 수 있다.
- `search_type="mmr"`은 유사 문서의 중복 반환을 방지해 다양한 컨텍스트를 LLM에 제공할 때 유리하다. `lambda_mult` 값이 1에 가까울수록 관련성, 0에 가까울수록 다양성을 우선한다.
- 대용량 데이터셋에서는 Chroma 대신 Pinecone, Weaviate, pgvector 등 분산 환경을 지원하는 벡터 DB로 전환을 고려해야 한다.
- LangChain의 `as_retriever()`는 LCEL(LangChain Expression Language) 체인에 바로 연결 가능해 RAG 파이프라인 구성이 간결해진다.