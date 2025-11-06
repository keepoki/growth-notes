# DesignPattern

# Strategy

알고리즘(행동)을 캡슐화해서 **런타임에 교체 가능**하게 만드는 패턴입니다.
→ `if/else`로 알고리즘 분기하는 대신, 전략 객체를 바꿔 끼우는 방식으로 유연하게 동작합니다.

## 기본 구현

```js
// Strategy 인터페이스
class Strategy {
  execute(a, b) {}
}

// 구체적인 전략들
class AddStrategy extends Strategy {
  execute(a, b) {
    return a + b;
  }
}

class SubtractStrategy extends Strategy {
  execute(a, b) {
    return a - b;
  }
}

class MultiplyStrategy extends Strategy {
  execute(a, b) {
    return a * b;
  }
}

// Context
class Calculator {
  setStrategy(strategy) {
    this.strategy = strategy;
  }

  calculate(a, b) {
    return this.strategy.execute(a, b);
  }
}

// 사용
const calculator = new Calculator();

calculator.setStrategy(new AddStrategy());
console.log(calculator.calculate(5, 3)); // 8

calculator.setStrategy(new MultiplyStrategy());
console.log(calculator.calculate(5, 3)); // 15
```

## 실제 활용 코드 예시

### 결재 시스템

```js
class CreditCardPayment {
  pay(amount) {
    console.log(`💳 신용카드로 ${amount}원 결제`);
  }
}

class PayPalPayment {
  pay(amount) {
    console.log(`📧 PayPal로 ${amount}원 결제`);
  }
}

class CryptoPayment {
  pay(amount) {
    console.log(`🪙 암호화폐로 ${amount}원 결제`);
  }
}

class PaymentContext {
  setStrategy(strategy) {
    this.strategy = strategy;
  }

  checkout(amount) {
    this.strategy.pay(amount);
  }
}

// 사용
const payment = new PaymentContext();

payment.setStrategy(new CreditCardPayment());
payment.checkout(10000);

payment.setStrategy(new CryptoPayment());
payment.checkout(25000);
```

## 정렬 알고리즘 교체

```js
class BubbleSort {
  sort(data) {
    console.log("BubbleSort 실행");
    return [...data].sort((a, b) => a - b); // 단순화
  }
}

class QuickSort {
  sort(data) {
    console.log("QuickSort 실행");
    return [...data].sort((a, b) => a - b); // 단순화
  }
}

class SortContext {
  setStrategy(strategy) {
    this.strategy = strategy;
  }

  sort(data) {
    return this.strategy.sort(data);
  }
}

// 사용
const sorter = new SortContext();

sorter.setStrategy(new BubbleSort());
console.log(sorter.sort([5, 3, 1, 4, 2]));

sorter.setStrategy(new QuickSort());
console.log(sorter.sort([5, 3, 1, 4, 2]));
```

## 언제 사용하면 좋은가?

- 알고리즘(비즈니스 로직)을 **상황에 따라 교체**해야 할 때
- `if/else` 또는 `switch`로 알고리즘 분기를 많이 하는 경우
- 결제 방식, 정렬 알고리즘, 경로 탐색 알고리즘 등 **여러 방법이 존재할 때**

## 정리

- Strategy는 **"알고리즘을 바꿔 끼우는"** 패턴입니다.
- Context는 알고리즘을 실행하되, 어떤 전략을 쓸지는 외부에서 주입.
- 장점: 유연성 ↑ / 단점: 클래스 개수 ↑
