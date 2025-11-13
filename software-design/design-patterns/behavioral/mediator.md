---
tags:
  - DesignPattern
  - Behavioral
title: Mediator Pattern
---

객체 간의 직접적인 상호작용을 피하고, **중재자(Mediator)** 객체를 통해서만 통신하게 하는 패턴입니다.
→ 객체들 간의 결합도를 낮추고, 복잡한 상호작용을 중앙에서 관리할 수 있습니다.

## 기본 구현

```js
class Mediator {
  constructor() {
    this.colleagues = [];
  }

  add(colleague) {
    this.colleagues.push(colleague);
    colleague.setMediator(this);
  }

  send(message, sender) {
    this.colleagues.forEach(c => {
      if (c !== sender) {
        c.receive(message);
      }
    });
  }
}

class Colleague {
  constructor(name) {
    this.name = name;
    this.mediator = null;
  }

  setMediator(mediator) {
    this.mediator = mediator;
  }

  send(message) {
    console.log(`${this.name} 보냄: ${message}`);
    this.mediator.send(message, this);
  }

  receive(message) {
    console.log(`${this.name} 받음: ${message}`);
  }
}

// 사용
const mediator = new Mediator();

const user1 = new Colleague("User1");
const user2 = new Colleague("User2");
const user3 = new Colleague("User3");

mediator.add(user1);
mediator.add(user2);
mediator.add(user3);

user1.send("안녕하세요!");

// 출력
// User1 보냄: 안녕하세요!
// User2 받음: 안녕하세요!
// User3 받음: 안녕하세요!
```

## 실제 활용 코드 예시

### 채팅방 (Mediator = 채팅방, Colleage = 유저)

```js
class ChatRoom {
  showMessage(user, message) {
    const time = new Date().toLocaleTimeString();
    console.log(`[${time}] ${user.name}: ${message}`);
  }
}

class User {
  constructor(name, chatRoom) {
    this.name = name;
    this.chatRoom = chatRoom;
  }

  send(message) {
    this.chatRoom.showMessage(this, message);
  }
}

// 사용
const chatRoom = new ChatRoom();

const alice = new User("Alice", chatRoom);
const bob = new User("Bob", chatRoom);

alice.send("안녕 Bob!");
bob.send("안녕 Alice!");
```

### UI 컴포넌트 간의 중재

```js
class FormMediator {
  constructor() {
    this.usernameInput = null;
    this.passwordInput = null;
    this.submitButton = null;
  }

  notify(sender, event) {
    if (sender === this.usernameInput || sender === this.passwordInput) {
      const canSubmit =
        this.usernameInput.value && this.passwordInput.value.length >= 6;
      this.submitButton.setEnabled(canSubmit);
    }
  }
}

class Input {
  constructor(mediator) {
    this.value = "";
    this.mediator = mediator;
  }

  setValue(value) {
    this.value = value;
    this.mediator.notify(this, "change");
  }
}

class Button {
  constructor() {
    this.enabled = false;
  }

  setEnabled(enabled) {
    this.enabled = enabled;
    console.log(`버튼 상태: ${enabled ? "활성화" : "비활성화"}`);
  }
}

// 사용
const mediator2 = new FormMediator();
const username = new Input(mediator2);
const password = new Input(mediator2);
const submit = new Button();

mediator2.usernameInput = username;
mediator2.passwordInput = password;
mediator2.submitButton = submit;

username.setValue("testUser");
password.setValue("12345"); // 버튼 비활성화
password.setValue("123456"); // 버튼 활성화
```

## 언제 사용하면 좋은가?

- 객체 간의 **상호작용이 복잡할 때** → 코드가 얽히지 않게 중앙에서 관리
- **UI 컴포넌트 간의 의존성을 줄이고 싶을 때**
- **채팅방, 이벤트 시스템, Pub/Sub 구조** 구현할 때

## 정리

- Mediator는 객체 간의 직접 참조를 없애고 **중앙 허브를 통해 통신**하게 합니다.
- 결합도 낮추기 좋지만, **Mediator 자체가 비대해지는 단점**이 있습니다.

## 관련 개념

- [[observer]]: 이벤트 기반 통신 패턴
- [[facade]]: 복잡한 시스템을 단순화하는 패턴
- [[event-bus]]: Mediator 패턴의 구현 예시
- [[publish-subscribe]]: 메시지 중개 패턴
- [[designpattern]]: 디자인 패턴 전체 개요
