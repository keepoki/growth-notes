---
tags:
  - DesignPattern
  - Messaging
title: Publish-Subscribe Pattern
---

**Publish–Subscribe (Pub/Sub)** 패턴은 **발행자(Publisher)** 와 **구독자(Subscriber)** 가 서로 직접 연결되지 않고, **중앙 이벤트 브로커(Event Bus)** 를 통해 메시지를 주고받는 구조입니다.

> “한쪽은 메시지를 발행(publish)하고,  
> 다른 쪽은 필요한 메시지만 구독(subscribe)한다.”

발행자와 구독자가 **서로를 몰라도 통신 가능**
시스템 간 **결합도(coupling)** 를 크게 낮출 수 있음

## 기본 구현

```js
class EventBus {
  constructor() {
    this.subscribers = {};
  }

  subscribe(eventType, callback) {
    if (!this.subscribers[eventType]) {
      this.subscribers[eventType] = [];
    }
    this.subscribers[eventType].push(callback);
  }

  publish(eventType, data) {
    const subscribers = this.subscribers[eventType] || [];
    for (const callback of subscribers) {
      callback(data);
    }
  }

  unsubscribe(eventType, callback) {
    this.subscribers[eventType] = (this.subscribers[eventType] || []).filter(
      (cb) => cb !== callback
    );
  }
}
```

## 실제 활용 예시 (프론트엔드 / 백엔드)

React 앱 내 전역 이벤트 버스

```js
// eventBus.js
export const eventBus = new EventBus();

// Component A (발행자)
import { eventBus } from "./eventBus";
function Button() {
  return <button onClick={() => eventBus.publish("USER_LOGOUT", null)}>Logout</button>;
}

// Component B (구독자)
import { useEffect } from "react";
import { eventBus } from "./eventBus";

function Header() {
  useEffect(() => {
    eventBus.subscribe("USER_LOGOUT", () => console.log("Header: user logged out"));
  }, []);

  return <h1>Welcome</h1>;
}
```

서로 관계없는 컴포넌트 간에 **전역 상태 없이도 이벤트를 주고받을 수 있음**

Node.js 마크로서비스 간 메시징

```js
// message-bus.js
const EventEmitter = require("events");
const bus = new EventEmitter();

// OrderService
bus.on("order.created", (order) => {
  console.log("📦 New order created:", order);
});

// PaymentService
function processPayment(order) {
  console.log("💳 Processing payment for:", order.id);
  bus.emit("payment.completed", { orderId: order.id });
}

// Main
const order = { id: 1, item: "Keyboard" };
bus.emit("order.created", order);
processPayment(order);
```

Node의 `EventEmitter` 자체가 Pub/Sub 구조
서비스 간 직접 호출 없이, 이벤트 중심으로 작동

## 언제 사용하면 좋은가?

- 여러 모듈/서비스가 느슨하게 연결되어야 할 때
- **이벤트 기반 아키텍처(Event-Driven Architecture)** 를 구현할 때
- 마이크로서비스 간 비동기 통신이 필요할 때
- React, Vue 등 프론트엔드에서 **전역 상태 없이 컴포넌트 간 통신**이 필요할 때

## 정리

### 장점

- 발행자와 구독자의 **결합도 최소화**
- 시스템 확장성, 유연성 향상
- 비동기 메시징에 적합 (이벤트 기반 시스템과 궁합 좋음)

### 단점

- 흐름 추적이 어려움 (누가 이벤트를 받는지 한눈에 안 보임)
- 디버깅, 로깅 복잡
- 메시지 손실 위험 (특히 네트워크 기반 브로커 사용 시)

### 사용 예시

- Node.js `EventEmitter`
- Frontend 이벤트 버스 (React, Vue, Svelte 등)
- Kafka, RabbitMQ, Redis Pub/Sub
- AWS SNS + SQS 조합

## 요약

|항목|내용|
|---|---|
|핵심 개념|발행자와 구독자가 이벤트 버스를 통해 간접 통신|
|장점|결합도 감소, 확장성 증가, 비동기 구조 용이|
|단점|이벤트 추적 어려움, 디버깅 복잡|
|대표 사용처|Frontend 이벤트 통신, 마이크로서비스 메시징, Kafka/RabbitMQ|

## 관련 개념

- [[observer]]: Observer 패턴의 확장 버전
- [[event-bus]]: Pub/Sub 구현 예시
- [[mediator]]: 중재자를 통한 통신 패턴
- [[event-driven]]: 이벤트 기반 아키텍처
- [[designpattern]]: 디자인 패턴 전체 개요
