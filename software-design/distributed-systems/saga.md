---
tags:
  - DesignPattern
  - DistributedSystems
  - Transaction
title: Saga Pattern
---

**Saga 패턴**은 **분산 트랜잭션 관리 패턴**으로,  
마이크로서비스 환경에서 **여러 서비스에 걸친 데이터 일관성을 유지**하기 위해 사용됩니다.
> “하나의 큰 트랜잭션을 여러 개의 **로컬 트랜잭션**(Saga)으로 나누고,  
> 중간에 실패하면 이전 단계를 **보상 트랜잭션(Compensating Transaction)** 으로 되돌린다.”

예를 들어,  `주문 서비스 → 결제 서비스 → 배송 서비스` 중 하나라도 실패하면,
이전 단계(결제 완료 등)를 **취소(rollback)** 하는 로직을 자동으로 실행합니다.

## 기본 구현

```js
class Saga {
  constructor(steps) {
    this.steps = steps;
  }

  async execute() {
    const executed = [];
    try {
      for (const step of this.steps) {
        await step.action();
        executed.push(step);
      }
      console.log("✅ Saga completed successfully");
    } catch (err) {
      console.log("❌ Saga failed:", err.message);
      // 보상 트랜잭션 수행 (역순)
      for (const step of executed.reverse()) {
        if (step.compensation) await step.compensation();
      }
      console.log("🔁 Compensating transactions executed.");
    }
  }
}

// 예시 단계
const saga = new Saga([
  {
    action: async () => {
      console.log("1️⃣ 주문 생성");
    },
    compensation: async () => {
      console.log("↩ 주문 취소");
    },
  },
  {
    action: async () => {
      console.log("2️⃣ 결제 승인");
      throw new Error("결제 실패!"); // 실패 발생
    },
    compensation: async () => {
      console.log("↩ 결제 취소");
    },
  },
  {
    action: async () => {
      console.log("3️⃣ 배송 요청");
    },
    compensation: async () => {
      console.log("↩ 배송 취소");
    },
  },
]);

saga.execute();
```

실행 결과:

```yaml
1️⃣ 주문 생성
2️⃣ 결제 승인
❌ Saga failed: 결제 실패!
↩ 주문 취소
🔁 Compensating transactions executed.
```

## 실제 활용 예시 (Node.js + Kafka 기반)

실무에서는 **이벤트 기반 아키텍처**(Kafka, RabbitMQ)와 결합해 동작합니다.

### 예시 흐름

1. 주문 서비스(Order) → `OrderCreated` 이벤트 발행
2. 결제 서비스(Payment) → `PaymentCompleted` 이벤트 수신 후 처리
3. 배송 서비스(Shipping) → `ShippingStarted` 이벤트 수신
4. 결제 실패 시 `PaymentFailed` 이벤트 발행 → 주문 서비스가 보상 트랜잭션(`CancelOrder`) 실행

```js
// Order Service
eventBus.on("PaymentFailed", (orderId) => {
  console.log("↩ 결제 실패 감지 → 주문 취소 수행:", orderId);
});
```

이처럼 Saga는 이벤트 체인을 통해 각 서비스의 상태를 **조율**합니다.

## 언제 사용하면 좋은가?

- 여러 서비스가 **하나의 비즈니스 프로세스**를 수행할 때 (예: 주문 → 결제 → 재고 → 배송)
- **트랜잭션 일관성(ACID)** 을 분산 환경에서 보장해야 할 때
- 중앙 DB 트랜잭션 대신 **비동기 이벤트 기반 보상 로직**으로 처리하고 싶을 때

## 정리

### 장점

- 분산 환경에서 데이터 일관성 유지
- 장애 복구 및 롤백 자동화 가능
- 중앙 트랜잭션 관리 없이 확장성 확보

### 단점

- 보상 트랜잭션 설계가 복잡함
- 오류 처리 로직이 많아짐 (특히 이벤트 기반일 때)
- 전체 프로세스 추적이 어려움

### 사용 예시

- 주문-결제-배송 시스템
- 은행 송금 (A → B 계좌 이체)
- 예약 시스템 (좌석 예약 후 결제 실패 시 자동 취소)

## 관련 개념

- [[microservices]]: Saga 패턴이 사용되는 환경
- [[event-driven]]: 이벤트 기반 Saga 구현
- [[event-sourcing]]: 이벤트 저장 패턴
- [[cqrs-(command-query-responsibility-segregation)]]: CQRS와 함께 사용
- [[unit-of-work]]: 로컬 트랜잭션 관리
