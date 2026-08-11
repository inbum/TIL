# Gemini Notebook 슬라이드 수정 방법

## 📌 Context
Gemini Notebook에서 생성된 슬라이드는 직접 편집이 불가능한 경우가 많다. 이를 수정하려면 PDF로 내보낸 뒤, Canva의 Magic Layers 기능을 활용해 편집 가능한 형태로 변환하는 우회 방법을 사용한다.

## ⚙️ Core

```text
[워크플로우]

1. Gemini Notebook에서 슬라이드 생성 완료
   ↓
2. 슬라이드를 PDF로 다운로드
   - 메뉴 > 파일 > 다운로드 > PDF 문서 (.pdf)
   ↓
3. Canva에서 PDF 가져오기
   - canva.com 접속 → 새 디자인 생성 또는 기존 프로젝트 열기
   - 업로드 탭 > PDF 파일 업로드
   ↓
4. Magic Layers 변환 적용
   - 업로드된 PDF 슬라이드 선택
   - 우클릭 또는 편집 메뉴 > "Magic Layers" 적용
   - Canva가 PDF 요소를 레이어별로 분리·변환
   ↓
5. 개별 요소 수정
   - 텍스트, 이미지, 도형 등 각 레이어를 독립적으로 편집
```

## 💡 Insight
- Magic Layers는 PDF의 정적 요소를 Canva 편집 가능 객체로 변환해주지만, 폰트나 레이아웃이 완벽하게 재현되지 않을 수 있어 변환 후 검토가 필요하다.
- 복잡한 그래프나 차트는 이미지로 고정될 수 있으므로, 데이터 기반 시각화는 별도로 재작성하는 것이 낫다.
- Gemini Notebook → Canva 파이프라인은 AI 생성 콘텐츠를 빠르게 브랜딩/커스터마이징하는 실용적인 워크플로우로, 발표 자료 제작 속도를 크게 높일 수 있다.