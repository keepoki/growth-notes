#DesignPattern 

# Decorator

객체에 새로운 기능을 추가할 때,  상속을 쓰지 않고 **런타임에 동적으로 기능을 확장**할 수 있도록 하는 패턴입니다.
흔히 "포장지(Wrapper)"로 비유합니다.

## 기본 구현

```js
// 기본 컴포넌트
class Coffee {
  cost() {
    return 5;
  }
  description() {
    return "기본 커피";
  }
}

// 데코레이터 기본 구조
class CoffeeDecorator extends Coffee {
  constructor(coffee) {
    super();
    this.coffee = coffee;
  }
  cost() {
    return this.coffee.cost();
  }
  description() {
    return this.coffee.description();
  }
}

// 구체적인 데코레이터
class MilkDecorator extends CoffeeDecorator {
  cost() {
    return super.cost() + 2;
  }
  description() {
    return super.description() + ", 우유 추가";
  }
}

class SugarDecorator extends CoffeeDecorator {
  cost() {
    return super.cost() + 1;
  }
  description() {
    return super.description() + ", 설탕 추가";
  }
}

// 사용
let coffee = new Coffee();
coffee = new MilkDecorator(coffee);
coffee = new SugarDecorator(coffee);

console.log(coffee.description()); // "기본 커피, 우유 추가, 설탕 추가"
console.log(coffee.cost());        // 8
```

## 실제 활용 코드 예시

### Express 미들웨어 (데코레이터 개념과 유사)
```js
function baseHandler(req) {
  return `Hello, ${req.user || "Guest"}`;
}

function authDecorator(handler) {
  return (req) => {
    if (!req.user) return "인증 필요!";
    return handler(req);
  };
}

function logDecorator(handler) {
  return (req) => {
    console.log(`[LOG] ${new Date().toISOString()}`);
    return handler(req);
  };
}

// 사용
let handler = baseHandler;
handler = authDecorator(handler);
handler = logDecorator(handler);

console.log(handler({ user: "Alice" })); // 인증 성공 → 로그 찍고 응답
console.log(handler({}));                // "인증 필요!"
```

### React HOC (Higher-Order Component)
```js
// 기본 컴포넌트
function Button(props) {
  return `<button>${props.label}</button>`;
}

// 데코레이터 역할을 하는 HOC
function withLogging(component) {
  return (props) => {
    console.log(`Rendering with props:`, props);
    return component(props);
  };
}

function withBorder(component) {
  return (props) => {
    const html = component(props);
    return `<div style="border:1px solid black">${html}</div>`;
  };
}

// 사용
let EnhancedButton = withLogging(Button);
EnhancedButton = withBorder(EnhancedButton);

console.log(EnhancedButton({ label: "Click Me" }));
```

## 언제 사용하면 좋은가?

- **객체 기능을 동적으로 추가/삭제해야 할 때**  
    → 예: 커피 주문 옵션, UI 컴포넌트 스타일, 요청 처리 파이프라인
- **상속보다 조합(Composition)을 선호할 때**  
    → 상속하면 클래스 폭발 문제가 생길 수 있음 (`MilkCoffee`, `SugarCoffee`, `MilkSugarCoffee` 등)
- **여러 데코레이터를 조합해서 유연한 기능 확장이 필요할 때**

## 정리

- `Decorator`는 객체를 감싸면서 기능을 확장하는 패턴이에요.
- `상속` 대신 `조합`을 통해 유연성을 확보합니다.
- 실무에서는 **미들웨어, HOC, 파이프라인 처리** 같은 곳에서 많이 쓰입니다.