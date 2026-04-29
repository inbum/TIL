# Vertical Slice Architecture vs Clean Architecture (바이브 코딩 관점)

## 📌 Context

AI 어시스턴트를 활용한 바이브 코딩(Vibe Coding) 방식으로 프로젝트를 빠르게 진행할 때, 아키텍처 선택은 개발 속도와 코드 품질에 큰 영향을 준다. 두 아키텍처의 철학과 구조적 차이를 이해하고, 바이브 코딩 흐름에서 어떤 방식이 더 효율적인지 판단하기 위해 정리했다.

## ⚙️ Core

### 구조 비교

**Clean Architecture (레이어 기반)**
```
src/
├── domain/          # Entities, Value Objects
├── application/     # Use Cases, Ports
├── infrastructure/  # DB, External APIs
└── presentation/    # Controllers, DTOs
```

**Vertical Slice Architecture (기능 기반)**
```
src/
├── features/
│   ├── CreateOrder/
│   │   ├── CreateOrderCommand.ts
│   │   ├── CreateOrderHandler.ts
│   │   ├── CreateOrderController.ts
│   │   └── CreateOrderRepository.ts
│   └── GetOrderById/
│       ├── GetOrderByIdQuery.ts
│       ├── GetOrderByIdHandler.ts
│       └── GetOrderByIdController.ts
```

### 핵심 코드 예시 (Vertical Slice)

```typescript
// features/CreateOrder/CreateOrderHandler.ts
import { PrismaClient } from '@prisma/client';

interface CreateOrderCommand {
  userId: string;
  items: { productId: string; quantity: number }[];
}

export class CreateOrderHandler {
  constructor(private db: PrismaClient) {}

  async handle(command: CreateOrderCommand) {
    const order = await this.db.order.create({
      data: {
        userId: command.userId,
        items: { create: command.items },
        status: 'PENDING',
      },
    });
    return order;
  }
}
```

### 장단점 비교표

| 항목 | Vertical Slice | Clean Architecture |
|---|---|---|
| 바이브 코딩 속도 | ✅ 빠름 (기능 단위 집중) | ❌ 느림 (레이어 세팅 필요) |
| AI 코드 생성 적합성 | ✅ 높음 (context 집중) | ⚠️ 보통 (레이어 간 의존 파악 필요) |
| 초기 구조 복잡도 | ✅ 낮음 | ❌ 높음 |
| 코드 중복 위험 | ⚠️ 있음 (공통 로직 분산) | ✅ 낮음 |
| 장기 유지보수성 | ⚠️ 슬라이스 비대화 위험 | ✅ 높음 |
| 테스트 단순성 | ✅ 슬라이스 단위 테스트 쉬움 | ⚠️ Mock 설계 필요 |
| 팀 확장성 | ✅ 병렬 개발 쉬움 | ✅ 높음 |

## 💡 Insight

- **바이브 코딩에는 Vertical Slice가 유리하다.** AI에게 "주문 생성 기능 만들어줘"라고 요청할 때, 레이어를 넘나드는 Clean Architecture보다 한 폴더에 응집된 슬라이스 구조가 프롬프트 컨텍스트 효율이 높다.
- **Clean Architecture는 규모가 커질수록 빛난다.** 도메인 로직이 복잡하거나 외부 의존성 교체 가능성이 있는 프로젝트에는 여전히 Clean Architecture가 적합하다.
- **하이브리드 전략:** 초기 바이브 코딩 단계에서 Vertical Slice로 빠르게 프로토타이핑하고, 도메인이 안정화되면 공통 레이어를 추출하는 방식이 실무적으로 효과적이다.
- **주의점:** Vertical Slice는 슬라이스 간 공통 로직(인증, 로깅, 에러 핸들링)을 어디에 둘지 초반에 명확히 정하지 않으면 중복 코드가 빠르게 쌓인다.