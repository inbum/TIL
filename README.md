# 📚 TIL — Today I Learned

> 매일 배운 것을 기록합니다. 완벽한 문서가 아닌, 살아있는 학습 노트입니다.

---

## 🗂️ 개요

| 항목 | 내용 |
|------|------|
| 목적 | 학습한 기술 내용을 문서화하여 지식을 체계적으로 축적 |
| 대상 | IT 기술 전반 (AI, 자동화, 모바일, 파이썬 등) |
| 작성 도구 | n8n + Claude API를 활용한 자동 발행 파이프라인 |
| 작성 주기 | 학습이 발생하는 즉시 |

---

## 📁 지식 카테고리

> 카테고리로 분류가 어려운 경우, 새 폴더를 생성하여 관리합니다.

<details id="cat-ai">
<summary>🤖 ai — LLM, 프롬프트 엔지니어링, AI 도구 활용</summary>

<!-- list-start:ai -->
- [2026-04-29-LangChain-LangGraph-LangFlow-LangSmith-비교](ai/2026-04-29-LangChain-LangGraph-LangFlow-LangSmith-비교.md)
- [2026-04-29-gpt-image-2-model-performance-and-usecases](ai/2026-04-29-gpt-image-2-model-performance-and-usecases.md)
<!-- list-end:ai -->

</details>

<details id="cat-n8n">
<summary>🔀 n8n — 워크플로우 자동화, 파이프라인 구성</summary>

<!-- list-start:n8n -->
- [2026-04-29-n8n-discord-trigger-to-webhook](n8n/2026-04-29-n8n-discord-trigger-to-webhook.md)
<!-- list-end:n8n -->

</details>

<details id="cat-python">
<summary>🐍 python — 파이썬 문법, 라이브러리, 스크립트</summary>

<!-- list-start:python -->
<!-- list-end:python -->

</details>

<details id="cat-mobile">
<summary>📱 mobile — iOS / Android 개발</summary>

<!-- list-start:mobile -->
<!-- list-end:mobile -->

</details>

<details id="cat-etc">
<summary>📦 etc — 기타 기술 주제</summary>

<!-- list-start:etc -->
- [2026-04-29-Replit-vibe-coding-app-development-and-deploy](etc/2026-04-29-Replit-vibe-coding-app-development-and-deploy.md)
- [2026-04-29-vertical-slice-vs-clean-architecture-vibe-coding](etc/2026-04-29-vertical-slice-vs-clean-architecture-vibe-coding.md)
<!-- list-end:etc -->

</details>

---

## 📄 문서 구조

모든 TIL 문서는 아래 3개 섹션으로 구성됩니다.

### 📌 Context
학습 배경 또는 해결하려는 문제 상황을 기술합니다.

### ⚙️ Core
실제 작동하는 코드 스니펫, 핵심 로직, 설정값 등을 포함합니다.

### 💡 Insight
실행 결과에 대한 기술적 검증과 개인적인 견해를 정리합니다.

---

## 📝 작성 및 관리 규칙

### 파일 명명 규칙
- 형식: `YYYY-MM-DD-제목.md`
- 공백은 하이픈(`-`)으로 대체
- 특수문자 사용 금지

```
✅ 2026-04-29-n8n-github-til-auto-publish.md
❌ 2026.04.29_n8n GitHub TIL.md
```

### 작성 원칙
- 그날 배운 내용을 **당일 또는 최대한 빠르게** 기록한다
- 완벽한 문서보다 **핵심을 담은 실용적인 기록**을 우선한다
- 코드는 반드시 **실제 동작 가능한 수준**으로 작성한다
- 파편적인 메모도 기술적 맥락을 보강하여 완결성을 높인다

### 자동화 파이프라인
이 레포지토리는 **n8n + Claude API** 기반의 자동 발행 파이프라인으로 운영됩니다.

```
입력 (DISCORD)
    ↓
Claude API — TIL 문서 자동 생성
    ↓
GitHub Contents API — 파일 커밋 & 푸시
```

---


## 🔗 참고

- [GitHub TIL 레포지토리](https://github.com/inbum/TIL)
- 작성자: [@inbum](https://github.com/inbum)
