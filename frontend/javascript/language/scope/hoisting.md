#Scope

# Hoisting (호이스팅)

자바스크립트에서 **호이스팅(Hoisting)이란**, **변수 선언과 함수 선언이 해당 스코프의 최상단으로 끌어올려진 것처럼 동작하는 현상**을 말합니다. **생성 단계에서 변수와 함수 선언을 메모리에 먼저 올리는 과정**입니다.

## 호이스팅의 종류

### 1. 변수 호이스팅

`var`로 선언된 변수는 **선언만** 위로 끌어올려지고, **할당은 끌어올려지지 않음**에 주의해야 합니다.

```javascript
console.log(a); // undefined
var a = 10;
console.log(a); // 10
```

내부적으로는 이렇게 동작한 것과 같음:

```javascript
var a;          // 선언만 호이스팅
console.log(a); // undefined
a = 10;         // 할당
console.log(a); // 10
```

> `let`, `const`는 **호이스팅은 되지만** "일시적 사각지대(TDZ, Temporal Dead Zone)" 때문에 선언 전에 접근 시 **ReferenceError**가 발생합니다.

```javascript
console.log(b); // ReferenceError
let b = 20;
```

### 2. 함수 호이스팅

**함수 선언문**(`function`)은 선언과 정의가 모두 호이스팅됩니다. 그래서 선언 전에 호출 가능해요.

```javascript
sayHello(); // "Hello!"

function sayHello() {
  console.log("Hello!");
}
```

내부적으로는 이렇게 된 것과 같음:

```javascript
function sayHello() {
  console.log("Hello!");
}
sayHello(); // "Hello!"
```

### 3. 함수 표현식의 경우

`함수 표현식(function expression)`은 변수에 할당하는 형태이므로, 변수 선언만 호이스팅되고 함수 정의는 호이스팅되지 않습니다.

```javascript
sayHi(); // TypeError: sayHi is not a function

var sayHi = function() {
  console.log("Hi!");
};
```

### 정리

- **`var` 변수 선언**: 선언만 호이스팅되고, 초기화는 호이스팅되지 않는다.
- **`let`, `const` 변수 선언**: 호이스팅은 되지만 TDZ 때문에 선언 전에 사용 불가.
- **함수 선언문**: 선언 + 정의 모두 호이스팅됨.
- **함수 표현식**: 변수 선언만 호이스팅됨, 함수 정의는 아님.

---

## `let`과 `const`에 TDZ를 도입한 이유

`let`과 `const`가 호이스팅 될 때 `TDZ(Temporal Dead Zone)`로 인해 초기화 전에 접근 할 수 없도록 설계된 이유는 다음과 같습니다.

1. **예측 가능성 및 버그 감소**: `var`의 경우 호이스팅 시 `undefined`로 초기화되기 때문에 변수 선언 전에 접근해도 에러가 발생하지 않고 `undefined`가 반환됩니다. 이는 때때로 개발자가 의도치 않은 동작을 유발하고 디버깅을 어렵게 만듭니다. `const`와 `let`은 TDZ를 통해 변수가 실제로 선언되고 값이 할당되기 전에는 접근 자체를 막음으로써 이러한 종류의 버그를 줄이고 코드의 예측 가능성을 높입니다.

2. **스코프 명확화**: `var`는 함수 스코프를 가지는 반면, `const`와 `let`은 블록 스코프를 가집니다. TDZ는 변수가 선언된 블록 내에서만 유효하며, 선언 라인 이전에는 접근할 수 없도록 함으로써 블록 스코프의 개념을 더욱 명확하게 합니다. 즉, 변수가 실제로 사용 가능한 시점은 선언 이후라는 것을 강제하여 스코프 혼란을 방지합니다.

3. **오류 조기 발견**: 초기화 전에 변수에 접근하려는 시도는 대부분 논리적인 오류입니다. TDZ는 이러한 오류를 런타임에 즉시 `ReferenceError`로 발생시켜 개발자가 문제를 조기에 인지하고 수정할 수 있도록 돕습니다.

4. **`const`의 불변성 보장**: `const`는 한번 할당된 값을 변경할 수 없도록 설계되었습니다. 만약 TDZ가 없다면, `const` 변수에 초기값 할당 전에 접근하여 `undefined`와 같은 임시 값을 사용할 수 있게 될 것이고, 이는 `const`의 불변성이라는 핵심 철학에 어긋날 수 있습니다. TDZ는 `const` 변수가 항상 정의된 값을 가지도록 강제합니다.

### 공식 문서 ECMA-262에서 14.3.1 let과 const 선언

다음은 ECMA-262에서 **14.3.1 Let and Const Declarations**에 해당하는 내용입니다.

`let`과 `const`로 선언된 변수들은 현재 코드가 실행되는 곳(실행 컨텍스트)의 `렉시컬 환경(LexicalEnvironment)`이라는 곳에 속하게 됩니다.

이 변수들은 **변수들을 포함하는 `환경 레코드(Environment Record)`가 만들어질 때 생성됩니다.** (이 부분이 바로 '호이스팅'과 관련이 있습니다. 선언문이 실행되기 전에 변수 공간이 미리 확보된다는 의미입니다.)

