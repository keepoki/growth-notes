#Scope 

# Dynamic Scope (다이나믹 스코프)

다이나믹 스코프는 **함수가 어디서 호출되었는지**에 따라 변수의 유효 범위를 결정하는 방식입니다. 이는 함수를 어디서 선언했는지에 따라 스코프가 정해지는 렉시컬 스코프와는 대조적입니다.

## 다이나믹 스코프의 특징

다이나믹 스코프에서는 함수가 호출될 때마다 새로운 스코프 체인이 만들어집니다. 이 스코프 체인은 호출 스택을 따라 상위 호출자의 스코프까지 올라가면서 변수를 찾습니다.

- **렉시컬 스코프**: 코드를 작성하는 시점에 스코프가 정해집니다.
- **다이나믹 스코프**: 코드가 실행되는 시점에 스코프가 정해집니다.

자바스크립트는 기본적으로 렉시컬 스코프를 따르므로, 순수한 다이나믹 스코프를 구현하는 것은 불가능합니다. 하지만 특정 상황에서는 **`this` 키워드**를 통해 다이나믹 스코프와 유사한 동작을 볼 수 있습니다.

#### 예시 1:
```js
const user = {
  name: 'Alice',
  sayName: function() {
    console.log(this.name); // 👈 여기서 'this'는 호출 방식에 따라 달라집니다.
  }
};

const anotherUser = {
  name: 'Bob'
};

user.sayName(); // ✅ 출력: 'Alice'
// 'sayName' 함수가 'user' 객체의 메서드로 호출되었으므로, 'this'는 'user'를 가리킵니다.

const sayNameFn = user.sayName;

sayNameFn(); // ❌ 출력: undefined (엄격 모드가 아닐 경우) 또는 에러
// 일반 함수로 호출되었으므로, 'this'는 전역 객체(window 또는 global)를 가리킵니다.
// 전역 객체에는 'name' 속성이 없으므로 'undefined'가 출력됩니다.

sayNameFn.call(anotherUser); // ✅ 출력: 'Bob'
// 'call' 메서드를 사용해 'this'를 'anotherUser'로 명시적으로 바인딩했습니다.
// 'sayName' 함수는 'anotherUser'의 컨텍스트에서 호출된 것처럼 동작합니다.
```

이 예시에서 `user.sayName()`과 `sayNameFn()`의 결과가 다른 이유는 `this`의 값이 함수를 호출한 방법에 따라 동적으로 결정되기 때문입니다. `this`는 마치 다이나믹 스코프처럼 함수 호출 시의 환경에 의존합니다. `call`, `apply`, `bind`와 같은 메서드는 이러한 `this`의 동작을 명시적으로 제어할 수 있게 해줍니다.

#### 예시 2:

```js
// 1️⃣ 함수와 객체 선언
const carA = {
  name: '붕붕카',
  print: null,
};

const carB = {
  name: '꼬마차',
  print: null,
}

function sayName() {
  console.log(`My Car Name is ${this.name}.`);
}

// 2️⃣ 함수를 객체의 메서드로 할당
carA.print = sayName;
carB.print = sayName;

// 3️⃣ 메서드 호출
carA.print(); // ✅ 출력: My Car Name is 붕붕카.
carB.print(); // ✅ 출력: My Car Name is 꼬마차.
```

이처럼 `this` 키워드의 값이 함수를 호출하는 시점의 컨텍스트에 따라 동적으로 결정되는 방식이 **다이나믹 스코프**와 유사한 동작을 보여주는 대표적인 예시예요.

---
## 바인딩 메서드 call, apply, bind

`call`, `apply`, `bind`는 모두 **`this` 키워드의 값을 명시적으로 설정**하는 메서드입니다. 이들은 함수 호출 시 `this`가 가리키는 대상을 원하는 객체로 바꿀 수 있게 해줍니다.

### `call` 메서드

`call`은 함수를 즉시 호출하면서, `this`로 사용할 객체와 개별적인 **인자(arguments) 리스트**를 전달합니다.

**문법:** `function.call(thisArg, arg1, arg2, ...)`
**예시:**

```js
const user = { name: 'Alice' };

function introduce(greeting, city) {
  console.log(`${greeting}, I'm ${this.name} from ${city}.`);
}

introduce.call(user, 'Hello', 'New York');
// 출력: "Hello, I'm Alice from New York."
```

`introduce` 함수 내부의 `this`는 `user` 객체를 가리키고, `greeting`과 `city`는 각각 `'Hello'`와 `'New York'`이 됩니다.

### `apply` 메서드

`apply`는 `call`과 거의 같지만, 함수에 전달할 인자를 **배열(array)** 형태로 전달한다는 점이 다릅니다.

**문법:** `function.apply(thisArg, [argsArray])`
**예시:**

```js
const user = { name: 'Bob' };

function introduce(greeting, city) {
  console.log(`${greeting}, I'm ${this.name} from ${city}.`);
}

introduce.apply(user, ['Hi', 'London']);
// 출력: "Hi, I'm Bob from London."
```

`call`과 동일한 결과를 얻지만, 인자를 배열로 묶어서 전달합니다. 인자의 개수가 가변적이거나, 이미 배열로 인자를 가지고 있을 때 유용합니다.

### `bind` 메서드

`bind`는 함수를 즉시 호출하지 않고, `this` 값이 영구적으로 고정된 **새로운 함수**를 반환합니다. 이 새로운 함수는 나중에 호출할 수 있습니다.

**문법:** `const newFunction = function.bind(thisArg, arg1, arg2, ...)`
**예시:**

```js
const user = { name: 'Charlie' };

function introduce(greeting) {
  console.log(`${greeting}, I'm ${this.name}.`);
}

const boundIntroduce = introduce.bind(user, 'Hey');
// 함수를 즉시 실행하지 않고, 'user'를 'this'로 고정시킨 새로운 함수를 만듭니다.

boundIntroduce();
// 출력: "Hey, I'm Charlie."
```

`boundIntroduce`를 호출하면 `this`는 항상 `user` 객체를 가리킵니다. 이벤트 핸들러나 콜백 함수에서 `this`를 특정 객체로 고정해야 할 때 유용하게 쓰입니다.

### 요약
- **`call`**: 함수를 **즉시 호출**하며, 인자를 **하나씩** 전달합니다.
- **`apply`**: 함수를 **즉시 호출**하며, 인자를 **배열**로 전달합니다.
- **`bind`**: `this`가 고정된 **새로운 함수를 반환**하며, 나중에 호출할 수 있습니다.