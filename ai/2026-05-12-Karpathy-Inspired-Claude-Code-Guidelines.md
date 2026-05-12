# Karpathy-Inspired Claude Code Guidelines

## 📌 Context

LLM 기반 코딩 어시스턴트(Claude Code 등)를 실무에 도입할 때 반복적으로 발생하는 세 가지 실패 패턴이 있다.

1. **Silent Assumptions** — 모호한 요청에 대해 모델이 내부적으로 해석을 결정하고 알리지 않음
2. **Overcomplication** — 요청 범위를 벗어난 추상화·방어 코드·미래 대비 코드를 자의적으로 추가
3. **Unintended Edits** — 수정 요청 범위 밖의 코드까지 변경

Andrej Karpathy의 실무 관찰에서 착안한 [andrej-karpathy-skills](https://github.com/forrestchang/andrej-karpathy-skills) 레포지터리는 이 문제를 해결하는 4가지 원칙을 제시하며, Claude Code Plugin / CLAUDE.md / Cursor 등 다양한 방식으로 프로젝트에 적용할 수 있다.

## ⚙️ Core

### 4가지 핵심 원칙

#### 1. Think Before Coding — 코딩 전 명시적 사고
```
- 요청이 모호하면 가능한 해석을 나열하고 확인을 요청한다
- 가정(assumption)을 코드 안에 숨기지 말고 먼저 말로 명시한다
- 추측으로 진행하지 않는다
```

#### 2. Simplicity First — 최소한의 코드
```
- 요청된 문제만 해결하는 가장 단순한 코드를 작성한다
- 사용되지 않는 추상화, 투기적 기능, 불가능한 시나리오에 대한 에러 핸들링은 추가하지 않는다
- 기준: 시니어 엔지니어가 "과하다"고 느끼면 과한 것이다
```

#### 3. Surgical Changes — 최소 범위 변경
```
- 요청받은 부분만 수정한다
- 기존 코드 스타일을 그대로 따른다
- 내 변경으로 인해 생긴 dead code만 제거하고,
  기존에 있던 dead code는 건드리지 않는다
```

#### 4. Goal-Driven Execution — 성공 기준 중심 실행
```
# 나쁜 예 (imperative)
"validation을 추가해줘"

# 좋은 예 (goal-driven)
"잘못된 입력에 대한 테스트를 먼저 작성하고,
 그 테스트를 통과하도록 validation을 구현해줘"
```

### CLAUDE.md 적용 예시

```markdown
# Project Guidelines

## AI Coding Rules (Karpathy-Inspired)

1. 요청이 모호하면 먼저 해석을 나열하고 확인 후 진행
2. 요청 범위 내 최소한의 코드만 작성
3. 요청 범위 밖 코드는 절대 수정하지 않음
4. 성공 기준(테스트/검증)을 먼저 정의하고 구현
```

## 💡 Insight

- **LLM은 루프에 강하다**: Karpathy의 핵심 관찰은 "LLM에게 할 일을 지시하지 말고, 성공 기준을 주라"는 것이다. 명확한 기준이 있으면 모델이 스스로 반복하며 목표에 수렴한다.
- **가장 큰 실무 위험은 범위 초과 수정**이다. 리뷰 없이 머지하면 의도치 않은 사이드 이펙트가 발생한다. Surgical Changes 원칙이 이 리스크를 줄인다.
- **CLAUDE.md는 팀 컨벤션으로 관리하면 강력하다**: 프로젝트별 CLAUDE.md에 이 4원칙을 명문화하면 팀 전체가 일관된 AI 협업 방식을 공유할 수 있다.
- **한계**: 원칙을 지시했어도 모델이 100% 준수하지 않을 수 있다. 중요한 변경은 반드시 diff 리뷰를 거쳐야 한다.
- 레포 자체가 127k stars를 달성했다는 점은 이 문제가 개인적 불편이 아니라 업계 공통 과제임을 방증한다.