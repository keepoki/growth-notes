#DesignPattern

# Circuit Breaker

- 분산 시스템에서 **외부 서비스 호출 실패**가 계속되면, 장애가 전파되어 전체 시스템이 불안정해질 수 있습니다.
- **Circuit Breaker**는 이를 방지하기 위해, **일정 횟수 이상 실패하면 회로(요청 경로)를 차단(break)** 하고, 일정 시간 후 재시도하게 하는 패턴입니다.
- 전기 회로 차단기와 유사하게 동작합니다.

  - **Closed(닫힘)**: 정상 동작, 요청 전달
  - **Open(열림)**: 오류 과다 발생 → 요청 차단, fallback 동작
  - **Half-Open(반쯤 열림)**: 일부 요청만 시도 → 성공하면 Closed로 복귀, 실패하면 다시 Open

## 기본 구현

```js
class CircuitBreaker {
  constructor(action, failureThreshold = 3, successThreshold = 2, timeout = 5000) {
    this.action = action; // 호출할 외부 서비스
    this.failureThreshold = failureThreshold;
    this.successThreshold = successThreshold;
    this.timeout = timeout;

    this.state = "CLOSED"; // 초기 상태
    this.failureCount = 0;
    this.successCount = 0;
    this.nextAttempt = Date.now();
  }

  async call(...args) {
    if (this.state === "OPEN") {
      if (Date.now() > this.nextAttempt) {
        this.state = "HALF";
      } else {
        throw new Error("Circuit is OPEN. Fallback executed.");
      }
    }

    try {
      const result = await this.action(...args);
      this.successCount++;

      if (this.state === "HALF" && this.successCount >= this.successThreshold) {
        this.state = "CLOSED";
        this.failureCount = 0;
      }

      return result;
    } catch (error) {
      this.failureCount++;
      if (this.failureCount >= this.failureThreshold) {
        this.state = "OPEN";
        this.nextAttempt = Date.now() + this.timeout;
      }
      throw error;
    }
  }
}
```

## 실제 활용 예시 (Node.js + 외부 API 호출)

```js
const fetch = require("node-fetch");

// 외부 API
async function fetchUser(userId) {
  const res = await fetch(`https://jsonplaceholder.typicode.com/users/${userId}`);
  if (!res.ok) throw new Error("API Error");
  return res.json();
}

const breaker = new CircuitBreaker(fetchUser, 3, 2, 5000);

(async () => {
  try {
    const user = await breaker.call(1);
    console.log("User:", user.name);
  } catch (e) {
    console.log("Fallback: returning cached/default user.");
  }
})();
```

## 언제 사용하면 좋은가?

- 마이크로서비스 간 통신 시 특정 서비스가 다운되면, 다른 서비스까지 **연쇄 장애**로 이어질 수 있을 때
- 외부 API 호출 시(예: 결제, 날씨, 배송 서비스) 응답 실패가 잦은 경우
- **고가용성 시스템**이 필요한 환경 (넷플릭스, 우버 등)

## 정리

- **장점**
  - 장애 전파 차단 → 전체 시스템 안정성 확보
  - fallback 처리 가능 → 사용자 경험 보호

- **단점**
  - 상태 관리가 복잡
  - 잘못 설정하면 정상 서비스도 차단될 수 있음

- **사용 예시**
  - Netflix Hystrix (대표적인 구현체)
  - Resilience4j (Java)
  - Node.js: `opossum` 라이브러리
