#Architecture

# Hexagonal

Hexagonal Architecture(육각형 아키텍처)는 **비즈니스 로직을 중심(도메인)에** 두고, **외부 요소(DB, UI, API 등)를 Port & Adapter**를 통해 연결하는 아키텍처 패턴입니다.

- **Domain(Core)**: 비즈니스 규칙과 로직
- **Ports**: 도메인과 외부를 연결하는 인터페이스 (예: Repository 인터페이스)
- **Adapters**: Port를 실제 구현 (예: MySQL, MongoDB, REST API, CLI 등)

## 핵심 원칙

도메인이 외부 기술에 의존하지 않고, 외부가 도메인에 맞춰지도록 설계

## 기본 구현

```js
// Domain (Business Logic)
class UserService {
  constructor(userRepository) {
    this.userRepository = userRepository; // Port
  }

  registerUser(name) {
    if (!name) throw new Error("Name is required");
    this.userRepository.save({ name });
  }

  listUsers() {
    return this.userRepository.findAll();
  }
}

// Port (Interface)
class UserRepositoryPort {
  save(user) { throw "Not implemented"; }
  findAll() { throw "Not implemented"; }
}

// Adapter (In-Memory DB)
class InMemoryUserRepository extends UserRepositoryPort {
  constructor() {
    super();
    this.users = [];
  }
  save(user) { this.users.push(user); }
  findAll() { return this.users; }
}

// 실행
const userRepo = new InMemoryUserRepository();
const userService = new UserService(userRepo);

userService.registerUser("Alice");
userService.registerUser("Bob");
console.log(userService.listUsers());
```

## 실제 활용 예시 (Express)

```js
// Domain (Service)
class UserService {
  constructor(userRepo) {
    this.userRepo = userRepo;
  }
  createUser(name) {
    this.userRepo.save({ name });
  }
  getUsers() {
    return this.userRepo.findAll();
  }
}

// Port (Interface)
class UserRepository {
  save(user) { throw "Not implemented"; }
  findAll() { throw "Not implemented"; }
}

// Adapter (Memory Repository)
class MemoryUserRepository extends UserRepository {
  constructor() { super(); this.users = []; }
  save(user) { this.users.push(user); }
  findAll() { return this.users; }
}

// Adapter (Express Controller)
const express = require("express");
const app = express();
app.use(express.json());

const userService = new UserService(new MemoryUserRepository());

app.post("/users", (req, res) => {
  userService.createUser(req.body.name);
  res.send("User created");
});

app.get("/users", (req, res) => {
  res.json(userService.getUsers());
});

app.listen(3000, () => console.log("Hexagonal app running on 3000"));
```

## 언제 사용하면 좋은가?

- 외부 기술(MySQL, MongoDB, REST API 등)이 자주 바뀔 수 있는 프로젝트
- 도메인 로직을 **외부 의존성에서 완전히 분리**하고 싶을 때
- 테스트에서 외부 환경(DB, API)을 모킹(mock)하고 싶을 때

## 정리

### 장점

- 도메인이 외부 기술에 독립적
- 테스트 용이 (Adapter만 교체하면 됨)
- 확장성 ↑ (다른 DB, API 추가 시 Adapter만 교체)

### 단점

- 설계와 구현이 다소 복잡
- 작은 프로젝트에서는 오버엔지니어링

### 사용 예시

- 마이크로서비스 구조
- 도메인 주도 설계(DDD) 기반 시스템
- 기술 스택 교체 가능성이 높은 애플리케이션
