#DesignPattern 

# Observer

객체의 상태 변화를 관찰하는 **옵저버(Observer)들**이 있고,  주체(Subject)의 상태가 바뀌면 옵저버들에게 자동으로 알림을 보내는 패턴입니다.
→ 이벤트 리스너, Pub/Sub 모델의 기초.

## 기본 구현

```js
class Subject {
  constructor() {
    this.observers = [];
  }

  attach(observer) {
    this.observers.push(observer);
  }

  detach(observer) {
    this.observers = this.observers.filter(o => o !== observer);
  }

  notify(data) {
    this.observers.forEach(o => o.update(data));
  }
}

class Observer {
  constructor(name) {
    this.name = name;
  }

  update(data) {
    console.log(`${this.name}가 알림 받음: ${data}`);
  }
}

// 사용
const subject = new Subject();
const obs1 = new Observer("Observer1");
const obs2 = new Observer("Observer2");

subject.attach(obs1);
subject.attach(obs2);

subject.notify("상태 변경됨!");

// 출력
// Observer1가 알림 받음: 상태 변경됨!
// Observer2가 알림 받음: 상태 변경됨!
```

## 실제 활용 코드 예시

### 뉴스레터 구독 시스템

```js
class NewsPublisher {
  constructor() {
    this.subscribers = [];
  }

  subscribe(subscriber) {
    this.subscribers.push(subscriber);
  }

  unsubscribe(subscriber) {
    this.subscribers = this.subscribers.filter(s => s !== subscriber);
  }

  publish(news) {
    this.subscribers.forEach(s => s.update(news));
  }
}

class Subscriber {
  constructor(name) {
    this.name = name;
  }

  update(news) {
    console.log(`${this.name}가 뉴스 받음: ${news}`);
  }
}

// 사용
const publisher = new NewsPublisher();
const alice = new Subscriber("Alice");
const bob = new Subscriber("Bob");

publisher.subscribe(alice);
publisher.subscribe(bob);

publisher.publish("오늘의 뉴스: 디자인 패턴 공부!");
```

### DOM 이벤트 리스너 (Obsever 패턴 내장 활용)

```js
// 버튼 클릭 이벤트: Subject = 버튼, Observer = 리스너
const button = document.createElement("button");
button.textContent = "Click me";
document.body.appendChild(button);

button.addEventListener("click", () => {
  console.log("Observer1: 버튼이 클릭됨!");
});

button.addEventListener("click", () => {
  console.log("Observer2: 버튼 클릭에 반응!");
});
```

### 간단한 Pub/Sub 구현

```js
class EventBus {
  constructor() {
    this.events = {};
  }

  subscribe(event, callback) {
    if (!this.events[event]) this.events[event] = [];
    this.events[event].push(callback);
  }

  publish(event, data) {
    if (this.events[event]) {
      this.events[event].forEach(cb => cb(data));
    }
  }
}

// 사용
const bus = new EventBus();

bus.subscribe("login", user => console.log(`${user} 로그인`));
bus.subscribe("login", user => console.log(`환영합니다, ${user}!`));

bus.publish("login", "Alice");
```

## 언제 사용하면 좋은가?

- **이벤트 기반 시스템** (UI 이벤트, WebSocket 이벤트 등)
- **실시간 알림 시스템** (뉴스레터, 주식 알림, 채팅 알림)
- **Pub/Sub 구조** 구현할 때

## 정리

- Observer는 "상태 변화 → 자동 알림" 구조입니다.
- JS의 `addEventListener`, RxJS, Vue의 반응형 시스템, React의 상태 관리에도 활용됩니다.
- 단점: 옵저버가 많아지면 성능 저하 및 디버깅이 어렵습니다.