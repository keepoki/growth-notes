---
tags:
  - DesignPattern
  - Behavioral
title: Iterator Pattern
---

컬렉션(배열, 리스트, 트리 등) 내부 구조를 노출하지 않고, 원소에 순차적으로 접근할 수 있게 해주는 패턴입니다.
자바스크립트는 `Iterator`와 `Iterable` 프로토콜을 기본적으로 지원하므로 활용도가 높습니다.

## 기본 구현

```js
class Iterator {
  constructor(items) {
    this.items = items;
    this.index = 0;
  }

  hasNext() {
    return this.index < this.items.length;
  }

  next() {
    return this.hasNext() ? this.items[this.index++] : null;
  }
}

// 사용
const iterator = new Iterator([1, 2, 3, 4]);
while (iterator.hasNext()) {
  console.log(iterator.next());
}
// 출력: 1 2 3 4
```

## 실제 활용 코드 예시

### 커스텀 Iterator 만들기

```js
class Range {
  constructor(start, end) {
    this.start = start;
    this.end = end;
  }

  [Symbol.iterator]() {
    let current = this.start;
    const end = this.end;

    return {
      next() {
        if (current <= end) {
          return { value: current++, done: false };
        }
        return { done: true };
      }
    };
  }
}

// 사용
for (const num of new Range(1, 5)) {
  console.log(num); 
}
// 출력: 1 2 3 4 5
```

### 트리 구조 순회

```js
class TreeNode {
  constructor(value, children = []) {
    this.value = value;
    this.children = children;
  }

  *[Symbol.iterator]() {
    yield this.value;
    for (const child of this.children) {
      yield* child;
    }
  }
}

// 사용
const tree = new TreeNode(1, [
  new TreeNode(2, [new TreeNode(4), new TreeNode(5)]),
  new TreeNode(3)
]);

for (const value of tree) {
  console.log(value);
}
// 출력: 1 2 4 5 3
```

## 언제 사용하면 좋은가?

- **컬렉션을 다양한 방식으로 순회하고 싶을 때**  
    → ex) 배열, 트리, 그래프 등
- **내부 구현을 감추고, 외부에서는 일관된 방식으로 접근하고 싶을 때**
- **여러 컬렉션 구조를 통일된 방식으로 처리해야 할 때**

## 정리

- Iterator는 "내부 구조를 몰라도 순회 가능"하게 해주는 패턴입니다.
- JS에서는 `for...of`, `Symbol.iterator`, 제너레이터(`function*`)로 자연스럽게 구현 가능합니다.
- React, Redux 같은 라이브러리 내부에서도 데이터 순회에 자주 쓰입니다.

## 관련 개념

- [[composite]]: 트리 구조를 순회할 때 사용
- [[visitor]]: 구조를 순회하면서 연산을 적용하는 패턴
- [[functional-programming]]: 함수형 프로그래밍의 반복 처리
- [[designpattern]]: 디자인 패턴 전체 개요
