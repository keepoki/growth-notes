#DesignPattern 

# bulkhead

**Bulkhead(벌크헤드)** 는 배의 “격벽”에서 유래된 개념입니다.
배가 침몰하지 않도록 구역을 나누듯이, 시스템에서도 **각 서비스나 리소스를 분리해 장애가 전체로 퍼지지 않게 하는 패턴**입니다.

> “한 서비스가 장애나 과부하에 걸려도,  
> 다른 서비스는 정상적으로 동작하도록 보호하는 구조.”

즉, **하나의 모듈이나 스레드 풀, 네트워크 연결 풀을 독립적으로 격리**시켜서 서로 간섭하지 못하게 만드는 것이 핵심입니다.

## 기본 구현

요청 격리
```js
class Bulkhead {
  constructor(limit) {
    this.limit = limit; // 동시에 실행 가능한 최대 작업 수
    this.activeCount = 0;
    this.queue = [];
  }

  async execute(task) {
    if (this.activeCount >= this.limit) {
      console.log("⚠️ Queueing task...");
      return new Promise((resolve) => this.queue.push(() => resolve(this.execute(task))));
    }

    this.activeCount++;
    try {
      const result = await task();
      return result;
    } finally {
      this.activeCount--;
      if (this.queue.length > 0) {
        const next = this.queue.shift();
        next();
      }
    }
  }
}

// 사용 예시
const bulkhead = new Bulkhead(2); // 동시에 2개만 실행 허용

async function simulateRequest(id) {
  console.log(`Start task ${id}`);
  await new Promise((r) => setTimeout(r, 1000));
  console.log(`Finish task ${id}`);
}

(async () => {
  for (let i = 1; i <= 5; i++) {
    bulkhead.execute(() => simulateRequest(i));
  }
})();
```
실행 결과
```sql
Start task 1
Start task 2
⚠️ Queueing task...
⚠️ Queueing task...
Finish task 1
Start task 3
Finish task 2
Start task 4
...
```

## 실제 활용 예시 (Node.js + Microservices 시나리오)

예를 들어,  `OrderService`가 동시에 너무 많은 요청을 `PaymentService`로 보낸다면, 결제 서비스가 다운될 수도 있습니다.

이를 방지하기 위해 각 외부 호출마다 **별도 연결 풀**을 둡니다.

```js
const fetch = require("node-fetch");

class PaymentClient {
  constructor(maxConcurrent) {
    this.bulkhead = new Bulkhead(maxConcurrent);
  }

  async requestPayment(orderId) {
    return this.bulkhead.execute(async () => {
      console.log(`💳 Processing payment for order ${orderId}`);
      const res = await fetch("http://payment-service/pay", { method: "POST" });
      return res.ok;
    });
  }
}

const paymentClient = new PaymentClient(5); // 동시에 5건까지만 허용
```
이를 통해 **한 서비스의 과도한 트래픽이 전체 장애로 확산되는 것을 방지**할 수 있습니다.

## 언제 사용하면 좋은가?

- 특정 서비스가 **자원(스레드, DB 연결, 네트워크 등)을 과도하게 점유**할 가능성이 있을 때
- 여러 외부 서비스(API, DB 등)를 병렬로 호출하지만,  
    하나의 실패가 전체 장애로 이어지지 않게 해야 할 때
- 마이크로서비스 아키텍처에서 **서비스 격리(fault isolation)** 가 중요할 때

## 정리

### 장점

- 장애 격리 → 전체 시스템 안정성 향상
- 서비스 간 자원 경쟁 방지
- 과부하 방어 (Rate Limit와 함께 사용 시 강력함)

### 단점

- 구성 복잡도 증가
- 리소스 할당 기준을 잘못 설정하면 오히려 성능 저하
- 큐 대기 시간이 생길 수 있음

### 사용 예시

- Netflix Hystrix (Thread Pool로 서비스 격리)
- Node.js + Semaphore 구현
- AWS Lambda 동시 실행 제한, Kubernetes Pod Resource Limit