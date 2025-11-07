#Architecture

# Layered

계층형 아키텍처는 애플리케이션을 **기능별로 계층(layer)으로 나누어 설계**하는 패턴입니다.  
각 계층은 **자신보다 아래 계층에만 의존**하며, 위 계층과는 느슨하게 결합합니다.

**일반적인 계층 구조 예시**:

1. **Presentation Layer (UI)**: 사용자 인터페이스
2. **Application Layer (Service)**: 비즈니스 로직과 서비스
3. **Domain Layer (Business Model)**: 도메인 엔티티와 규칙

## 기본 구현

```js
// Infrastructure Layer
class UserRepository {
  constructor() {
    this.users = [];
  }
  addUser(user) {
    this.users.push(user);
  }
  getUsers() {
    return this.users;
  }
}

// Domain Layer
class User {
  constructor(name) {
    this.name = name;
  }
}

// Application Layer
class UserService {
  constructor(userRepo) {
    this.userRepo = userRepo;
  }
  registerUser(name) {
    const user = new User(name);
    this.userRepo.addUser(user);
  }
  listUsers() {
    return this.userRepo.getUsers();
  }
}

// Presentation Layer
class UserController {
  constructor(userService) {
    this.userService = userService;
  }
  addUser(name) {
    this.userService.registerUser(name);
  }
  showUsers() {
    console.log("Users:", this.userService.listUsers().map(u => u.name));
  }
}

// 실행
const userRepo = new UserRepository();
const userService = new UserService(userRepo);
const userController = new UserController(userService);

userController.addUser("Alice");
userController.addUser("Bob");
userController.showUsers();
```

## 실제 활용 예시 (Express)

```js
// Infrastructure Layer - DB
const users = [];

class UserRepository {
  add(user) { users.push(user); }
  getAll() { return users; }
}

// Application Layer - Service
class UserService {
  constructor(repo) { this.repo = repo; }
  createUser(name) { this.repo.add({ name }); }
  listUsers() { return this.repo.getAll(); }
}

// Presentation Layer - Controller
const express = require("express");
const app = express();
app.use(express.json());

const userService = new UserService(new UserRepository());

app.post("/users", (req, res) => {
  userService.createUser(req.body.name);
  res.send("User added");
});

app.get("/users", (req, res) => {
  res.json(userService.listUsers());
});

app.listen(3000, () => console.log("Server running on port 3000"));
```

## 언제 사용하면 좋은가?

- 시스템 규모가 커서 **책임과 역할 분리가 필요할 때**
- 유지보수와 테스트를 쉽게 하고 싶을 때
- 각 계층별로 **독립적인 교체 또는 확장**이 필요할 때

## 정리

- **장점**: 계층별 책임 명확, 유지보수 용이, 확장성과 테스트 용이
- **단점**: 계층 간 호출이 많으면 성능 저하 가능, 작은 시스템에는 오버헤드
- **사용 예시**: 엔터프라이즈 웹 애플리케이션, Java/Spring, NodeJS/Express 구조
