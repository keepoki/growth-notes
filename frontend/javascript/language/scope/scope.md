#Scope

# Scope (스코프)

## 스코프란?

자바스크립트의 스코프는 **변수의 유효 범위를 결정하는 규칙**이며, 크게 네 가지 유형으로 나눌 수 있습니다. 각각의 스코프는 변수 선언 방식과 함수의 위치에 따라 결정됩니다.

### 스코프의 종류와 예시

#### 1. 전역 스코프 (Global Scope)

코드의 **가장 바깥쪽에 선언된 변수**가 속하는 스코프입니다. 어디서든 접근할 수 있습니다.

```js
const globalVar = '전역 변수';

function myFunction() {
  console.log(globalVar); // ✅ 접근 가능
}
myFunction();
```

#### 2. 함수 스코프 (Function Scope)

**함수 내부에 선언된 변수**(`var`)가 속하는 스코프입니다. 해당 함수 안에서만 유효합니다.

```js
function myFunction() {
  var functionVar = '함수 변수';
  console.log(functionVar); // ✅ 접근 가능
}

myFunction();
// console.log(functionVar); // ❌ 에러: 함수 밖에서 접근 불가
```

#### 3. 블록 스코프 (Block Scope)

**`let`과 `const`로 선언된 변수**가 속하는 스코프입니다. `{}`(중괄호)로 둘러싸인 코드 블록 안에서만 유효합니다.

```js
if (true) {
  let blockVar = '블록 변수';
  console.log(blockVar); // ✅ 접근 가능
}
// console.log(blockVar); // ❌ 에러: 블록 밖에서 접근 불가
```

#### 4. 렉시컬 스코프 (Lexical Scope)

함수가 **어디서 선언되었는지**에 따라 스코프가 정해지는 방식입니다. 코드가 작성될 때 이미 결정됩니다. 자바스크립트의 기본 스코프 규칙입니다.

```js
const globalVar = '전역';

function outer() {
  const outerVar = '외부';

  function inner() {
    console.log(globalVar, outerVar); // ✅ 상위 스코프에 접근
  }
  inner();
}
outer();
```

#### 다이나믹 스코프 (Dynamic Scope)

자바스크립트에는 순수한 다이나믹 스코프가 없지만, **`this` 키워드**가 함수가 **어떻게 호출되었는지**에 따라 동적으로 바인딩되는 점에서 다이나믹 스코프와 유사하게 동작합니다.

```js
const user = { name: 'Alice' };

function sayName() {
  console.log(this.name);
}

sayName.call(user); // ✅ 호출 방식에 따라 'this'가 'user'로 바뀝니다. 출력: Alice
user.printName = sayName;
user.printName(); // ✅ 호출 방식에 따라 'this'가 'user'로 바뀝니다. 출력: Alice
```

---

### 스코프 종류 비교표

|구분|스코프|결정 시점|결정 기준|주로 사용하는 키워드|
|---|---|---|---|---|
|**정적 스코프**|**전역 스코프**|선언 시점|코드의 최상위|`var`, `let`, `const`|
||**함수 스코프**|선언 시점|함수 내부|`var`|
||**블록 스코프**|선언 시점|코드 블록 내부 (`{}`)|`let`, `const`|
||**렉시컬 스코프**|선언 시점|함수가 작성된 위치|-|
|**동적 스코프**|**다이나믹 스코프**|실행 시점|함수가 호출된 위치|`this` (자바스크립트의 유사 개념)|

### 렉시컬 스코프 vs 다이나믹 스코프

|구분|렉시컬 스코프|다이나믹 스코프|
|---|---|---|
|**스코프 결정 시점**|**코드 작성 시점** (정적)|**코드 실행 시점** (동적)|
|**결정 기준**|**함수가 어디서 선언되었는가**|**함수가 어디서 호출되었는가**|
|**자바스크립트**|기본적으로 채택|`this` 키워드가 유사하게 동작|
