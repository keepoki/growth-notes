#Context

# Context (컨텍스트)

자바스크립트에서 **컨텍스트**는 `this` 키워드가 가리키는 대상을 의미하며, 함수가 어떻게 호출되었는지에 따라 동적으로 결정됩니다. `this`가 바인딩되는 주요 컨텍스트는 다음과 같습니다.

## 1. 전역 컨텍스트 (Global Context)

전역 공간에서 함수를 호출하거나, 메서드가 아닌 일반 함수로 호출할 때의 컨텍스트입니다.

- **브라우저 환경**: `window` 객체를 가리킵니다.
- **Node.js 환경**: `global` 객체를 가리킵니다.
- **엄격 모드(Strict Mode)**: `undefined`를 가리킵니다.

```js
console.log(this); // 브라우저: Window 객체 출력

function showThis() {
  console.log(this);
}
showThis(); // 브라우저: Window 객체 출력 (전역 컨텍스트)
```

## 2. 메서드 컨텍스트 (Method Context)

함수가 객체의 속성으로 정의되어 메서드로 호출될 때의 컨텍스트입니다. 이 경우 `this`는 해당 메서드를 소유한 **객체**를 가리킵니다.

```js
const user = {
  name: 'John',
  greet: function() {
    console.log(this.name); // 'this'는 user 객체
  }
};

user.greet(); // 출력: John
```

## 3. 생성자 컨텍스트 (Constructor Context)

`new` 키워드를 사용하여 함수를 생성자(Constructor)로 호출할 때의 컨텍스트입니다. `this`는 새롭게 생성된 **인스턴스(객체)를** 가리킵니다.

```js
function Person(name) {
  this.name = name; // 'this'는 새로 만들어진 인스턴스
}

const person1 = new Person('Alice');
console.log(person1.name); // 출력: Alice
```

## 4. 명시적 컨텍스트 (Explicit Context)

`call()`, `apply()`, `bind()`와 같은 메서드를 사용해 `this`가 가리킬 객체를 **개발자가 직접 지정**하는 방식입니다.

```js
const car = {
  brand: 'Hyundai'
};

function showBrand() {
  console.log(this.brand);
}

// call() 메서드를 사용해 showBrand 함수의 'this'를 car 객체로 바인딩
showBrand.call(car); // 출력: Hyundai
```

## 5. 화살표 함수 컨텍스트 (Arrow Function Context)

화살표 함수는 일반 함수와 다르게 **자신만의 `this`를 가지지 않습니다.** 대신, 자신이 선언된 **렉시컬(정적) 스코프**의 `this`를 상속받습니다. 즉, 상위 스코프의 컨텍스트를 그대로 사용합니다.

```js
const person = {
  name: 'Bob',
  arrowGreet: () => {
    // 화살표 함수는 자신의 'this'가 없으므로,
    // 이 함수의 'this'는 상위 스코프(전역 스코프)를 가리킨다.
    console.log(`Hello, I'm ${this.name}`);
  }
};

person.arrowGreet(); // 출력: Hello, I'm undefined (브라우저 기준)
```

## 스코프와 컨텍스트의 차이점 요약

| 구분         | 스코프(Scope)                    | 컨텍스트(Context)                |
| ---------- | ----------------------------- | ---------------------------- |
| **핵심 개념**  | 변수의 접근 가능 범위                  | `this` 키워드가 가리키는 대상          |
| **관련 키워드** | 변수, 함수, `let`, `const`, `var` | `this`                       |
| **결정 시점**  | 함수를 선언하는 시점 (렉시컬)             | 함수를 호출하는 시점 (동적)             |
| **무엇을 해결** | 변수 이름 충돌, 변수 유효성              | 어떤 객체(Context)가 현재 함수를 실행했는지 |

## 실행 컨텍스트와 메모리

자바스크립트에서 **실행 컨텍스트(Execution Context)가** 만들어질 때, 두 단계로 나뉘어 메모리를 처리합니다:
1. **생성 단계(Creation Phase)**
    - 변수 선언(`var`)과 함수 선언이 메모리에 등록됩니다.
    - `var` 변수는 **undefined**로 초기화, `let`/`const`는 **초기화되지 않은 상태(TDZ)로** 남습니다.
    - 함수 선언은 전체 **함수 객체**가 메모리에 할당됩니다.
2. **실행 단계(Execution Phase)**
    - 실제 코드가 위에서 아래로 실행되면서 변수에 값이 할당됩니다.
    - 함수 호출 시 실행 컨텍스트가 만들어지고 다시 같은 과정이 반복됩니다.

### 예제로 보기

```javascript
console.log(a); // undefined (var는 생성 단계에서 undefined로 초기화)
var a = 10;

console.log(b); // ReferenceError (let은 TDZ)
let b = 20;

sayHello(); // "Hello!" (함수 선언문은 메모리에 함수 객체가 이미 있음)
function sayHello() {
  console.log("Hello!");
}
```

**설명:**
- `var a` → 생성 단계에서 `a`라는 공간이 메모리에 잡히고 값은 `undefined`
- `let b` → 메모리 공간은 생기지만 초기화 안 됨 → TDZ 발생
- `function sayHello()` → 생성 단계에서 **함수 객체**가 메모리에 할당 → 선언 전에 호출 가능

### 핵심 요약

- **호이스팅 = 생성 단계에서 변수와 함수 선언을 메모리에 먼저 올리는 과정**
- `var`는 `undefined`로 초기화되며, `let`/`const`는 초기화 전까지 TDZ
- **메모리 할당 시점**은 **실행 컨텍스트 생성 단계**에서 이루어짐

원하면 제가 그림으로 **호이스팅과 메모리 할당 과정을 한눈에 보는 다이어그램** 만들어서 보여드릴 수도 있어요.  
그림으로 보면 훨씬 직관적이에요. 만들까요?