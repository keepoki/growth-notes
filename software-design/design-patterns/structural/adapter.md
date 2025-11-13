---
tags:
  - DesignPattern
  - Structural
title: Adapter Pattern
---

서로 다른 인터페이스를 가진 클래스들이 함께 동작할 수 있도록 **중간에 변환기(Adapter)** 를 두는 패턴입니다.  
흔히 "어댑터 = 변환기(돼지코 콘센트)"로 비유합니다.

## 기본 구현

```js
// 기존 서비스 (호환 안 됨)
class OldAPI {
  specificRequest() {
    return "데이터 (Old API 형식)";
  }
}

// 새로운 인터페이스 (클라이언트가 원하는 방식)
class NewAPI {
  request() {
    return "데이터 (New API 형식)";
  }
}

// Adapter: OldAPI를 NewAPI처럼 사용할 수 있게 변환
class Adapter extends NewAPI {
  constructor(oldAPI) {
    super();
    this.oldAPI = oldAPI;
  }

  request() {
    return this.oldAPI.specificRequest();
  }
}

// 사용
const oldAPI = new OldAPI();
const adapted = new Adapter(oldAPI);

console.log(adapted.request()); // "데이터 (Old API 형식)"
```

## 실제 활용 코드 예시

### API 응답 변환

```js
// 기존 API 응답
class LegacyUserService {
  getUser() {
    return { fname: "John", lname: "Doe" }; // 옛날 형식
  }
}

// 새로운 코드에서 기대하는 형식
class ModernUserService {
  getUser() {
    return { firstName: "John", lastName: "Doe" };
  }
}

// Adapter
class UserServiceAdapter extends ModernUserService {
  constructor(legacyService) {
    super();
    this.legacyService = legacyService;
  }

  getUser() {
    const user = this.legacyService.getUser();
    return { firstName: user.fname, lastName: user.lname };
  }
}

// 사용
const legacy = new LegacyUserService();
const adapted = new UserServiceAdapter(legacy);

console.log(adapted.getUser()); // { firstName: 'John', lastName: 'Doe' }
```

### 외부 라이브러리 호환

```js
// 외부 라이브러리 (호환 안 되는 인터페이스)
class ThirdPartyPayment {
  makePayment(amountInCents) {
    return `결제 완료: ${amountInCents} cents`;
  }
}

// 내가 원하는 인터페이스
class PaymentProcessor {
  pay(amount) {}
}

// Adapter
class PaymentAdapter extends PaymentProcessor {
  constructor(thirdParty) {
    super();
    this.thirdParty = thirdParty;
  }

  pay(amount) {
    return this.thirdParty.makePayment(amount * 100); // 원 → 센트 변환
  }
}

// 사용
const thirdParty = new ThirdPartyPayment();
const payment = new PaymentAdapter(thirdParty);

console.log(payment.pay(50)); // "결제 완료: 5000 cents"
```

## 언제 사용하면 좋은가?

- **기존 코드(레거시)와 새로운 코드 간의 인터페이스가 맞지 않을 때**
- **외부 라이브러리나 API를 우리 시스템에 맞게 통합해야 할 때**
- **한쪽을 수정하기 어려운 경우(외부 SDK, 레거시 시스템 등)**

## 정리

Adapter는 "**돼지코 변환기**"처럼, 서로 맞지 않는 인터페이스를 중간에서 연결해주는 역할을 합니다.  
덕분에 기존 코드를 고치지 않고 재사용할 수 있다는 장점이 있습니다.

## 관련 개념

- [[facade]]: 복잡한 시스템을 단순화하는 패턴
- [[bridge]]: 추상화와 구현을 분리하는 패턴
- [[proxy]]: 객체 접근을 제어하는 패턴
- [[designpattern]]: 디자인 패턴 전체 개요