**하지만 이 변수들은 해당 변수의 `LexicalBinding`이 평가되기 전까지는 어떤 방식으로도 접근할 수 없습니다.** (이 부분이 바로 `TDZ(Temporal Dead Zone)`입니다. 변수는 만들어졌지만, 아직 사용할 준비가 안 된 상태인 것이죠.)

만약 `LexicalBinding`에 `초기값(Initializer)`이 함께 있다면, 이 변수에는 **`LexicalBinding`이 평가될 때** 그 초기값(`AssignmentExpression`의 결과)이 할당됩니다. 변수가 만들어질 때(호이스팅 시점)가 아닙니다.

만약 `let` 선언에 `초기값`이 없다면, 해당 변수는 `LexicalBinding`이 평가될 때 `undefined` 값이 할당됩니다.

**요약:**

1. **`let`/`const` 변수는 실행 컨텍스트의 `렉시컬 환경`에 속한다.** (즉, 블록 스코프를 따른다.)
2. **변수 공간은 `환경 레코드`가 생성될 때 미리 확보된다.** (이게 호이스팅입니다. 하지만 `var`처럼 `undefined`로 초기화되지는 않습니다.)
3. **`LexicalBinding`이 평가되기 전까지는 변수에 접근할 수 없다.** (이 기간이 `TDZ`이고, 접근 시 `ReferenceError`가 발생한다.)
4. **변수에 실제 값이 할당되는 시점은 `LexicalBinding`이 평가될 때이다.**
   - 초기값이 있으면 그 값으로 할당된다.
   - `let` 변수에 초기값이 없으면 `undefined`로 할당된다. (`const`는 반드시 초기값이 있어야 합니다.)

### LexicalBinding

`LexicalBinding`이라는 용어는 자바스크립트 "구문"의 일부분을 의미합니다. 쉽게 말해, **변수를 선언하고 선택적으로 값을 할당하는 부분 전체**를 `LexicalBinding`이라고 이해할 수 있습니다.

`LexicalBinding`은 ECMAScript 문법의 한 요소이며, 실제 자바스크립트 코드에서 `let` 또는 `const` 키워드 뒤에 오는 변수 선언 부분을 가리킵니다.

**예제 1: `let` 선언 (초기값 없음)**

```js
let x; // 이 전체가 하나의 LexicalBinding 입니다. Initializer가 없는 경우입니다.

console.log(x); // 'x'의 LexicalBinding이 평가된 후이므로 undefined가 출력됩니다.
```

여기서 `let x;` 이 구문 전체가 `LexicalBinding`입니다. 표준 문서에 따르면, 이 `LexicalBinding`이 평가될 때 `x`에 `undefined`가 할당됩니다.

**예제 2: `let` 선언 (초기값 있음)**

```js
let y = 10; // 이 전체가 하나의 LexicalBinding 입니다. Initializer가 있는 경우입니다.

console.log(y); // 'y'의 LexicalBinding이 평가된 후이므로 10이 출력됩니다.
```

여기서 `let y = 10;` 이 구문 전체가 `LexicalBinding`입니다. 이 `LexicalBinding`이 평가될 때 `y`에 `10`이 할당됩니다.

**예제 3: `const` 선언 (반드시 초기값 있음)**

```js
const z = 'hello'; // 이 전체가 하나의 LexicalBinding 입니다. Initializer가 필수입니다.

console.log(z); // 'z'의 LexicalBinding이 평가된 후이므로 'hello'가 출력됩니다.
```

여기서 `const z = 'hello';` 이 구문 전체가 `LexicalBinding`입니다. 이 `LexicalBinding`이 평가될 때 `z`에 `'hello'`가 할당됩니다.

#### TDZ와 `LexicalBinding`의 관계를 보여주는 예제

```js
console.log(a); // ReferenceError: Cannot access 'a' before initialization
// ^^^^^^^^^^^^
// 여기서 'a'는 이미 호이스팅되어 LexicalEnvironment에 "생성"되었지만,
// 아직 'let a = 10;' 이라는 LexicalBinding이 "평가"되기 전입니다.
// 이 평가 전까지의 구간이 바로 TDZ입니다.

let a = 10; // 여기가 LexicalBinding 입니다. 이 시점에서 'a'가 초기화되고 10이 할당됩니다.
console.log(a); // 10
```

이 예제에서 첫 번째 `console.log(a)`가 실행되는 시점에는 `a`라는 변수 공간은 존재하지만, `let a = 10;`이라는 `LexicalBinding`이 아직 실행되지 않아 `a`가 초기화되지 않은 상태입니다. 이 상태가 바로 `TDZ`에 있는 것이고, 그래서 `ReferenceError`가 발생합니다.

`LexicalBinding`은 자바스크립트 엔진이 코드를 파싱하고 실행하는 과정에서 변수를 처리하는 방식에 대한 내부적인 용어라고 이해하시면 됩니다. 개발자가 직접적으로 `LexicalBinding`을 조작하는 것은 아니지만, 이 개념을 통해 `let`과 `const`가 `var`와 다르게 동작하는 이유(특히 TDZ)를 더 깊이 이해할 수 있습니다.
