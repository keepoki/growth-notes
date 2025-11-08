#DesignPattern

# API Gateway

**API Gateway**는 마이크로서비스 아키텍처에서 **모든 클라이언트 요청의 단일 진입점(Entry Point)** 역할을 하는 패턴입니다.

- 클라이언트는 각각의 서비스를 직접 호출하지 않고, **Gateway**를 통해 요청을 보냄.
- Gateway는 **요청 라우팅, 인증, 로깅, 캐싱, 요청 집계, 장애 처리** 등을 담당.

"모든 서비스로 가는 문은 하나뿐이고, 그 문이 바로 API Gateway다."

## 기본 구현

```js
const express = require("express");
const fetch = require("node-fetch");
const app = express();
const PORT = 8080;

// 라우팅 설정
app.get("/api/users", async (req, res) => {
  const response = await fetch("http://localhost:3001/users");
  const data = await response.json();
  res.json(data);
});

app.get("/api/orders", async (req, res) => {
  const response = await fetch("http://localhost:3002/orders");
  const data = await response.json();
  res.json(data);
});

app.listen(PORT, () => console.log(`API Gateway running on ${PORT}`));
```

`/api/users` 요청이 실제로는 `User Service`로,  `/api/orders` 요청이 `Order Service`로 자동 라우팅됨

## 실제 활용 예시 (Node.js + Express + JWT 인증)

`apit-gateway.js`

```js
const express = require("express");
const fetch = require("node-fetch");
const jwt = require("jsonwebtoken");
const app = express();
const PORT = 4000;

// 미들웨어: JWT 인증
app.use((req, res, next) => {
  if (req.path.startsWith("/public")) return next();

  const token = req.headers.authorization?.split(" ")[1];
  if (!token) return res.status(401).json({ error: "Unauthorized" });

  try {
    req.user = jwt.verify(token, "SECRET_KEY");
    next();
  } catch {
    res.status(403).json({ error: "Invalid token" });
  }
});

// 라우팅
app.get("/api/users/:id", async (req, res) => {
  const response = await fetch(`http://localhost:3001/users/${req.params.id}`);
  const data = await response.json();
  res.json(data);
});

app.get("/api/orders/:userId", async (req, res) => {
  const response = await fetch(`http://localhost:3002/orders/${req.params.userId}`);
  const data = await response.json();
  res.json(data);
});

// 공용 라우트 (예: 로그인)
app.get("/public/health", (req, res) => res.json({ status: "OK" }));

app.listen(PORT, () => console.log(`API Gateway running on ${PORT}`));
```

`user-service.js`

```js
const express = require("express");
const app = express();
app.get("/users/:id", (req, res) => res.json({ id: req.params.id, name: "Alice" }));
app.listen(3001, () => console.log("User Service on 3001"));
```

`order-service.js`

```js
const express = require("express");
const app = express();
app.get("/orders/:userId", (req, res) => res.json([{ id: 1, userId: req.params.userId, item: "Book" }]));
app.listen(3002, () => console.log("Order Service on 3002"));
```

클라이언트는 오직 `http://localhost:4000/api/...`만 호출하면 됨. Gateway가 모든 인증, 라우팅, 로깅을 대신 처리함.

## 언제 사용하면 좋은가?

- 마이크로서비스 아키텍처에서 클라이언트가 여러 서비스를 직접 호출해야 하는 경우
- 공통 기능(인증, 로깅, 캐싱, 압축 등)을 **중앙 집중화**하고 싶을 때
- API 버전 관리, 요청 제한(rate limiting), 트래픽 제어가 필요한 경우

## 정리

- **장점**
  - 클라이언트 복잡도 ↓ (단일 진입점)
  - 인증/보안/로깅/캐싱 통합 관리
  - 요청 집계 가능 (여러 서비스 데이터를 한 번에 반환 가능)

- **단점**
  - Gateway가 **단일 장애점(Single Point of Failure)**
  - 요청 지연(latency) 증가 가능
  - 관리 복잡도 증가 (특히 대규모 시스템에서)

- **사용 예시**
  - Netflix Zuul
  - NGINX / Kong / AWS API Gateway / Express Gateway
  - GraphQL Federation (API Gateway의 진화형)
