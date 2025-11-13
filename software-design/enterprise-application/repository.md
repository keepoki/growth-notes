---
tags:
  - DesignPattern
  - EnterpriseApplication
  - DataAccess
title: Repository Pattern
---

- 도메인 로직(비즈니스 로직)과 데이터 접근(DB, API 호출 등)을 분리
- **Repository = 데이터 저장소 추상화 계층**
- 서비스 계층은 DB 세부 구현을 몰라도, Repository 인터페이스를 통해 데이터에 접근 가능

`Service → Repository → DB` 구조로 데이터 접근을 캡슐화합니다.

## 기본 구현

```js
// UserRepository.js
class UserRepository {
  constructor(db) {
    this.db = db; // 예: MongoDB, MySQL 등
  }

  async getById(id) {
    return this.db.find((u) => u.id === id);
  }

  async getAll() {
    return this.db;
  }

  async add(user) {
    this.db.push(user);
    return user;
  }

  async remove(id) {
    const index = this.db.findIndex((u) => u.id === id);
    if (index !== -1) this.db.splice(index, 1);
  }
}

// 사용
const db = [{ id: 1, name: "Alice" }];
const userRepo = new UserRepository(db);

(async () => {
  console.log(await userRepo.getAll()); // [ { id: 1, name: "Alice" } ]
  await userRepo.add({ id: 2, name: "Bob" });
  console.log(await userRepo.getById(2)); // { id: 2, name: "Bob" }
})();
```

## 실제 활용 예시 (Express.js + Sequelize ORM)

```js
// repository/UserRepository.js
class UserRepository {
  constructor(UserModel) {
    this.UserModel = UserModel;
  }

  async findById(id) {
    return this.UserModel.findByPk(id);
  }

  async findAll() {
    return this.UserModel.findAll();
  }

  async create(userData) {
    return this.UserModel.create(userData);
  }
}

// service/UserService.js
class UserService {
  constructor(userRepository) {
    this.userRepository = userRepository;
  }

  async registerUser(userData) {
    // 비즈니스 로직 (ex: 이메일 중복 확인)
    const existing = await this.userRepository.findAll();
    if (existing.some((u) => u.email === userData.email)) {
      throw new Error("이미 존재하는 이메일입니다.");
    }
    return this.userRepository.create(userData);
  }
}

// app.js (Express 라우터)
const express = require("express");
const { Sequelize, DataTypes } = require("sequelize");
const UserRepository = require("./repository/UserRepository");
const UserService = require("./service/UserService");

const app = express();
app.use(express.json());

const sequelize = new Sequelize("sqlite::memory:");
const User = sequelize.define("User", {
  name: DataTypes.STRING,
  email: DataTypes.STRING,
});

(async () => {
  await sequelize.sync();

  const userRepo = new UserRepository(User);
  const userService = new UserService(userRepo);

  app.post("/users", async (req, res) => {
    try {
      const user = await userService.registerUser(req.body);
      res.json(user);
    } catch (err) {
      res.status(400).json({ error: err.message });
    }
  });

  app.get("/users", async (req, res) => {
    const users = await userRepo.findAll();
    res.json(users);
  });

  app.listen(3000, () => console.log("🚀 Server started on 3000"));
})();
```

## 언제 사용하면 좋은가?

- 데이터 접근 코드와 비즈니스 로직을 분리하고 싶을 때
- DB를 교체할 수 있도록 **추상화** 필요할 때 (ex. MySQL → MongoDB)
- 테스트 시 **Mock Repository**로 대체 가능하게 하고 싶을 때

## 정리

- Repository = "데이터 접근 추상화 계층"
- 장점: **관심사 분리, 테스트 용이, DB 교체 유연성**
- 단점: 너무 단순하면 ORM과 중복, 너무 복잡하면 오버엔지니어링

## 관련 개념

- [[data-mapper]]: 도메인과 DB를 분리하는 유사한 패턴
- [[dao-(data-access-object)]]: 데이터 접근 추상화의 다른 접근법
- [[unit-of-work]]: Repository와 함께 사용되는 트랜잭션 패턴
- [[service-layer]]: Repository를 사용하는 상위 계층
- [[clean-architecture]]: Repository 패턴을 활용하는 아키텍처
- [[ddd-(domain-driven-design)]]: 애그리게잇 저장에 Repository 사용
