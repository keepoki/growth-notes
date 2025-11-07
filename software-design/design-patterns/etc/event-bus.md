#DesignPattern

# Event Bus

애플리케이션 내에서 **컴포넌트 간 메시지(이벤트)를 중개하는 중앙 통신 허브**입니다.
발행-구독(Pub/Sub) 패턴의 한 구현으로 볼 수 있습니다.

- 컴포넌트 간 직접 참조 없이 → **느슨한 결합**
- UI 프레임워크(React, Vue, Angular)나 Node.js 환경에서 자주 쓰임

## 기본 구현

```js
class EventBus {
  constructor() {
    this.events = {};
  }

  on(event, listener) {
    if (!this.events[event]) this.events[event] = [];
    this.events[event].push(listener);
  }

  off(event, listener) {
    this.events[event] = this.events[event].filter(l => l !== listener);
  }

  emit(event, data) {
    if (this.events[event]) {
      this.events[event].forEach(listener => listener(data));
    }
  }
}

// 사용
const bus = new EventBus();

function onUserLogin(user) {
  console.log("✅ 로그인:", user.name);
}
bus.on("login", onUserLogin);

bus.emit("login", { id: 1, name: "경준" });
bus.off("login", onUserLogin);
```

## 실제 활용 코드 예시

### Vue.js 전역 Event Bus

```js
// event-bus.js
import Vue from "vue";
export const EventBus = new Vue();

// ComponentA.vue
import { EventBus } from "./event-bus";
EventBus.$emit("user-logged-in", { name: "경준" });

// ComponentB.vue
import { EventBus } from "./event-bus";
EventBus.$on("user-logged-in", (user) => {
  console.log("다른 컴포넌트에서 감지:", user.name);
});
```

### React 외부 Event Bus 활용

```js
// eventBus.js
import mitt from "mitt";
export const eventBus = mitt();

// ComponentA.jsx
import { eventBus } from "./eventBus";
eventBus.emit("notify", "새 알림 도착");

// ComponentB.jsx
import { eventBus } from "./eventBus";
eventBus.on("notify", (msg) => {
  console.log("알림 받음:", msg);
});
```

### Node.js 내장 EventEmitter 활용

```js
import { EventEmitter } from "events";
const bus = new EventEmitter();

bus.on("orderCreated", (order) => {
  console.log("📦 주문 처리:", order.id);
});

bus.emit("orderCreated", { id: 123, product: "노트북" });
```

## 언제 사용하면 좋은가?

- **컴포넌트 간 직접 의존성을 줄이고 싶을 때**
- **여러 모듈이 동일한 이벤트를 감지해야 할 때**
- 예:
  - 알림(Notification) 시스템
  - 사용자 로그인/로그아웃 이벤트 공유
  - 게임 이벤트 (플레이어 점프, 아이템 획득 등)
  - 실시간 채팅, 소켓 메시지 브로드캐스팅

## 정리

- Event Bus = "앱 내부 이벤트 중개소"
- 장점: 느슨한 결합, 이벤트 중심 아키텍처 구현 용이
- 단점: 이벤트 흐름이 복잡해지면 추적 어려움 (디버깅 주의)
