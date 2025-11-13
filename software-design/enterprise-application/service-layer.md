---
tags:
  - DesignPattern
  - EnterpriseApplication
  - BusinessLogic
title: Service Layer Pattern
---

- **비즈니스 로직**을 한 곳(서비스 계층)에 모아두고, 프레젠테이션(UI)이나 인프라(DB, API)와 분리하는 패턴
- 컨트롤러 → 서비스 → DAO/Repository 구조로 나누어 **관심사 분리(Separation of Concerns)를** 달성

Controller는 요청을 받고, Service는 비즈니스 로직만 담당, Repository는 DB만 다룸

## 기본 구현

```js
// Repository (DB 접근 전담)
class UserRepository {
  constructor() {
    this.db = [];
  }

  save(user) {
    this.db.push(user);
    return user;
  }

  findById(id) {
    return this.db.find(u => u.id === id);
  }
}

// Service Layer (비즈니스 로직 담당)
class UserService {
  constructor(userRepository) {
    this.userRepository = userRepository;
  }

  registerUser(id, name) {
    const user = { id, name };
    return this.userRepository.save(user);
  }

  getUserProfile(id) {
    const user = this.userRepository.findById(id);
    if (!user) throw new Error("사용자를 찾을 수 없음");
    return { ...user, profile: `안녕하세요, ${user.name}님!` };
  }
}

// Controller (요청/응답 처리)
class UserController {
  constructor(userService) {
    this.userService = userService;
  }

  register(req) {
    return this.userService.registerUser(req.id, req.name);
  }

  profile(req) {
    return this.userService.getUserProfile(req.id);
  }
}

// 사용 예시
const repo = new UserRepository();
const service = new UserService(repo);
const controller = new UserController(service);

console.log(controller.register({ id: 1, name: "Alice" }));
console.log(controller.profile({ id: 1 }));
```

## 실제 활용 예시 (Express.js + Service Layer)

```js
// user.repository.js
const users = [];
export class UserRepository {
  save(user) {
    users.push(user);
    return user;
  }
  findById(id) {
    return users.find(u => u.id === id);
  }
}

// user.service.js
import { UserRepository } from "./user.repository.js";

export class UserService {
  constructor() {
    this.repo = new UserRepository();
  }
  registerUser(id, name) {
    return this.repo.save({ id, name });
  }
  getUserProfile(id) {
    const user = this.repo.findById(id);
    if (!user) throw new Error("User not found");
    return { ...user, profile: `Hello, ${user.name}!` };
  }
}

// user.controller.js
import express from "express";
import { UserService } from "./user.service.js";

const router = express.Router();
const userService = new UserService();

router.post("/register", (req, res) => {
  const { id, name } = req.body;
  res.json(userService.registerUser(id, name));
});

router.get("/:id", (req, res) => {
  try {
    res.json(userService.getUserProfile(Number(req.params.id)));
  } catch (err) {
    res.status(404).json({ error: err.message });
  }
});

export default router;
```

## 언제 사용하면 좋은가?

- **대규모 애플리케이션**에서 UI, 비즈니스 로직, DB 로직을 확실히 분리하고 싶을 때
- 여러 UI(웹, 모바일 앱 등)에서 같은 비즈니스 로직을 재사용해야 할 때
- 테스트 코드에서 비즈니스 로직만 독립적으로 검증하고 싶을 때

## 비교

- **Active Record**: 데이터와 로직이 한 클래스에
- **DAO / Repository**: DB 로직 분리
- **Service Layer**: DB를 포함한 **비즈니스 로직 전체를 독립 계층으로 관리**

## 정리

- Service Layer = "비즈니스 로직의 집합"
- 장점: 관심사 분리, 재사용성 ↑, 테스트 용이
- 단점: 코드 계층이 많아져서 작은 앱엔 다소 복잡

## 관련 개념

- [[repository]]: Service Layer가 사용하는 데이터 접근 계층
- [[dao-(data-access-object)]]: 데이터 접근 객체
- [[unit-of-work]]: 트랜잭션 관리 패턴
- [[layered]]: Service Layer가 속한 계층형 아키텍처
- [[ddd-(domain-driven-design)]]: 도메인 서비스 개념
- [[clean-architecture]]: Use Case 계층과 유사
