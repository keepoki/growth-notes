#DesignPattern 

# Leader-Followers

여러 개의 스레드(또는 워커)가 있을 때,
- **Leader**: 이벤트를 기다리다가 → 처리할 작업 발견 → **Follower**로 교체 후 처리
- **Follower**: 대기 중 → Leader가 빠지면 → 새 Leader가 됨

**스레드풀 안에서 효율적으로 이벤트를 분배하는 패턴**입니다.  
Node.js 같은 단일 이벤트 루프 환경보다는 **멀티스레드 서버**나 **Worker Pool**에서 자주 사용됩니다.

## 기본 구조

```js
class LeaderFollowers {
  constructor(workerCount = 3) {
    this.queue = [];
    this.workers = new Array(workerCount).fill(null).map((_, i) => ({
      id: i + 1,
      isLeader: false,
    }));
    this.currentLeader = null;
  }

  addTask(task) {
    this.queue.push(task);
    this.assignLeader();
  }

  assignLeader() {
    if (this.currentLeader || this.queue.length === 0) return;

    // 대기 중인 워커 중 한 명을 Leader로 선택
    const nextLeader = this.workers.find((w) => !w.isLeader);
    if (!nextLeader) return;

    nextLeader.isLeader = true;
    this.currentLeader = nextLeader;
    this.processTask();
  }

  async processTask() {
    const task = this.queue.shift();
    if (!task) return;

    console.log(`👑 Worker ${this.currentLeader.id} (Leader) 처리 시작: ${task}`);
    await new Promise((res) => setTimeout(res, 1000));
    console.log(`✅ Worker ${this.currentLeader.id} 처리 완료: ${task}`);

    // Leader -> Follower로 변경
    this.currentLeader.isLeader = false;
    this.currentLeader = null;

    // 다음 Leader 선출
    if (this.queue.length > 0) {
      this.assignLeader();
    }
  }
}

// 사용 예시
const system = new LeaderFollowers(3);

system.addTask("요청 A");
system.addTask("요청 B");
system.addTask("요청 C");
system.addTask("요청 D");
```

## 실제 활용 예시 코드 (Node.js Worker Threads 활용)

```js
const { Worker, isMainThread, parentPort } = require("worker_threads");

if (isMainThread) {
  // 메인 스레드 (Leader-Followers 매니저)
  const tasks = ["파일 읽기", "DB 쿼리", "API 호출", "로그 저장"];
  const workers = [];

  for (let i = 0; i < 2; i++) {
    const worker = new Worker(__filename);
    workers.push(worker);

    worker.on("message", (msg) => {
      console.log(`✅ 워커 응답: ${msg}`);

      // 다음 Task 할당
      const nextTask = tasks.shift();
      if (nextTask) {
        worker.postMessage(nextTask);
      }
    });
  }

  // 첫 번째 Task 할당 (Leader 역할)
  workers.forEach((w, i) => {
    const task = tasks.shift();
    if (task) w.postMessage(task);
  });
} else {
  // 워커 스레드 (Follower -> Leader 전환)
  parentPort.on("message", async (task) => {
    console.log(`👷 워커 작업 시작: ${task}`);
    await new Promise((res) => setTimeout(res, 1000));
    parentPort.postMessage(`작업 완료: ${task}`);
  });
}
```

## 언제 사용하면 좋은가?

- **멀티스레드 서버**: Nginx, IIS, Java NIO 서버
- **스레드풀 기반 워커 관리**: Node.js Worker Threads, Python concurrent.futures
- **리소스 관리 최적화**: Leader만 이벤트 기다리므로 CPU 낭비 줄임

## 정리

- Leader-Followers = "스레드풀에서 Leader가 이벤트 처리 → 끝나면 다른 Follower가 Leader로 승격"
- 장점: **리소스 효율성**, **스레드 전환 최소화**
- 단점: **구현 난이도**, Leader 교체 관리 필요