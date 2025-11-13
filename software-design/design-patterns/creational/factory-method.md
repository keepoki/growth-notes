---
tags:
  - DesignPattern
  - Creational
title: Factory Method Pattern
---

객체 생성 로직을 별도의 **팩토리(Factory) 메서드**로 캡슐화해서, 클라이언트 코드에서 `new` 키워드를 직접 사용하지 않고 객체를 생성할 수 있도록 하는 패턴입니다.

## 기본 구현

```js
class Product {
  constructor(name) {
    this.name = name;
  }
}

class ProductFactory {
  createProduct(type) {
    switch (type) {
      case "A":
        return new Product("Type A Product");
      case "B":
        return new Product("Type B Product");
      default:
        throw new Error("Unknown product type");
    }
  }
}

// 사용
const factory = new ProductFactory();
const p1 = factory.createProduct("A");
const p2 = factory.createProduct("B");

console.log(p1.name); // "Type A Product"
console.log(p2.name); // "Type B Product"
```

## 실제 활용 코드 예시

### 버튼 UI 생성(React 같은 환경에서 활용 가능)

```js
class Button {
  render() {
    return "<button>Default Button</button>";
  }
}

class PrimaryButton extends Button {
  render() {
    return "<button class='btn btn-primary'>Primary</button>";
  }
}

class DangerButton extends Button {
  render() {
    return "<button class='btn btn-danger'>Danger</button>";
  }
}

class ButtonFactory {
  createButton(type) {
    switch (type) {
      case "primary":
        return new PrimaryButton();
      case "danger":
        return new DangerButton();
      default:
        return new Button();
    }
  }
}

// 사용
const factory = new ButtonFactory();
console.log(factory.createButton("primary").render());
console.log(factory.createButton("danger").render());
```

### DB 커넥션 객체 생성

```js
class MySQLConnection {
  connect() {
    return "MySQL connected!";
  }
}

class MongoDBConnection {
  connect() {
    return "MongoDB connected!";
  }
}

class DBFactory {
  createConnection(type) {
    if (type === "mysql") return new MySQLConnection();
    if (type === "mongo") return new MongoDBConnection();
    throw new Error("Unsupported DB type");
  }
}

// 사용
const dbFactory = new DBFactory();
const mysql = dbFactory.createConnection("mysql");
const mongo = dbFactory.createConnection("mongo");

console.log(mysql.connect()); // "MySQL connected!"
console.log(mongo.connect()); // "MongoDB connected!"
```

## 언제 사용하면 좋은가?

- **객체 생성 로직이 복잡하거나, 여러 종류의 객체가 있을 때**
  - DB 연결, UI 컴포넌트 생성, 메시지 파서 등
- **클라이언트 코드가 객체 생성 방법을 몰라도 되도록 하고 싶을 때**
  - `new` 대신 `factory.createSomething()` 같은 방식으로 추상화
- **OCP(개방-폐쇄 원칙)** 적용 → 새로운 타입 추가 시 팩토리 메서드만 확장하면 됨

## 정리

`Factory Method`는 **객체 생성을 한 곳에서 관리**할 수 있고, **유연하게 확장**할 수 있다는 장점이 있어요.  
다만, 클래스나 메서드가 늘어나서 구조가 복잡해질 수 있다는 단점도 있습니다.

## 관련 개념

- [[abstract-factory]]: 관련된 객체 제품군을 생성하는 패턴
- [[builder]]: 복잡한 객체를 단계적으로 생성하는 패턴
- [[prototype]]: 객체를 복제하여 생성하는 패턴
- [[singleton]]: 인스턴스를 하나만 생성하는 패턴
- [[designpattern]]: 디자인 패턴 전체 개요
