# JavaScript

좋은 질문이에요 👍

자바스크립트를 **"프로토타입 기반 언어(prototype-based language)"** 라고 부르는 이유는, 객체 지향 방식을 구현하는 방식이 **클래스(class)** 가 아니라 **프로토타입(prototype)** 을 중심으로 이루어져 있기 때문입니다.

---

## 1. 클래스 기반 언어와 차이

- **클래스 기반 언어 (Java, C#, C++ 등)**
  객체를 만들기 전에 설계도(클래스)를 정의하고, 그 클래스로부터 인스턴스를 생성합니다.

```java
class Person {
    constructor(name) { this.name = name; }
    sayHi() { console.log("Hi " + this.name); }
}
const p = new Person("Tom");
p.sayHi(); // Hi Tom
```

- **프로토타입 기반 언어 (JavaScript)**
  원래는 `class` 키워드가 없었고, 모든 객체는 다른 객체를 **프로토타입으로 상속**받습니다. 즉, 객체가 다른 객체를 "참조"해서 기능을 확장하는 방식이에요.

---

## 2. 자바스크립트의 프로토타입

모든 객체는 **[[software-design/design-patterns/creational/prototype]]**(내부 슬롯)을 가지고 있고, 이는 다른 객체를 가리킵니다.
이 체인을 **프로토타입 체인(prototype chain)** 이라고 부릅니다.

```javascript
function Person(name) {
  this.name = name;
}
Person.prototype.sayHi = function() {
  console.log("Hi " + this.name);
};

const p = new Person("Tom");
p.sayHi(); // Hi Tom

// p에는 sayHi가 없지만, p.__proto__ (즉, Person.prototype)에 있기 때문에 호출 가능
```

즉, **객체가 다른 객체를 자신의 원형(프로토타입)으로 삼아 동작하는 구조**라서 "프로토타입 기반 언어"라고 불립니다.

---

## 3. 왜 이렇게 설계했을까?

- 자바스크립트는 처음에 브라우저에서 간단한 스크립트 실행을 위해 설계되었기 때문에, 복잡한 클래스 체계 대신 **동적으로 객체를 확장**하는 방식이 더 적합했습니다.
- 그래서 자바스크립트의 상속은 "복사"가 아니라 **참조(위임)** 개념에 가까워요.

---

## 4. ES6 `class`는 사실?

ES6에서 `class` 문법이 추가되었지만, **문법적 설탕(syntactic sugar)** 일 뿐, 내부적으로는 여전히 **프로토타입 기반 상속**으로 동작합니다.

```javascript
class Person {
  constructor(name) {
    this.name = name;
  }
  sayHi() {
    console.log("Hi " + this.name);
  }
}

// 내부적으로는 Person.prototype에 메서드가 저장됨
```

---

자바스크립트는 객체가 다른 객체를 원형으로 삼아 상속하는 **프로토타입 기반 객체지향 언어**라서, "프로토타입 언어"라고 부르는 거예요.
