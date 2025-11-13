---
tags:
  - DesignPattern
  - Creational
title: Builder Pattern
---

복잡한 객체를 생성할 때, **객체 생성 과정을 단계별로 나누어 캡슐화**하는 패턴입니다.
클라이언트는 객체 생성 과정을 몰라도, 빌더를 통해 유연하게 다양한 객체를 만들 수 있습니다.

## 기본 구현

```js
class Car {
  constructor() {
    this.engine = null;
    this.tires = null;
    this.color = null;
  }
}

class CarBuilder {
  constructor() {
    this.car = new Car();
  }

  setEngine(engine) {
    this.car.engine = engine;
    return this; // 체이닝 가능
  }

  setTires(tires) {
    this.car.tires = tires;
    return this;
  }

  setColor(color) {
    this.car.color = color;
    return this;
  }

  build() {
    return this.car;
  }
}

// 사용
const car = new CarBuilder()
  .setEngine("V8")
  .setTires("Michelin")
  .setColor("Red")
  .build();

console.log(car);
// Car { engine: 'V8', tires: 'Michelin', color: 'Red' }
```

## 실제 활용 코드 예시

### SQL Query Builder

```js
class QueryBuilder {
  constructor() {
    this.query = "";
  }

  select(columns) {
    this.query += `SELECT ${columns.join(", ")} `;
    return this;
  }

  from(table) {
    this.query += `FROM ${table} `;
    return this;
  }

  where(condition) {
    this.query += `WHERE ${condition} `;
    return this;
  }

  build() {
    return this.query.trim() + ";";
  }
}

// 사용
const query = new QueryBuilder()
  .select(["id", "name", "email"])
  .from("users")
  .where("id = 1")
  .build();

console.log(query); 
// SELECT id, name, email FROM users WHERE id = 1;
```

### UI Form Builder

```js
class FormBuilder {
  constructor() {
    this.form = [];
  }

  addTextInput(name, label) {
    this.form.push(`<label>${label}: <input type="text" name="${name}" /></label>`);
    return this;
  }

  addCheckbox(name, label) {
    this.form.push(`<label><input type="checkbox" name="${name}" /> ${label}</label>`);
    return this;
  }

  addSubmit(label) {
    this.form.push(`<button type="submit">${label}</button>`);
    return this;
  }

  build() {
    return `<form>${this.form.join("")}</form>`;
  }
}

// 사용
const formHTML = new FormBuilder()
  .addTextInput("username", "Username")
  .addTextInput("email", "Email")
  .addCheckbox("subscribe", "Subscribe to newsletter")
  .addSubmit("Register")
  .build();

console.log(formHTML);
```

## 언제 사용하면 좋은가?

- **객체 생성 과정이 복잡하고, 다양한 설정 옵션이 있을 때**  
    → 자동차, 컴퓨터, 주문, SQL 쿼리, UI 폼 등
- **같은 객체 구조지만 다른 조합을 만들고 싶을 때**  
    → `CarBuilder`로 엔진/타이어/색상 조합 변경
- **코드를 가독성 좋게 체이닝 방식으로 작성하고 싶을 때**

## 정리

- `Factory Method` / `Abstract Factory`는 **무엇을 만들지(타입 선택)에** 초점이고,
- `Builder`는 **어떻게 만들지(과정과 조립)에** 초점이 있어요.

## 관련 개념

- [[factory-method]]: 객체 타입 선택에 초점을 둔 패턴
- [[abstract-factory]]: 제품군 생성 패턴
- [[composite]]: Builder로 복잡한 트리 구조 생성 가능
- [[designpattern]]: 디자인 패턴 전체 개요
