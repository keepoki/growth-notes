#DesignPattern 

# State

객체의 내부 상태에 따라 행동을 바꾸는 패턴입니다.
→ `if/else`나 `switch`문으로 상태를 분기하지 않고, 상태를 **객체로 분리**해서 관리합니다.

## 기본 구현

```js
class State {
  handle(context) {}
}

class ConcreteStateA extends State {
  handle(context) {
    console.log("상태 A: 행동 실행");
    context.setState(new ConcreteStateB());
  }
}

class ConcreteStateB extends State {
  handle(context) {
    console.log("상태 B: 행동 실행");
    context.setState(new ConcreteStateA());
  }
}

class Context {
  constructor(state) {
    this.state = state;
  }

  setState(state) {
    this.state = state;
  }

  request() {
    this.state.handle(this);
  }
}

// 사용
const context = new Context(new ConcreteStateA());
context.request(); // 상태 A → 상태 B
context.request(); // 상태 B → 상태 A
```

## 실제 활용 코드 예시

### 교통 신호등

```js
class RedLight {
  handle(context) {
    console.log("🔴 빨간불: 멈추세요");
    context.setState(new GreenLight());
  }
}

class GreenLight {
  handle(context) {
    console.log("🟢 초록불: 지나가세요");
    context.setState(new YellowLight());
  }
}

class YellowLight {
  handle(context) {
    console.log("🟡 노란불: 주의하세요");
    context.setState(new RedLight());
  }
}

class TrafficLight {
  constructor() {
    this.state = new RedLight();
  }

  setState(state) {
    this.state = state;
  }

  change() {
    this.state.handle(this);
  }
}

// 사용
const light = new TrafficLight();
light.change(); // 🔴 → 🟢
light.change(); // 🟢 → 🟡
light.change(); // 🟡 → 🔴
```

### 미디어 플레이어 상태

```js
class PlayingState {
  pressButton(player) {
    console.log("⏸ 일시정지");
    player.setState(new PausedState());
  }
}

class PausedState {
  pressButton(player) {
    console.log("▶ 재생");
    player.setState(new PlayingState());
  }
}

class MediaPlayer {
  constructor() {
    this.state = new PausedState();
  }

  setState(state) {
    this.state = state;
  }

  pressButton() {
    this.state.pressButton(this);
  }
}

// 사용
const player = new MediaPlayer();
player.pressButton(); // ▶ 재생
player.pressButton(); // ⏸ 일시정지
```

## 언제 사용하면 좋은가?

- 객체의 행동이 **상태에 따라 달라질 때**
- `if/else`나 `switch`문으로 상태 분기를 자주 할 때 → 가독성/유지보수 힘들어질 때
- 상태 전환이 자주 일어나는 시스템 (UI, 게임, 워크플로우 등)

## 정리

- State는 "상태를 객체화"해서 **조건문 대신 다형성으로 행동 제어**.
- 예시: 교통 신호등, 미디어 플레이어, 워크플로우 단계 관리.
- 단점: 상태 클래스가 많아지면 구조가 복잡해질 수 있습니다.
