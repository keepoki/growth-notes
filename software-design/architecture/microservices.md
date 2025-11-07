#Architecture

# Microservices

마이크로서비스 아키텍처는 애플리케이션을 **작고 독립적으로 배포 가능한 서비스 단위**로 나누어 설계하는 아키텍처 패턴입니다.  
각 서비스는 **독립적인 비즈니스 기능**을 담당하며, **자체 데이터베이스와 로직**을 갖고, **API(주로 HTTP/REST, gRPC, 메시징)** 를 통해 통신합니다.

특징:

- 서비스 간 **독립적 배포 가능**
- 각 서비스는 다른 언어/프레임워크 사용 가능
- 서비스 간 데이터베이스 공유 ❌ (서비스별 독립적 데이터 저장소 보유)

## 기본 구현

```js
// user-service.js
const express = require("express");
const app = express();
app.use(express.json());

let users = [];

app.post("/users", (req, res) => {
  users.push({ id: users.length + 1, name: req.body.name });
  res.send("User created");
});

app.get("/users", (req, res) => {
  res.json(users);
});

app.listen(3001, () => console.log("User Service running on 3001"));


// order-service.js
const express = require("express");
const axios = require("axios");
const app = express();
app.use(express.json());

let orders = [];

app.post("/orders", async (req, res) => {
  const userId = req.body.userId;
  // 다른 서비스(User Service)에 요청
  const { data: users } = await axios.get("http://localhost:3001/users");
  const user = users.find(u => u.id === userId);

  if (!user) return res.status(400).send("User not found");

  orders.push({ id: orders.length + 1, userId });
  res.send("Order created for " + user.name);
});

app.get("/orders", (req, res) => {
  res.json(orders);
});

app.listen(3002, () => console.log("Order Service running on 3002"));
```

## 실제 활용 예시

- **Netflix**: 스트리밍, 추천, 결제 등을 개별 서비스로 운영
- **Amazon**: 상품, 결제, 배송, 추천 시스템을 각각 독립 서비스로 운영
- **쿠팡**: 주문, 결제, 배송 추적 서비스가 각각 독립적으로 배포됨

## 언제 사용하면 좋은가?

- 서비스 규모가 커져서 **단일 모놀리식(monolith) 구조가 복잡**해졌을 때
- **팀 단위로 독립 개발/배포**를 하고 싶을 때
- 특정 기능(결제, 채팅 등)만 독립적으로 확장/배포해야 할 때
- 장애가 다른 서비스로 전파되지 않도록 격리하고 싶을 때

## 정리

### 장점

- 서비스별 독립적 배포/확장 가능
- 장애 격리 → 한 서비스가 죽어도 전체 다운 ❌
- 다양한 기술 스택 사용 가능

### 단점

- 서비스 간 통신 복잡 (네트워크 비용 증가)
- 분산 시스템 관리 난이도 ↑ (로그, 모니터링, 배포)
- 데이터 일관성 관리 어려움

### 사용 예시

- 넷플릭스, 아마존, 쿠팡, 카카오, 네이버 등 대규모 서비스
