# 엔지니어 역할의 대전환: AI 주도 개발 (Harness Engineering)

## 📌 Context

전통적인 소프트웨어 개발에서 엔지니어는 코드를 한 줄씩 직접 작성하는 역할을 담당했다. 하지만 AI 코딩 에이전트(예: OpenAI Codex, GitHub Copilot Workspace 등)가 고도화됨에 따라 **Harness Engineering** 환경이 등장하였다.

Harness Engineering 환경에서는 사람이 코드를 단 한 줄도 직접 작성하지 않는다. 대신 사람은 비즈니스 목표와 요구사항을 정의하고, AI가 전체 개발 주기를 자율적으로 수행한다. 이는 엔지니어의 역할 자체가 근본적으로 재정의되는 패러다임 전환이다.

## ⚙️ Core

### 역할 분리 구조

```
┌─────────────────────────────────────────────┐
│              Human (엔지니어)                │
│  - 비즈니스 목표 설정                        │
│  - 사용자 요구사항 → 수용 기준(AC) 변환      │
│  - 최종 결과물 검증 및 승인                  │
└────────────────────┬────────────────────────┘
                     │ Acceptance Criteria
                     ▼
┌─────────────────────────────────────────────┐
│              AI Agent (예: Codex)            │
│  - 코드 작성                                 │
│  - 테스트 작성 및 실행                       │
│  - CI/CD 구성                                │
│  - 문서화 (README, API Docs 등)              │
│  - PR 생성 및 자체 병합                      │
└─────────────────────────────────────────────┘
```

### 수용 기준(Acceptance Criteria) 작성 예시

```markdown
## Feature: 사용자 로그인 API

### Acceptance Criteria
- [ ] POST /api/auth/login 엔드포인트가 존재해야 한다
- [ ] 올바른 자격증명으로 요청 시 JWT 토큰을 반환해야 한다
- [ ] 잘못된 자격증명으로 요청 시 401 상태 코드를 반환해야 한다
- [ ] 응답 시간은 200ms 이내여야 한다
- [ ] 단위 테스트 커버리지 80% 이상
```

### AI 에이전트 워크플로우 (개념적 흐름)

```yaml
# harness-workflow.yaml (개념 예시)
workflow:
  trigger: acceptance_criteria_provided
  steps:
    - name: analyze_requirements
      agent: codex
      action: parse_acceptance_criteria

    - name: implement_feature
      agent: codex
      action: write_code
      output: src/

    - name: write_tests
      agent: codex
      action: generate_tests
      coverage_target: 80

    - name: run_ci
      action: execute_pipeline
      steps: [lint, test, build]

    - name: generate_docs
      agent: codex
      action: write_documentation

    - name: create_and_merge_pr
      agent: codex
      action: submit_pull_request
      auto_merge: true
      condition: all_checks_passed
```

## 💡 Insight

- **엔지니어의 핵심 역량 이동**: 코딩 스킬보다 **요구사항 정의 능력**과 **수용 기준(AC) 작성 능력**이 더 중요해진다. 모호한 AC는 AI가 엉뚱한 코드를 생성하는 원인이 된다.
- **검증자로서의 엔지니어**: AI가 생성한 코드·테스트·문서가 비즈니스 의도에 부합하는지 판단하는 역할이 핵심이 된다. 코드를 읽는 능력은 여전히 필수이다.
- **트레이드오프**: AI 자율 병합(auto-merge)은 속도를 높이지만, 예상치 못한 부작용이나 보안 취약점이 검토 없이 배포될 위험이 있다. 실무에서는 critical 경로에 대해 인간 검토 단계를 유지하는 하이브리드 접근이 현실적이다.
- **조직 구조 변화**: 동일한 비즈니스 산출물을 훨씬 적은 인원으로 달성 가능해지며, 엔지니어 1인이 여러 AI 에이전트를 동시에 지휘하는 **에이전트 오케스트레이터** 역할이 부상할 것으로 예상된다.
- **현재 한계**: 복잡한 아키텍처 결정, 레거시 시스템과의 통합, 조직 맥락이 필요한 판단은 아직 AI가 완전히 대체하기 어렵다.