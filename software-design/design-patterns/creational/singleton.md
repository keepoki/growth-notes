---
tags:
  - DesignPattern
  - Creational
title: Singleton Pattern
---

하나의 클래스에 대해 **인스턴스가 단 하나만 존재**하도록 보장하고, 어디서든 그 인스턴스에 접근할 수 있게 하는 패턴입니다.

## 기본 구현

```js
class Singleton {
  constructor() {
    if (Singleton.instance) {
      return Singleton.instance; // 이미 존재하면 같은 인스턴스 반환
    }
    this.value = Math.random(); // 테스트용 속성
    Singleton.instance = this;
  }
}

const s1 = new Singleton();
const s2 = new Singleton();

console.log(s1 === s2); // true
console.log(s1.value, s2.value); // 같은 값
```

## 실제 활용 코드 예시

### 설정(config) 관리

```js
class Config {
  constructor() {
    if (Config.instance) return Config.instance;

    this.settings = {
      apiBaseUrl: "https://api.example.com",
      theme: "dark",
    };

    Config.instance = this;
  }

  get(key) {
    return this.settings[key];
  }

  set(key, value) {
    this.settings[key] = value;
  }
}

// 사용
const config1 = new Config();
const config2 = new Config();

console.log(config1.get("theme")); // "dark"
config2.set("theme", "light");
console.log(config1.get("theme")); // "light" (동일 인스턴스)
```

### 로깅(logger) 시스템

```js
class Logger {
  constructor() {
    if (Logger.instance) return Logger.instance;
    this.logs = [];
    Logger.instance = this;
  }

  log(message) {
    const entry = `[${new Date().toISOString()}] ${message}`;
    this.logs.push(entry);
    console.log(entry);
  }

  getLogs() {
    return this.logs;
  }
}

const logger1 = new Logger();
const logger2 = new Logger();

logger1.log("첫 번째 로그");
logger2.log("두 번째 로그");

console.log(logger1.getLogs() === logger2.getLogs()); // true
```

### 언제 사용하면 좋은가?

- **전역 상태를 하나로 유지해야 할 때**
  - 예: 앱 설정(Config), 로거(Logger), 캐시(Cache), 데이터베이스 연결 풀(DB Connection Pool)
- **리소스를 공유하거나 중복 생성 비용이 큰 경우**
  - 데이터베이스 커넥션, API 토큰 관리, 스레드 풀 등

## 정리

`Singleton`은 간단해 보이지만, **전역 상태를 가지게 되어 테스트하기 어려워지고, 코드 의존성을 강하게 만들 수 있다는 단점**도 있어요. 따라서 꼭 필요한 경우(예: 설정, 공통 유틸)에서만 사용하는 게 좋아요.

## 관련 개념

- [[factory-method]]: 객체 생성을 관리하는 다른 패턴
- [[object-pool]]: 인스턴스 재사용 패턴 (Singleton과 유사)
- [[dependency-injection]]: Singleton의 대안으로 사용되는 패턴
- [[designpattern]]: 디자인 패턴 전체 개요
