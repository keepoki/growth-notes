#DesignPattern

# Service Discovery

분산 시스템(특히 **마이크로서비스**)에서는 서비스 인스턴스가 **동적으로 생성되고 사라집니다.**  
예를 들어, `User Service`, `Order Service`, `Payment Service` 등이 각각 여러 서버나 컨테이너에 배포될 수 있습니다.

**Service Discovery**는 이런 서비스들의 **위치(IP, Port)** 를 자동으로 탐색하고 관리하는 패턴입니다.
즉, "주문 서비스가 어디 있는지 자동으로 찾아 연결해주는" 시스템입니다.

## 기본 구현

```js
// Service Registry (간단한 인메모리 저장소)
class ServiceRegistry {
  constructor() {
    this.services = new Map();
  }

  register(serviceName, host, port) {
    this.services.set(serviceName, { host, port, timestamp: Date.now() });
    console.log(`Registered: ${serviceName} at ${host}:${port}`);
  }

  unregister(serviceName) {
    this.services.delete(serviceName);
    console.log(`Unregistered: ${serviceName}`);
  }

  get(serviceName) {
    return this.services.get(serviceName);
  }
}

// 사용 예시
const registry = new ServiceRegistry();
registry.register("user-service", "localhost", 3001);
registry.register("order-service", "localhost", 3002);

console.log("Find user service:", registry.get("user-service"));
```

## 실제 활용 예시 (Node.js + Express 시뮬레이션)

`service-registry.js`

```js
// 중앙 서비스 레지스트리 서버
const express = require("express");
const app = express();
app.use(express.json());

const services = {};

app.post("/register", (req, res) => {
  const { name, host, port } = req.body;
  services[name] = { host, port };
  console.log(`Registered ${name} at ${host}:${port}`);
  res.sendStatus(200);
});

app.get("/discover/:name", (req, res) => {
  const service = services[req.params.name];
  if (service) res.json(service);
  else res.status(404).json({ error: "Service not found" });
});

app.listen(4000, () => console.log("Service Registry running on 4000"));
```

`user-service.js`

```js
const express = require("express");
const fetch = require("node-fetch");
const app = express();
const port = 3001;

(async () => {
  // 서비스 등록
  await fetch("http://localhost:4000/register", {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({ name: "user-service", host: "localhost", port }),
  });
})();

app.get("/user/:id", (req, res) => {
  res.json({ id: req.params.id, name: "Alice" });
});

app.listen(port, () => console.log(`User Service running on ${port}`));
```

`gateway.js` (서비스 탐색 및 요청)

```js
const express = require("express");
const fetch = require("node-fetch");
const app = express();
const port = 5000;

app.get("/api/user/:id", async (req, res) => {
  const serviceRes = await fetch("http://localhost:4000/discover/user-service");
  const service = await serviceRes.json();
  const userRes = await fetch(`http://${service.host}:${service.port}/user/${req.params.id}`);
  const user = await userRes.json();
  res.json(user);
});

app.listen(port, () => console.log(`API Gateway running on ${port}`));
```

이제 Gateway → Registry → User-Service로 자동 라우팅됨.

## 언제 사용하면 좋은가?

- **마이크로서비스 아키텍처** 환경에서 서비스 인스턴스가 **동적 확장/축소**되는 경우
- 컨테이너 오케스트레이션 (Kubernetes, Docker Swarm) 환경
- **로드 밸런싱**, **헬스체크**, **자동 장애 복구**를 지원하고 싶을 때

## 정리

### 장점

  - 서비스 위치를 자동으로 관리 → 구성 단순화
  - 인스턴스 추가/삭제 시 자동 반영
  - 확장성, 유연성 향상

### 단점

  - 중앙 서비스 레지스트리 장애 시 전체 시스템 영향 가능
  - 네트워크 트래픽 증가

### 사용 예시
  - Netflix Eureka
  - Consul, Etcd, Zookeeper
  - Kubernetes Service & DNS
