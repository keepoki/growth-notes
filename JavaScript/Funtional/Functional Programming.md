#Functional 

# Functional Programming (함수형 프로그래밍)

함수(수학적 개념에 가까운)를 중심으로 구성된 프로그래밍 패러다임.
상태 변화와 부수 효과(side effects)를 피하고, 순수 함수(pure function)와 고차 함수(higher-order functions)를 강조합니다.

## 핵심 개념 (JavaScript 관점)

| 개념                     | 설명                                          |
| ---------------------- | ------------------------------------------- |
| **순수 함수**              | 같은 입력 → 항상 같은 출력, 외부 상태를 변경하지 않음            |
| **불변성(immutability)**  | 변수나 데이터 구조를 변경하지 않고 복사하여 새로운 값을 만듦          |
| **고차 함수**              | 함수를 인자로 받거나, 함수를 반환하는 함수                    |
| **선언형 코드**             | "무엇을 할 것인가"에 집중 (ex. `.map()`, `.filter()`) |
| **함수 조합(composition)** | 작은 함수들을 조합하여 더 큰 동작을 구성                     |
## 자바스크립트가 함수형에 적합한 이유

- 함수가 일급 객체 (first-class citizen) → 함수를 변수에 저장, 인자로 전달, 반환 가능
- 고차 함수 지원 (`map`, `filter`, `reduce`, `forEach` 등)
- 클로저, 화살표 함수, 비구조화 할당 등 현대 문법 지원

## 간단한 예제

문제: 숫자 배열에서 짝수만 제곱한 뒤 총합 구하기
### 명령형 스타일 (Imperative)

```js
const nums = [1, 2, 3, 4, 5];
let result = 0;

for (let i = 0; i < nums.length; i++) {
  if (nums[i] % 2 === 0) {
    result += nums[i] ** 2;
  }
}

console.log(result); // 20 (2^2 + 4^2)
```

### 함수형 스타일 (Functional)

```js
const nums = [1, 2, 3, 4, 5];

const result = nums
  .filter(n => n % 2 === 0)  // [2, 4]
  .map(n => n ** 2)          // [4, 16]
  .reduce((sum, n) => sum + n, 0); // 20

console.log(result);
```

- `filter`: 짝수만 추출
- `map`: 제곱
- `reduce`: 합산
- 데이터는 변경되지 않음 (불변성)
- 선언형 코드 → 가독성 좋음

## 고차 함수 예시

```js
function makeMultiplier(factor) {
  return function (x) {
    return x * factor;
  };
}

const double = makeMultiplier(2);
const triple = makeMultiplier(3);

console.log(double(5)); // 10
console.log(triple(5)); // 15
```

## 순수 함수 vs 비순수 함수

```js
// 순수 함수: 외부 상태에 의존하지 않고, 변경도 없음
function add(a, b) {
  return a + b;
}

// 비순수 함수: 외부 상태 변경
let count = 0;
function increase() {
  count++;
}
```

## 단점

|단점|설명|
|---|---|
|성능 이슈|깊은 복사, 중간 연산 많으면 느려질 수 있음|
|추상화 과다|작은 함수가 많아지면 초보자에겐 어려울 수 있음|
|디버깅|함수 체인이 길면 디버깅 어려움|

## 결론

자바스크립트는 함수형 프로그래밍을 **강력하게 지원**하는 언어입니다.

- 작고 재사용 가능한 함수로 코드를 분리하고
- 불변성과 순수함수를 지키면
    → **버그를 줄이고 유지보수성 높은 코드**를 만들 수 있습니다.
