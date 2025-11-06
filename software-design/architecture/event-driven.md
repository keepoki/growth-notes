#Architecture

# Event-Driven

이벤트 기반 아키텍처는 시스템이 **이벤트(event)** 를 중심으로 동작하는 구조입니다.

- **Event**: 시스템 상태 변화나 발생한 사실 (예: "사용자가 회원가입했다")
- **Producer(발행자)**: 이벤트를 발생시키는 주체
- **Consumer(구독자)**: 이벤트를 구독하고 처리하는 주체
- **Event Bus/Broker**: 이벤트를 전달하는 중개자 (Kafka, RabbitMQ, Redis Pub/Sub 등)

**컴포넌트 간 강한 결합을 피하고, 느슨하게 연결된 시스템을 만든다.**

## 기본 구현 (Node.js EventEmitter)

```js
const EventEmitter = require("events");

class MyEventBus extends EventEmitter {}
const eventBus = new MyEventBus();

// Consumer (구독자)
eventBus.on("userRegistered", (user) => {
  console.log("Send welcome email to:", user.name);
});

eventBus.on("userRegistered", (user) => {
  console.log("Add user to analytics system:", user.name);
});

// Producer (발행자)
function registerUser(name) {
  const user = { id: Date.now(), name };
  console.log("User registered:", user.name);
  eventBus.emit("userRegistered", user); // 이벤트 발행
}

// 실행
registerUser("Alice");
registerUser("Bob");
```

## 실제 활용 예시 (Kafka 스타일)

```js
// Producer (회원가입 서비스)
const { Kafka } = require("kafkajs");
const kafka = new Kafka({ clientId: "app", brokers: ["localhost:9092"] });
const producer = kafka.producer();

async function registerUser(name) {
  await producer.connect();
  await producer.send({
    topic: "userRegistered",
    messages: [{ value: JSON.stringify({ name }) }],
  });
  console.log("User Registered:", name);
  await producer.disconnect();
}

registerUser("Alice");
```

```js
// Consumer (이메일 서비스)
const { Kafka } = require("kafkajs");
const kafka = new Kafka({ clientId: "app", brokers: ["localhost:9092"] });
const consumer = kafka.consumer({ groupId: "email-service" });

async function start() {
  await consumer.connect();
  await consumer.subscribe({ topic: "userRegistered" });
  await consumer.run({
    eachMessage: async ({ message }) => {
      const user = JSON.parse(message.value.toString());
      console.log("Send welcome email to:", user.name);
    },
  });
}

start();
```

## 언제 사용하면 좋은가?

- 서비스 간 **실시간 반응**이 필요한 경우 (알림, 채팅, 로그 처리)
- 마이크로서비스 아키텍처에서 서비스 간 결합도를 낮추고 싶을 때
- **비동기 처리**가 중요한 경우 (주문 → 결제 → 배송 같은 워크플로우)
- **확장성**이 중요한 경우 (구독자를 쉽게 추가 가능)

## 정리

**장점**
- 서비스 간 결합도 ↓ (유연하고 확장 쉬움)
- 실시간 반응 처리 가능
- 새로운 소비자(Consumer)를 추가하기 쉬움
**단점**
- 이벤트 순서 보장 어려움
- 디버깅 및 모니터링 복잡
- 메시지 브로커 관리 비용 발생
**사용 예시**
- Kafka, RabbitMQ, AWS SNS/SQS, Redis Pub/Sub
- 실시간 로그 수집, 알림 서비스, IoT 데이터 처리, 결제/주문 워크플로우
