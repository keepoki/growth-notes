---
tags:
  - DesignPattern
  - BusinessLogic
  - DDD
title: Specification Pattern
---

**Specification 패턴**은 비즈니스 규칙(조건식, 검증 로직 등)을 객체로 분리해 **캡슐화**하고, 이들을 조합(and/or/not)할 수 있게 하는 패턴입니다.

> “조건문(if-else) 지옥을 없애고,  
> 재사용 가능한 명세 객체로 관리하자!”

예를 들어, “활성화된 고객 중 구매 금액이 10만 원 이상인 사람” 이라는 조건을 `IsActiveCustomerSpec` + `HighValueCustomerSpec` 두 개의 명세를 조합해서 표현할 수 있습니다.

## 기본 구현

```js
class Specification {
  isSatisfiedBy(candidate) {
    throw new Error("You must implement isSatisfiedBy()");
  }

  and(other) {
    return new AndSpecification(this, other);
  }

  or(other) {
    return new OrSpecification(this, other);
  }

  not() {
    return new NotSpecification(this);
  }
}

class AndSpecification extends Specification {
  constructor(one, other) {
    super();
    this.one = one;
    this.other = other;
  }
  isSatisfiedBy(candidate) {
    return this.one.isSatisfiedBy(candidate) && this.other.isSatisfiedBy(candidate);
  }
}

class OrSpecification extends Specification {
  constructor(one, other) {
    super();
    this.one = one;
    this.other = other;
  }
  isSatisfiedBy(candidate) {
    return this.one.isSatisfiedBy(candidate) || this.other.isSatisfiedBy(candidate);
  }
}

class NotSpecification extends Specification {
  constructor(inner) {
    super();
    this.inner = inner;
  }
  isSatisfiedBy(candidate) {
    return !this.inner.isSatisfiedBy(candidate);
  }
}
```

## 실제 활용 예시 (비즈니스 로직 예시)

```js
// 구체 명세
class IsActiveCustomerSpec extends Specification {
  isSatisfiedBy(customer) {
    return customer.isActive;
  }
}

class HighValueCustomerSpec extends Specification {
  isSatisfiedBy(customer) {
    return customer.totalPurchase >= 100000;
  }
}

// 명세 조합
const spec = new IsActiveCustomerSpec().and(new HighValueCustomerSpec());

const customers = [
  { name: "철수", isActive: true, totalPurchase: 150000 },
  { name: "영희", isActive: false, totalPurchase: 200000 },
  { name: "민수", isActive: true, totalPurchase: 50000 },
];

const filtered = customers.filter((c) => spec.isSatisfiedBy(c));
console.log(filtered); // [{ name: "철수", ... }]
```

조건을 캡슐화했기 때문에 비즈니스 규칙이 변경되어도 `if`문이 아닌 객체 교체로 대응 가능.

## 언제 사용하면 좋은가?

- **비즈니스 규칙이 자주 바뀌는 도메인**  
    (예: 금융, 쇼핑몰, 정책 엔진 등)
- 복잡한 조건문(`if`, `&&`, `||`)을 단순하게 만들고 싶을 때
- 규칙을 **테스트 가능하고 조합 가능한 객체로 표현**하고 싶을 때
- **DDD(Domain-Driven Design)** 환경에서 “도메인 규칙”을 명확히 분리할 때

## 정리

### 장점

- 규칙 변경 시 코드 수정 최소화
- 객체 간 재사용 가능
- 조건 조합이 깔끔하고 가독성 향상

### 단점

- 코드 구조가 복잡해질 수 있음 (규칙이 단순할 때는 오히려 과함)
- 초기에 설계 비용이 큼

### 사용 예시

- 도메인 모델 검증 로직 (회원, 주문, 정책)
- 필터링, 검색 조건 빌더
- DDD 기반 애플리케이션

### 요약

| 항목     | 내용                           |
| ------ | ---------------------------- |
| 핵심 개념  | 비즈니스 규칙을 객체로 캡슐화하여 조합 가능하게 함 |
| 장점     | 조건 재사용, 조합 용이, 유지보수성 향상      |
| 단점     | 단순 로직에는 오버엔지니어링이 될 수 있음      |
| 대표 사용처 | DDD, 정책 검증, 검색 필터링 시스템       |

## 관련 개념

- [[ddd-(domain-driven-design)]]: Specification이 자주 사용되는 설계 방법론
- [[strategy]]: 규칙을 전략 객체로 표현
- [[composite]]: Specification을 조합하는 패턴
- [[repository]]: Specification으로 쿼리 조건 전달
- [[designpattern]]: 디자인 패턴 전체 개요
