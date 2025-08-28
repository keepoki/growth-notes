#Scope
# Closure(클로저)

클로저(Closure)는 자바스크립트의 핵심적이고 강력한 개념 중 하나로, "함수와 그 함수가 선언될 당시의 렉시컬(정적) 환경의 조합"을 의미합니다. 좀 더 쉽게 말하면, **함수가 자신이 생성될 때의 환경(스코프)을 기억해서, 함수가 나중에 실행될 때 그 환경에 접근할 수 있게 해주는 기능**입니다.

## 클로저의 핵심 원리

1. **함수가 중첩될 때 발생**: 클로저는 주로 함수 안에 또 다른 함수가 정의될 때 나타납니다.
2. **외부 함수의 렉시컬 환경 기억**: 내부 함수는 자신이 정의된 외부 함수의 렉시컬 환경(변수, 다른 함수 등)을 기억합니다.
3. **외부 함수 종료 후에도 접근 가능**: 외부 함수가 실행을 마치고 스택에서 제거되더라도, 내부 함수는 자신이 기억하는 외부 함수의 렉시컬 환경에 접근하여 해당 변수들을 사용할 수 있습니다.

### 예시를 통한 이해

가장 기본적인 클로저의 예시입니다.

```js
function makeCounter() {
  let count = 0; // 이 변수는 makeCounter의 렉시컬 환경에 속합니다.

  return function() { // 이 익명 함수가 클로저입니다.
    count = count + 1;
    return count;
  };
}

const counter1 = makeCounter(); // makeCounter를 호출하여 클로저를 생성합니다.
const counter2 = makeCounter(); // 또 다른 클로저를 생성합니다.

console.log(counter1()); // 1 (counter1이 기억하는 count는 0에서 시작)
console.log(counter1()); // 2 (counter1이 기억하는 count는 1에서 시작)

console.log(counter2()); // 1 (counter2는 자신만의 독립적인 count를 기억합니다)
console.log(counter2()); // 2
```

위 예시에서:

* `makeCounter` 함수는 `count`라는 지역 변수를 가지고 있습니다.
* `makeCounter`는 익명 함수(클로저)를 반환합니다. 이 익명 함수는 `makeCounter`의 렉시컬 환경 안에 정의되었으므로, `count` 변수를 기억합니다.
* `makeCounter` 함수가 실행을 마치면, 일반적으로 `count` 변수는 사라져야 할 것 같지만, `counter1`과 `counter2`가 각각 반환된 익명 함수(클로저)를 참조하고 있기 때문에 `count` 변수는 메모리에 계속 남아있게 됩니다.
* `counter1()`을 호출할 때마다 `counter1` 클로저가 기억하는 `count` 변수의 값이 증가하고, `counter2()`를 호출할 때마다 `counter2` 클로저가 기억하는 `count` 변수의 값이 증가합니다. 이들은 서로 독립적인 `count` 변수를 가집니다.

## 클로저의 주요 활용 사례

클로저는 자바스크립트 개발에서 매우 다양하고 유용하게 활용됩니다.

1. **정보 은닉 (Encapsulation) / 프라이빗 변수 구현**: 외부에 노출하고 싶지 않은 변수나 함수를 클로저를 통해 캡슐화할 수 있습니다. 위 `makeCounter` 예시가 대표적입니다.

2. **상태 유지**: 특정 값을 기억하고 유지해야 할 때 사용됩니다. 이벤트 핸들러나 콜백 함수에서 자주 사용됩니다.

3. **함수형 프로그래밍**: 부분 적용(Partial Application), 커링(Currying)과 같은 함수형 프로그래밍 기법을 구현하는 데 사용됩니다.

4. **모듈 패턴**: 여러 함수와 데이터를 묶어서 하나의 모듈처럼 사용할 때 클로저를 활용합니다.

5. **콜백 함수**: 비동기 작업(setTimeout, AJAX 등)의 콜백 함수가 외부 스코프의 변수에 접근해야 할 때 자연스럽게 클로저가 형성됩니다.

