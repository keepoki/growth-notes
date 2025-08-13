#Functional

# loadsh/fp

유틸 함수 라이브러리 `loadsh`, 그중에서도 함수형 스타일을 강화한 `loadsh/fp`를 소개합니다.

설치: `npm install lodash`

## `lodash` vs `lodash/fp` 차이

| 항목                    | `lodash` (일반)       | `lodash/fp` (Functional Programming) |
| --------------------- | ------------------- | ------------------------------------ |
| **데이터 우선(data-last)** | ❌ 아니고, 데이터가 첫 번째 인자 | ✅ 맞음 (함수 커리 활용 가능)                   |
| **변경 가능성**            | 원본을 수정하는 함수도 있음     | 대부분 불변성 보장                           |
| **커리 지원**             | 기본적으로 커리 미지원        | ✅ 기본적으로 커리 지원                        |

## `lodash` vs `lodash/fp` 비교

목표: 숫자 배열에서 짝수만 제곱하고 합산

### 일반 `lodash`
```js
import _ from 'lodash';

const nums = [1, 2, 3, 4, 5];

const result = _.sum(_.map(_.filter(nums, n => n % 2 === 0), n => n ** 2));
console.log(result); // 20
```
- 데이터가 **앞에 위치**
- **중첩 호출**이 많아 **가독성 저하**

### `lodash/fp` 버전
```js
import fp from 'lodash/fp';

const nums = [1, 2, 3, 4, 5];

const process = fp.flow(
  fp.filter(n => n % 2 === 0),
  fp.map(n => n ** 2),
  fp.sum
);

const result = process(nums);
console.log(result); // 20
```
- `flow()`로 **함수 조합**
- 데이터는 마지막에 넘김 (**data-last** 스타일)
- 함수형 프로그래밍에 **더 친숙한 구조**

## 주요 함수형 유틸 목록 (`lodash/fp`)

| 함수          | 설명                      |
| ----------- | ----------------------- |
| `map`       | 배열/객체의 각 요소에 함수 적용      |
| `filter`    | 조건에 맞는 요소만 필터링          |
| `reduce`    | 누산기 연산                  |
| `flow`      | 함수들을 왼쪽 → 오른쪽으로 조합      |
| `compose`   | 함수들을 오른쪽 → 왼쪽으로 조합      |
| `curry`     | 함수를 커리화                 |
| `get`       | 객체에서 특정 경로의 값 가져오기 (불변) |
| `set`       | 객체에 경로에 값 설정 (불변)       |
| `cloneDeep` | 깊은 복사 (원본 보호)           |

### `fp.curry` 예제
```js
import fp from 'lodash/fp';

const add = (a, b) => a + b;
const curriedAdd = fp.curry(add);

console.log(curriedAdd(2)(3)); // 5
```
- `curry`를 사용하면 함수를 부분 적용(partial application)하여 더 유연하게 조합 가능

### 객체 다루기 예제
```js
import fp from 'lodash/fp';

const user = {
  name: 'Jane',
  address: {
    city: 'Seoul',
    zip: '12345'
  }
};

// city 값 가져오기
const city = fp.get('address.city', user);
console.log(city); // "Seoul"

// zip 코드 변경 (불변하게)
const updated = fp.set('address.zip', '99999', user);
console.log(updated.address.zip); // "99999"
console.log(user.address.zip);    // "12345" (원본 불변)
```

## 정리: `lodash/fp`는 이런 상황에서 특히 유용
- 함수형 스타일로 데이터를 처리하고 싶을 때
- 객체/배열을 **불변하게** 다루고 싶을 때
- 함수 조합과 커리를 활용해 **가독성 좋은 코드**를 만들고 싶을 때
