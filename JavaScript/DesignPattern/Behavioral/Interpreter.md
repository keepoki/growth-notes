#DesignPattern 

# Interpreter

특정 도메인의 언어(DSL, Domain-Specific Language)를 정의하고, 해당 언어의 문장을 해석(interpret)하는 해석기를 만드는 패턴입니다.

→ "문법 규칙"을 클래스로 표현하고, **트리 구조로 해석**합니다.  
→ SQL 파서, 정규식 엔진, 수식 계산기 같은 곳에서 쓰입니다.

## 기본 구현

```js
// Abstract Expression
class Expression {
  interpret(context) {}
}

// Terminal Expression (숫자)
class NumberExpression extends Expression {
  constructor(number) {
    super();
    this.number = number;
  }
  interpret() {
    return this.number;
  }
}

// Non-Terminal Expression (덧셈)
class AddExpression extends Expression {
  constructor(left, right) {
    super();
    this.left = left;
    this.right = right;
  }
  interpret() {
    return this.left.interpret() + this.right.interpret();
  }
}

// Non-Terminal Expression (곱셈)
class MultiplyExpression extends Expression {
  constructor(left, right) {
    super();
    this.left = left;
    this.right = right;
  }
  interpret() {
    return this.left.interpret() * this.right.interpret();
  }
}

// 사용: (5 + 3) * 2
const expr = new MultiplyExpression(
  new AddExpression(new NumberExpression(5), new NumberExpression(3)),
  new NumberExpression(2)
);

console.log(expr.interpret()); // 16
```

## 실제 활용 코드 예시

### 간단한 Boolean 식 해석기

```js
class Context {
  constructor(variables) {
    this.variables = variables;
  }
  get(name) {
    return this.variables[name];
  }
}

class VariableExpression extends Expression {
  constructor(name) {
    super();
    this.name = name;
  }
  interpret(context) {
    return context.get(this.name);
  }
}

class AndExpression extends Expression {
  constructor(left, right) {
    super();
    this.left = left;
    this.right = right;
  }
  interpret(context) {
    return this.left.interpret(context) && this.right.interpret(context);
  }
}

class OrExpression extends Expression {
  constructor(left, right) {
    super();
    this.left = left;
    this.right = right;
  }
  interpret(context) {
    return this.left.interpret(context) || this.right.interpret(context);
  }
}

// 사용
const context = new Context({ x: true, y: false });

const expr2 = new OrExpression(
  new VariableExpression("x"),
  new VariableExpression("y")
);

console.log(expr2.interpret(context)); // true
```

### 미니 DSL (간단한 수학 파서 느낌)

```js
function interpret(expression) {
  // 아주 간단한 인터프리터 (eval 없이 구현)
  const tokens = expression.split(" ");
  let stack = [];

  tokens.forEach(token => {
    if (!isNaN(token)) {
      stack.push(Number(token));
    } else {
      const b = stack.pop();
      const a = stack.pop();
      if (token === "+") stack.push(a + b);
      if (token === "*") stack.push(a * b);
    }
  });

  return stack.pop();
}

console.log(interpret("5 3 + 2 *")); // (5 + 3) * 2 = 16

```


## 언제 사용하면 좋은가?

- **간단한 언어(DSL)를 해석해야 할 때**  
    → SQL, 정규식, 수학식 계산기
- **반복되는 해석 로직을 객체로 분리하고 싶을 때**
- **트리 구조로 문법을 정의해야 할 때**

## 정리

- Interpreter는 **언어(문법)와 해석기를 객체로 정의**하는 패턴입니다.
- 작은 DSL(도메인 전용 언어) 구현에 적합하지만,  복잡한 문법(프로그래밍 언어 전체 수준)에는 비효율적
  → 이럴 땐 파서/컴파일러 도구 사용하는 것이 좋습니다.