클로저는 처음에는 다소 복잡하게 느껴질 수 있지만, 자바스크립트의 강력한 유연성과 기능을 제공하는 핵심 메커니즘이므로, 반드시 이해해야 할 중요한 개념입니다.

## 실제로 사용되는 패턴 예시

클로저는 자바스크립트 개발에서 매우 흔하게 사용되는 패턴입니다. 아래에 실제로 자주 사용되는 클로저 예시 2가지를 설명해 드릴게요.

### 프라이빗 변수와 메서드  encapsulation (정보 은닉)

클로저는 클래스의 `private` 키워드처럼 외부에서 직접 접근할 수 없는 변수나 함수를 만드는 데 유용합니다. 이렇게 하면 객체의 내부 상태를 안전하게 보호할 수 있죠.

```js
// counter 함수는 클로저를 반환합니다.
function createCounter() {
  let count = 0; // 이 변수는 외부에서 직접 접근할 수 없습니다.

  return {
    increment: function() {
      count++;
      return count;
    },
    decrement: function() {
      count--;
      return count;
    },
    getCount: function() {
      return count;
    }
  };
}

const myCounter = createCounter();

console.log(myCounter.getCount());   // 출력: 0
console.log(myCounter.increment());  // 출력: 1
console.log(myCounter.increment());  // 출력: 2
console.log(myCounter.decrement());  // 출력: 1

// 직접적으로 count 변수에 접근하려고 하면 오류가 발생합니다.
// myCounter.count; // undefined
// myCounter.count = 100; // 아무런 효과가 없습니다.
```

`createCounter` 함수가 실행을 마치면 `count` 변수는 사라져야 하지만, `myCounter`가 반환된 객체의 메서드들을 참조하고 있기 때문에 `count` 변수는 메모리에 계속 남아있습니다. 이 메서드들을 통해서만 `count`에 접근하고 수정할 수 있어 **정보 은닉**이 가능해집니다.

### 이벤트 핸들러에 데이터 전달하기

클로저는 이벤트 핸들러가 특정 값이나 상태를 기억하게 할 때도 유용하게 쓰입니다. 특히 반복문을 통해 여러 DOM 요소에 이벤트 핸들러를 붙일 때 자주 사용됩니다.

```html
<button data-color="red">빨강</button>
<button data-color="green">초록</button>
<button data-color="blue">파랑</button>

<script>
  function makeColorChanger(color) {
    // 이 함수가 클로저입니다. 외부 함수(makeColorChanger)의 color 변수를 기억합니다.
    return function() {
      document.body.style.backgroundColor = color;
      console.log(`배경색을 ${color}로 변경했습니다.`);
    };
  }

  const buttons = document.querySelectorAll('button');

  buttons.forEach(button => {
    const color = button.getAttribute('data-color');
    // makeColorChanger 함수를 호출해서 각 버튼에 맞는 클로저를 만듭니다.
    button.addEventListener('click', makeColorChanger(color));
  });
</script>
```

위 예시에서 `makeColorChanger` 함수는 `color` 매개변수를 받아서 새로운 함수를 반환합니다. 이 반환된 함수는 `color` 값을 기억하는 클로저가 됩니다.

* `빨강` 버튼에 이벤트 리스너를 추가할 때는 `makeColorChanger('red')`가 호출되어 `color`가 `'red'`인 클로저가 생성됩니다.
* `초록` 버튼에 이벤트 리스너를 추가할 때는 `color`가 `'green'`인 또 다른 클로저가 생성됩니다.

덕분에 각 버튼을 클릭했을 때, 해당 버튼에 맞는 색상으로 배경이 바뀌게 됩니다. 클로저가 없었다면 반복문 바깥에서 `color` 변수를 관리해야 하는 복잡함이 생겼을 겁니다.

이 두 가지 예시는 클로저가 어떻게 데이터를 안전하게 보호하거나, 동적으로 변수를 기억하게 하는 데 사용되는지 보여줍니다.

