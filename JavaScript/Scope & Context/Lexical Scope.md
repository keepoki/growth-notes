#Scope

# Lexical Scope (렉시컬 스코프)

## 렉시컬 스코프란?

자바스크립트의 **렉시컬 스코프(Lexical Scope)** 는 함수를 어디서 선언했는지에 따라 결정되는 변수의 접근 규칙이에요. 즉, 함수를 호출하는 위치가 아니라 함수를 작성한 위치에 따라 스코프가 정해진다는 의미죠.

### 렉시컬 스코프의 원리

렉시컬 스코프는 **정적 스코프**라고도 불리며, 코드가 작성될 때 이미 정해져요. 함수는 자신만의 스코프를 가지고, 이 스코프 안에서 선언된 변수들에 접근할 수 있어요. 또한, 함수가 선언된 **상위 스코프**의 변수에도 접근이 가능하죠. 반대로, 하위 스코프의 변수에는 접근할 수 없어요. 이 구조는 중첩된 함수가 있을 때 특히 명확하게 드러나요.

### 실제 예시

다음 코드는 렉시컬 스코프의 개념을 잘 보여주는 예시예요.
```javascript

const globalVar = "전역 변수"; // 1️⃣ 전역 스코프

function outerFunction() {
  const outerVar = "외부 함수 변수"; // 2️⃣ outerFunction 스코프
  
  function innerFunction() {
    const innerVar = "내부 함수 변수"; // 3️⃣ innerFunction 스코프

    console.log(globalVar); // ✅ 접근 가능: 상위 스코프(전역)
    console.log(outerVar); // ✅ 접근 가능: 상위 스코프(outerFunction)
    console.log(innerVar); // ✅ 접근 가능: 자신의 스코프
  }
  
  innerFunction();
  // console.log(innerVar); // ❌ 에러 발생: 하위 스코프의 변수에는 접근 불가
}

outerFunction();
```

이 예시를 통해 알 수 있는 점은 다음과 같아요:
* `innerFunction`은 자신이 속한 스코프(`innerVar`)뿐만 아니라, 자신을 감싸고 있는 `outerFunction`의 스코프(`outerVar`)와 가장 바깥쪽인 **전역 스코프**(`globalVar`)의 변수에도 접근할 수 있어요.

* 반면, `outerFunction`은 `innerFunction` 내부에 선언된 `innerVar`에 접근할 수 없어요. 스코프 체인은 항상 안에서 바깥으로만 이어지기 때문이죠.

만약 `innerFunction`을 `outerFunction` 밖에서 호출한다면, `outerVar`에 접근할 수 없어요. 함수를 어디서 호출했는지는 중요하지 않고, **어디서 선언했는지**가 렉시컬 스코프를 결정하는 핵심이에요.