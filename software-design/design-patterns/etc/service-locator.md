---
tags:
  - DesignPattern
  - DependencyManagement
title: Service Locator Pattern
---

의존성 주입(Dependency Injection)과 비슷하지만,  객체를 **중앙 레지스트리(Service Locator)에** 등록해두고 필요한 곳에서 찾아 쓰는 패턴입니다.

- 장점: 의존성 관리 간단, 전역 접근 가능
- 단점: 의존성이 숨겨져 테스트/유지보수 어려울 수 있음 (안티패턴 취급되기도 함)

## 기본 구현

```js
class ServiceLocator {
  static services = new Map();

  static register(name, service) {
    this.services.set(name, service);
  }

  static get(name) {
    return this.services.get(name);
  }
}

// 서비스 정의
class UserService {
  getUser(id) {
    return { id, name: "경준" };
  }
}

class ProductService {
  getProduct(id) {
    return { id, name: "노트북" };
  }
}

// 서비스 등록
ServiceLocator.register("userService", new UserService());
ServiceLocator.register("productService", new ProductService());

// 사용
const userService = ServiceLocator.get("userService");
console.log(userService.getUser(1));

const productService = ServiceLocator.get("productService");
console.log(productService.getProduct(101));
```

## 실제 활용 코드 예시

### Express 미들웨어/서비스 관리

```js
// services/logger.js
class Logger {
  log(msg) {
    console.log("[LOG]", msg);
  }
}
export default new Logger();

// service-locator.js
import Logger from "./services/logger.js";

class ServiceLocator {
  static services = {
    logger: Logger
  };

  static get(name) {
    return this.services[name];
  }
}

export default ServiceLocator;

// app.js
import express from "express";
import ServiceLocator from "./service-locator.js";

const app = express();
const logger = ServiceLocator.get("logger");

app.get("/", (req, res) => {
  logger.log("홈페이지 접근");
  res.send("Hello");
});
```

### 게임 개발 (전역 시스템 접근)

```js
class AudioService {
  play(sound) {
    console.log(`🔊 Playing ${sound}`);
  }
}

ServiceLocator.register("audio", new AudioService());

// 어디서든 사용
ServiceLocator.get("audio").play("explosion.mp3");
```

## 언제 사용하면 좋은가?

- 작은 프로젝트에서 의존성 주입 컨테이너(DI Container) 대신 간단히 의존성 관리할 때
- 전역적으로 공유해야 하는 서비스가 많을 때 (로그, 설정, 번역, 캐싱)
- 빠르게 프로토타입 만들 때
⚠️ 하지만 **대규모 프로젝트**에서는 **DI(Dependency Injection)를** 권장 (의존성이 명시적으로 드러나고 테스트하기 쉬움)

## 정리

- Service Locator = "서비스 중앙 전화번호부"
- 장점: 편리, 빠른 접근
- 단점: 의존성이 숨겨져 디버깅/테스트 어려움 → **남용 주의**

## 관련 개념

- [[dependency-injection]]: Service Locator의 대안 패턴
- [[singleton]]: 전역 인스턴스 관리 패턴
- [[factory-method]]: 객체 생성 패턴
- [[designpattern]]: 디자인 패턴 전체 개요