---

## 클래스에서 private 변수와 클로저의 차이점

클래스에서 `private` 변수를 사용하는 것과 클로저를 사용하는 것은 유사한 목적(정보 은닉)을 가지지만, 작동 방식과 사용법에서 중요한 차이가 있습니다.

### 클래스 (Class)에서의 private 변수

클래스에서 `private` 변수는 `#` 접두사를 사용해 정의합니다. 이는 ES2022에서 정식으로 도입된 기능이며, **언어 차원에서** 변수 접근을 제한하는 문법적 방법입니다.

* **언어적 지원**: 자바스크립트 엔진이 `#`으로 시작하는 변수는 클래스 외부에서 접근할 수 없도록 강제합니다.
* **구조적**: 클래스 문법 내에서 변수와 메서드를 구조적으로 묶어 관리합니다. `this.#변수명` 형태로 클래스 내부에서만 접근할 수 있습니다.
* **컴파일 타임**: `#` 변수들은 코드 컴파일(파싱) 단계에서부터 외부 접근이 불가능하도록 처리됩니다.

```javascript
class Counter {
  #count = 0; // private 변수

  increment() {
    this.#count++;
    return this.#count;
  }

  getCount() {
    return this.#count;
  }
}

const myCounter = new Counter();
console.log(myCounter.increment()); // 1
// console.log(myCounter.#count); // SyntaxError: Private field '#count' must be declared in an enclosing class
```

### 클로저 (Closure)를 사용한 정보 은닉

클로저는 **함수의 렉시컬 스코프를 활용**하여 `private` 변수와 비슷한 효과를 냅니다. 클래스와 달리 별도의 특별한 문법이 필요하지 않습니다.

* **스코프 기반**: 클로저는 함수가 정의될 때의 스코프를 기억하는 원리를 이용합니다. 외부 함수가 종료되어도, 반환된 내부 함수(클로저)가 외부 함수의 지역 변수에 대한 참조를 유지합니다.
* **함수적**: 함수 호출을 통해 새로운 "인스턴스"를 생성하는 함수형 패턴에 가깝습니다.
* **런타임**: 클로저는 런타임에 스코프 체인이 형성되면서 동작합니다.

```js
function createCounter() {
  let count = 0; // 클로저에 의해 'private'처럼 작동하는 변수

  return {
    increment: function() {
      count++;
      return count;
    },
    getCount: function() {
      return count;
    }
  };
}

const myCounter = createCounter();
console.log(myCounter.increment()); // 1
console.log(myCounter.count); // undefined (외부에서 접근 불가능)
```

### 주요 차이점 요약

| 특징         | 클래스 `private` 변수            | 클로저를 사용한 `private` 변수           |
| :--------- | :-------------------------- | :------------------------------ |
| **작동 원리**  | 언어적 문법 (`#`)에 의한 강제 접근 제한   | 함수의 렉시컬 스코프를 활용한 접근 제한          |
| **코드 스타일** | 객체 지향 프로그래밍 (OOP) 방식        | 함수형 프로그래밍 방식                    |
| **장점**     | 명확하고 직관적인 문법, `this` 키워드 사용 | ES6 이전 버전에서도 사용 가능, 더 유연한 패턴 생성 |
| **단점**     | ES2022 이후 버전에서만 사용 가능       | 클래스보다 코드가 복잡해 보일 수 있음           |

클래스의 `private` 변수는 현대 자바스크립트에서 객체 지향적인 정보 은닉을 구현하는 **표준적인 방법**입니다. 반면, 클로저는 ES5 시절부터 사용되어 온 **함수 기반의 패턴**이며, 여전히 다양한 상황에서 유연하게 활용될 수 있는 강력한 도구입니다.

결론적으로, 클래스를 사용한다면 `private` 변수 문법을 쓰는 것이 더 명확하고 현대적인 방식입니다. 하지만 클로저의 원리를 이해하는 것은 자바스크립트의 깊은 부분을 이해하는 데 필수적입니다.
