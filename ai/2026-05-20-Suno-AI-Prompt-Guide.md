# Suno AI 프롬프트 가이드

## 📌 Context
Suno AI로 음악을 생성할 때 막연한 키워드만 입력하면 원하는 장르나 분위기와 동떨어진 결과물이 나오는 경우가 많다. "Style of Music" 필드에 구조화된 공식을 적용하면 AI가 스타일을 명확하게 해석하여 의도에 맞는 음악을 생성할 확률이 크게 높아진다.

## ⚙️ Core
Suno AI "Style of Music" 필드에 아래 공식을 적용한다.

```
[장르] + [악기] + [분위기] + [보컬 성향]
```

**예시**
```
lo-fi hip hop, piano, mellow and relaxing, soft male vocals
cinematic orchestral, strings and brass, epic and intense, no vocals
indie pop, acoustic guitar, upbeat and joyful, female vocals with harmonies
K-pop, synthesizer and electric guitar, energetic and trendy, high-pitched female vocals
```

**작성 팁**
- 각 요소는 쉼표(,)로 구분
- 악기명은 구체적으로 명시 (e.g., `piano`, `electric guitar`, `violin`)
- 분위기 형용사는 2개 이상 조합하면 정확도 향상
- 보컬이 없는 경우 `no vocals` 또는 `instrumental` 명시

## 💡 Insight
- 공식 없이 단순 키워드만 입력하면 장르가 뒤섞이거나 예상치 못한 보컬 스타일이 적용되는 경우가 많음
- "Style of Music" 필드는 Lyrics 필드와 독립적으로 동작하므로 두 필드를 함께 세밀하게 조정해야 일관된 결과 도출 가능
- 동일한 프롬프트도 생성마다 결과가 달라질 수 있으므로 마음에 드는 버전은 즉시 즐겨찾기 저장
- 상업적 제작보다 아이디어 프로토타이핑, 개인 프로젝트 BGM, 영상 배경음 제작에 특히 실용적