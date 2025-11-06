#Architecture

# CQRS (Command Query Responsibility Segregation)

CQRS는 시스템에서 **읽기(Query)와 쓰기(Command)를 분리하는 아키텍처 패턴**입니다.

- **Command**: 데이터를 변경하는 작업 (Create, Update, Delete)
- **Query**: 데이터를 읽는 작업 (Read)  
    보통 일반적인 CRUD에서는 같은 모델/엔드포인트가 읽기와 쓰기를 모두 담당하지만, 
    CQRS는 **읽기와 쓰기 모델을 분리**하여 확장성, 성능, 복잡한 도메인 처리를 효율적으로 한다.

## 기본 구현

```js
// Command (쓰기)
class UserCommandService {
  constructor(userRepository) {
    this.userRepository = userRepository;
  }

  createUser(name) {
    const user = { id: Date.now(), name };
    this.userRepository.save(user);
    return user;
  }
}

// Query (읽기)
class UserQueryService {
  constructor(userRepository) {
    this.userRepository = userRepository;
  }

  getUserById(id) {
    return this.userRepository.findById(id);
  }
}

// Repository (공용 저장소)
class UserRepository {
  constructor() {
    this.users = new Map();
  }
  save(user) {
    this.users.set(user.id, user);
  }
  findById(id) {
    return this.users.get(id);
  }
}

// 실행
const repo = new UserRepository();
const commandService = new UserCommandService(repo);
const queryService = new UserQueryService(repo);

const user = commandService.createUser("Alice");
console.log(queryService.getUserById(user.id));
```

## 실제 활용 예시 (React + API 분리)

### 백엔드 예시 (Express)

```js
const express = require("express");
const app = express();
app.use(express.json());

const users = [];

// Command API
app.post("/users", (req, res) => {
  const user = { id: Date.now(), name: req.body.name };
  users.push(user);
  res.json(user);
});

// Query API
app.get("/users/:id", (req, res) => {
  const user = users.find(u => u.id == req.params.id);
  res.json(user || {});
});

app.listen(3000, () => console.log("CQRS API running"));
```

### 프론트엔드 예시 (React - Qeuery와 Command 분리)

```js
import React, { useState } from "react";

export default function UserPage() {
  const [user, setUser] = useState(null);

  async function createUser(name) {
    const res = await fetch("/users", {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({ name }),
    });
    const data = await res.json();
    console.log("User Created:", data);
  }

  async function getUser(id) {
    const res = await fetch(`/users/${id}`);
    const data = await res.json();
    setUser(data);
  }

  return (
    <div>
      <button onClick={() => createUser("Alice")}>Create User</button>
      <button onClick={() => getUser(1)}>Get User</button>
      {user && <p>User: {user.name}</p>}
    </div>
  );
}
```

## 언제 사용하면 좋은가?

- 시스템의 **읽기/쓰기 요구사항이 크게 다른 경우**
    - 예: 읽기 요청은 초당 수천 건, 쓰기 요청은 적은 경우
- **복잡한 도메인 로직**이 있어 Command와 Query를 나누어 관리하면 이해/유지보수가 쉬움
- **확장성**이 중요한 경우 (읽기/쓰기 DB 분리 → 예: Master-Slave 구조)
- 이벤트 소싱(Event Sourcing)과 함께 자주 사용됨

## 정리

**장점**
- 성능 최적화 가능 (읽기 전용 DB 캐싱 등)
- 도메인 복잡성이 큰 시스템에서 코드 명확성 ↑
- 확장성과 유연성 확보
**단점**
- 구조가 복잡해지고 구현 비용 ↑
- 작은 프로젝트에서는 오버엔지니어링 될 수 있음
**사용 예시**
- 전자상거래 (주문: 쓰기, 주문 조회: 읽기)
- 금융 시스템 (거래 생성 vs 거래 내역 조회)
- 고성능 API (읽기와 쓰기를 다른 서버로 분리
