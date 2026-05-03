# Claude에서 PPT 작업 워크플로우

## 📌 Context
Claude를 활용해 PPT 슬라이드를 제작할 때, 일관된 디자인 시스템을 적용하기 위한 프롬프트 설정 워크플로우를 정리한다. getdesign.md 템플릿을 기반으로 Brandlogy 브랜드 가이드와 Pretendard 폰트를 적용한 커스텀 디자인 시스템 프롬프트를 구성하는 방법이다.

## ⚙️ Core

### 워크플로우
```
1. getdesign.md 접속
2. 원하는 템플릿 선택
3. 템플릿 프롬프트 복사
4. 아래 수정 요청을 Claude에 전달
```

### Claude 프롬프트 수정 요청 템플릿
```text
나는 지금 Claude에서 PPT를 만들기 위한 프롬프트이자 디자인 시스템 설정 프롬프트를 쓰는 중이야.
아래 내 요청을 반영해서 프롬프트를 수정해줘.

- 폰트는 오직 Pretendard만 사용해야 하며, 다른 폰트는 사용하지 않음 (업로드 파일 참고)
- 로고는 MiniMax 것이 아니라 Brandlogy 것을 사용 (지식 베이스에 업로드 예정)
- 슬라이드 비율은 오직 16:9만 사용
- 챕터명, 제목, 부제목이 매 페이지마다 동일한 위치에 배치
- 본문 하단을 비워두지 말고 디자인에 악영향을 끼치지 않는 선에서 밀도 있게 채울 것
- 프롬프트 언어는 영어로 유지
```

### 디자인 시스템 핵심 설정 요약
```yaml
font: Pretendard (exclusive)
logo: Brandlogy
slide_ratio: 16:9
layout:
  chapter_title: fixed position per page
  title: fixed position per page
  subtitle: fixed position per page
content_density: high (no empty bottom areas)
prompt_language: English
```

## 💡 Insight
- getdesign.md 템플릿을 직접 수정하지 않고 Claude에게 수정 요청을 위임하는 방식이므로, 재사용성과 유연성이 높다.
- 폰트와 로고를 지식 베이스(파일 업로드)로 관리하면 세션이 바뀌어도 일관된 브랜드 아이덴티티를 유지할 수 있다.
- 레이아웃 고정 요청(`매 페이지 같은 위치`)은 슬라이드 일관성을 높이지만, 콘텐츠 양에 따라 여백 처리 규칙과 충돌할 수 있으므로 우선순위를 명확히 지정하는 것이 좋다.
- 프롬프트를 영어로 유지하면 Claude의 디자인 관련 응답 품질이 더 안정적인 경향이 있다.