---
tags:
  - DesignPattern
  - DistributedSystems
  - EventDriven
title: Event Sourcing Pattern
---

**Event Sourcing**은 시스템의 상태(state)를 직접 저장하지 않고,  
그 상태를 만들기까지의 **모든 이벤트(event)** 를 기록해두는 방식입니다.

즉, 데이터베이스에 “현재 상태”를 저장하는 대신 **상태 변화의 히스토리(이벤트 로그)** 를 저장하고, 필요할 때 이 이벤트들을 다시 재생(replay)해서 상태를 복원합니다.

> “현재 상태 = 이벤트들의 누적 결과”

예를 들어 은행 계좌를 생각해봅시다.

- 초기 상태: 잔고 0
- 이벤트1: +100 입금
- 이벤트2: -30 출금
- 최종 상태 = 잔고 70

## 기본 구현

```js
class EventStore {
  constructor() {
    this.events = [];
  }

  save(event) {
    this.events.push(event);
  }

  getEvents() {
    return this.events;
  }
}

class Account {
  constructor(id, eventStore) {
    this.id = id;
    this.eventStore = eventStore;
    this.balance = 0;
  }

  apply(event) {
    if (event.type === "deposit") this.balance += event.amount;
    if (event.type === "withdraw") this.balance -= event.amount;
  }

  replay() {
    this.balance = 0;
    for (const event of this.eventStore.getEvents()) {
      if (event.accountId === this.id) this.apply(event);
    }
  }

  deposit(amount) {
    this.eventStore.save({ accountId: this.id, type: "deposit", amount });
  }

  withdraw(amount) {
    this.eventStore.save({ accountId: this.id, type: "withdraw", amount });
  }
}

// 사용 예시
const store = new EventStore();
const account = new Account(1, store);

account.deposit(100);
account.withdraw(30);
account.replay();

console.log("현재 잔고:", account.balance); // 70
```

## 실제 활용 예시 (Node.js + 주문 시스템)

```js
// 예시 이벤트
[
  { type: "OrderCreated", orderId: 1, customer: "John" },
  { type: "ItemAdded", orderId: 1, item: "Keyboard" },
  { type: "ItemAdded", orderId: 1, item: "Mouse" },
  { type: "OrderPaid", orderId: 1 }
]

// 상태 복원
function rebuildOrder(events) {
  const state = { items: [], status: "pending" };
  for (const e of events) {
    if (e.type === "ItemAdded") state.items.push(e.item);
    if (e.type === "OrderPaid") state.status = "paid";
  }
  return state;
}
```

이렇게 하면 **이전 주문 이력이나 상태 변화의 이유**를 언제든지 추적할 수 있습니다.  
(예: “왜 이 주문이 취소됐지?” → 이벤트 로그를 보면 다 나옴)

## 언제 사용하면 좋은가?

- 데이터 변경 이력을 **정확히 추적해야 하는 시스템**  
    (금융, 회계, 거래, 주문 처리 등)
- 과거 상태 복원이 필요한 경우
- 여러 마이크로서비스가 동일한 이벤트 스트림을 **비동기적으로 처리**해야 할 때
- CQRS(Command Query Responsibility Segregation)와 함께 사용할 때 (상태 조회 성능 향상)

## 정리

### 장점

- 모든 변경 이력 기록 (감사 로그, 감사 추적에 탁월)
- 상태 복원 및 타임머신 기능
- 이벤트 기반 아키텍처와 자연스럽게 통합

### 단점

- 데이터 구조 복잡 (이벤트 스키마 버전 관리 필요)
- 쿼리가 어려움 → Projection 필요
- 이벤트 재생 시 성능 저하 가능

### 사용 예시

- CQRS와 함께: Command는 이벤트를 기록, Query는 Projection으로 조회
- Kafka, EventStoreDB, DynamoDB Streams 등으로 구현
- 은행 거래 내역, 주문 처리 시스템, IoT 센서 로그 등

### 요약

| 항목     | 내용                               |
| ------ | -------------------------------- |
| 핵심 개념  | 상태를 저장하지 않고, 이벤트 히스토리를 저장        |
| 장점     | 변경 이력 추적, 복원 가능, 이벤트 기반 아키텍처 친화적 |
| 단점     | 복잡한 구조, 조회 시 별도 Projection 필요    |
| 대표 사용처 | 금융, 주문, IoT, CQRS 기반 시스템         |

## 관련 개념

- [[cqrs-(command-query-responsibility-segregation)]]: Event Sourcing과 함께 사용되는 패턴
- [[event-driven]]: 이벤트 기반 아키텍처
- [[saga]]: 분산 트랜잭션 관리
- [[microservices]]: Event Sourcing이 사용되는 환경
- [[server-architecture]]: 이벤트 소싱을 사용하는 서버 아키텍처
