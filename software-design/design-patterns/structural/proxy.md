---
tags:
  - DesignPattern
  - Structural
title: Proxy Pattern
---

실제 객체(Real Subject)에 접근하기 전에 **대리인(Proxy)** 을 두어,  접근 제어, 로깅, 캐싱, 지연 초기화 등을 처리하는 패턴입니다.

- **Proxy** = 대신해서 일 처리해주는 "대리" 객체
- **Real Subject** = 실제 핵심 기능을 가진 객체

## 기본 구현

```js
// 실제 객체
class RealImage {
  constructor(filename) {
    this.filename = filename;
    this.loadFromDisk();
  }

  loadFromDisk() {
    console.log(`Loading image: ${this.filename}`);
  }

  display() {
    console.log(`Displaying: ${this.filename}`);
  }
}

// 프록시
class ProxyImage {
  constructor(filename) {
    this.filename = filename;
    this.realImage = null;
  }

  display() {
    if (!this.realImage) {
      this.realImage = new RealImage(this.filename); // 지연 로딩
    }
    this.realImage.display();
  }
}

// 사용
const img = new ProxyImage("test.png");

// 아직 안 불러옴
console.log("Proxy created.");

// 필요할 때만 로딩
img.display();
// Loading image: test.png
// Displaying: test.png

img.display();
// Displaying: test.png (이미 캐싱됨 → 재로딩 없음)
```

## 실제 활용 코드 예시

### API 호출 캐싱

```js
class Api {
  async fetchData(id) {
    console.log(`Fetching from server: ${id}`);
    return { id, data: `data-${id}` };
  }
}

class ApiProxy {
  constructor(api) {
    this.api = api;
    this.cache = {};
  }

  async fetchData(id) {
    if (!this.cache[id]) {
      this.cache[id] = await this.api.fetchData(id);
    } else {
      console.log(`Returning cached data: ${id}`);
    }
    return this.cache[id];
  }
}

// 사용
(async () => {
  const api = new Api();
  const proxy = new ApiProxy(api);

  console.log(await proxy.fetchData(1)); // 서버 호출
  console.log(await proxy.fetchData(1)); // 캐시 사용
})();
```

### 접근 제어

```js
class Document {
  constructor(content) {
    this.content = content;
  }
  read() {
    return this.content;
  }
}

class SecureDocumentProxy {
  constructor(doc, userRole) {
    this.doc = doc;
    this.userRole = userRole;
  }
  read() {
    if (this.userRole !== "admin") {
      throw new Error("Access denied!");
    }
    return this.doc.read();
  }
}

// 사용
const secretDoc = new Document("Top Secret!");
const proxy = new SecureDocumentProxy(secretDoc, "guest");

try {
  console.log(proxy.read());
} catch (e) {
  console.log(e.message); // Access denied!
}
```

## 언제 사용하면 좋은가?

- **지연 초기화 (Lazy Loading)**  
    → 무거운 객체를 실제 필요할 때만 로딩
- **접근 제어 (Access Control)**  
    → 권한 체크, 인증/인가
- **로깅 & 모니터링**  
    → 메서드 호출 기록 남기기
- **캐싱 (Caching)**  
  → API 응답, DB 조회 결과 재사용

## 정리

- Proxy는 "**실제 객체에 접근하기 전 중간 단계를 추가**"하는 패턴.
- 실무에서 **API 캐싱, 인증, 로깅, 가상 프록시(지연 로딩)** 등에서 자주 활용됩니다.

## 관련 개념

- [[decorator]]: 객체에 기능을 추가하는 패턴
- [[adapter]]: 인터페이스를 변환하는 패턴
- [[facade]]: 복잡한 시스템을 단순화하는 패턴
- [[lazy-initialization]]: 지연 로딩 패턴
- [[designpattern]]: 디자인 패턴 전체 개요
