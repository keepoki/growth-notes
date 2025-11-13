---
tags:
  - DesignPattern
  - SafetyPattern
title: Null Object Pattern
---

`null` 대신 "아무 일도 하지 않는 기본 객체"를 제공해서, `null 체크` 분기문을 줄이는 패턴입니다.

## 기본 구현

```js
class User {
  constructor(name) {
    this.name = name;
  }
  greet() {
    return `Hello, ${this.name}`;
  }
}

class NullUser {
  greet() {
    return "Hello, Guest";
  }
}

function getUser(isLoggedIn) {
  if (isLoggedIn) {
    return new User("경준");
  }
  return new NullUser(); // null 대신
}

// 사용
const user1 = getUser(true);
console.log(user1.greet()); // Hello, 경준

const user2 = getUser(false);
console.log(user2.greet()); // Hello, Guest
```

## 실제 활용 코드 예시

### React 컴포넌트 Prop 기본값

```js
function Avatar({ user = new NullUser() }) {
  return <div>{user.greet()}</div>;
}
```

### Express 미들웨어 기본 동작

```js
const NullLogger = {
  log: () => {} // 아무 것도 안 함
};

function createService(logger = NullLogger) {
  return {
    doSomething() {
      logger.log("실행됨");
    }
  };
}
```

## 언제 사용하면 좋은가?

- `if (obj != null)` 같은 체크가 코드 전반에 많을 때
- 기본 동작을 "아무 일도 하지 않음"으로 안전하게 처리하고 싶을 때
- 예:
  - 게스트 사용자 처리 (로그인 안 된 경우)
  - 기본 로거(Logger) 제공 (log 생략 가능)
  - 옵셔널 서비스 의존성

## 정리

- Null Object 패턴은 **null 대신 안전한 객체를 반환**해 코드 간결화를 목적으로 사용 합니다.
- **분기문 제거 → 가독성, 유지보수성 ↑**
- 단점: "아무 일도 안 하는 객체"가 많아지면 디버깅 시 헷갈릴 수 있음.

## 관련 개념

- [[strategy]]: 기본 전략 객체 제공
- [[state]]: 기본 상태 객체 제공
- [[factory-method]]: Null Object 생성
- [[designpattern]]: 디자인 패턴 전체 개요
