---
tags:
  - Architecture
title: Clean Architecture
---

Clean Architecture는 **로버트 C. 마틴(‘Uncle Bob’)** 이 제안한 소프트웨어 아키텍처 패턴으로,  
**비즈니스 규칙(엔터프라이즈 로직)을** 애플리케이션의 중심에 두고, **UI, DB, 프레임워크 같은 외부 의존성을 바깥쪽 계층**에 배치하는 구조입니다.

핵심 원칙:

- **의존성 규칙(Dependency Rule)**: 안쪽 계층은 바깥쪽 계층에 대해 몰라야 함.
- 안쪽 → 바깥쪽 방향으로 의존성 존재 X (즉, DB, UI는 언제든 교체 가능).

**계층 구조 예시 (안쪽에서 바깥쪽 순):**

1. **Entities**: 비즈니스 규칙, 핵심 모델
2. **Use Cases (Application Layer)**: 유스케이스, 서비스
3. **Interface Adapters**: 컨트롤러, 프레젠터, 리포지토리 구현
4. **Frameworks & Drivers**: DB, UI, Web, External APIs

## 기본 구현

```js
// Entities (Domain Model)
class User {
  constructor(name) {
    this.name = name;
  }
}

// Use Cases
class CreateUser {
  constructor(userRepo) {
    this.userRepo = userRepo;
  }
  execute(name) {
    const user = new User(name);
    this.userRepo.save(user);
    return user;
  }
}

class ListUsers {
  constructor(userRepo) {
    this.userRepo = userRepo;
  }
  execute() {
    return this.userRepo.findAll();
  }
}

// Interface (Port)
class UserRepository {
  save(user) { throw "Not implemented"; }
  findAll() { throw "Not implemented"; }
}

// Infrastructure (Adapter - InMemory)
class InMemoryUserRepository extends UserRepository {
  constructor() {
    super();
    this.users = [];
  }
  save(user) { this.users.push(user); }
  findAll() { return this.users; }
}

// Framework (Express.js)
const express = require("express");
const app = express();
app.use(express.json());

const userRepo = new InMemoryUserRepository();
const createUser = new CreateUser(userRepo);
const listUsers = new ListUsers(userRepo);

app.post("/users", (req, res) => {
  const user = createUser.execute(req.body.name);
  res.json(user);
});

app.get("/users", (req, res) => {
  res.json(listUsers.execute());
});

app.listen(3000, () => console.log("Clean Architecture running on 3000"));
```

## 실제 활용 예시

- **Node.js + Express**: 위와 같이 계층 분리 적용 가능
- **NestJS**: 모듈, 서비스, 리포지토리 구조가 클린 아키텍처와 유사
- **DDD(Domain Driven Design)** 와 결합해 대규모 서비스 설계에 활용

## 언제 사용하면 좋은가?

- 장기적으로 유지보수/확장이 중요한 대규모 시스템
- 외부 기술(DB, UI, 프레임워크) 교체 가능성이 있는 경우
- 테스트 주도 개발(TDD)을 하고 싶을 때

## 정리

### 장점

- 비즈니스 로직이 외부 기술에 독립적
- 테스트 용이 (DB 없이도 유스케이스 단위 테스트 가능)
- 유연하고 확장 가능한 구조

### 단점

- 초기 설계 복잡도 ↑
- 작은 프로젝트에는 오버엔지니어링

### 사용 예시

- 대규모 엔터프라이즈 시스템
- 마이크로서비스 및 도메인 주도 설계 기반 프로젝트
- 은행, 금융, 의료 시스템 같은 장기 유지보수 필수 서비스

## 관련 개념

- [[hexagonal]]: 클린 아키텍처와 유사한 Port & Adapter 구조
- [[ddd-(domain-driven-design)]]: 함께 사용하면 좋은 도메인 주도 설계
- [[layered]]: 계층형 아키텍처의 발전된 형태
- [[repository]]: 클린 아키텍처에서 데이터 접근 추상화에 사용되는 패턴
- [[dependency-injection]]: 의존성 규칙 구현에 필수적인 패턴
