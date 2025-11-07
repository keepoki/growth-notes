#DesignPattern

# Thread Pool

새로운 스레드를 매번 만드는 대신, **미리 생성된 스레드 묶음(풀)을 두고** 요청이 들어올 때마다 **빈 스레드에 작업 할당**하고 작업 끝나면 스레드를 반납하여 재사용합니다.
성능 개선 + 리소스 절약을 위해 **웹 서버, DB, 메시지 브로커** 등에서 거의 필수적으로 사용됩니다.

## 기본 구현

```js
class ThreadPool {
  constructor(size) {
    this.size = size;
    this.queue = [];
    this.activeWorkers = 0;
  }

  async runTask(task) {
    if (this.activeWorkers < this.size) {
      this.activeWorkers++;
      try {
        await task();
      } finally {
        this.activeWorkers--;
        this.next();
      }
    } else {
      this.queue.push(task);
    }
  }

  next() {
    if (this.queue.length > 0 && this.activeWorkers < this.size) {
      const task = this.queue.shift();
      this.runTask(task);
    }
  }
}

// 사용 예시
const pool = new ThreadPool(2);

function createTask(id, duration) {
  return async () => {
    console.log(`🟢 시작: 작업 ${id}`);
    await new Promise((res) => setTimeout(res, duration));
    console.log(`✅ 완료: 작업 ${id}`);
  };
}

pool.runTask(createTask(1, 2000));
pool.runTask(createTask(2, 1000));
pool.runTask(createTask(3, 500));
pool.runTask(createTask(4, 800));

/*
출력: 
🟢 시작: 작업 1
🟢 시작: 작업 2
✅ 완료: 작업 2
🟢 시작: 작업 3
✅ 완료: 작업 1
🟢 시작: 작업 4
✅ 완료: 작업 3
✅ 완료: 작업 4
*/
```

## 실제 활용 코드 예시 (Node.js Worker Threads 기반)

```js
const { Worker } = require("worker_threads");

class WorkerPool {
  constructor(size, workerFile) {
    this.size = size;
    this.workerFile = workerFile;
    this.workers = [];
    this.queue = [];

    for (let i = 0; i < size; i++) {
      this.workers.push(this.createWorker());
    }
  }

  createWorker() {
    const worker = new Worker(this.workerFile);
    worker.isBusy = false;
    worker.on("message", () => {
      worker.isBusy = false;
      this.next();
    });
    return worker;
  }

  runTask(data) {
    const worker = this.workers.find((w) => !w.isBusy);
    if (worker) {
      worker.isBusy = true;
      worker.postMessage(data);
    } else {
      this.queue.push(data);
    }
  }

  next() {
    if (this.queue.length > 0) {
      const task = this.queue.shift();
      this.runTask(task);
    }
  }
}

// worker.js
// const { parentPort } = require("worker_threads");
// parentPort.on("message", (task) => {
//   console.log(`🟢 워커 처리: ${task}`);
//   setTimeout(() => {
//     console.log(`✅ 완료: ${task}`);
//     parentPort.postMessage("done");
//   }, 1000);
// });

// 메인 실행
const pool = new WorkerPool(2, "./worker.js");

pool.runTask("작업 1");
pool.runTask("작업 2");
pool.runTask("작업 3");
pool.runTask("작업 4");
```

## 언제 사용하면 좋은가?

- **고성능 서버 (웹, DB)**
  - ex) Apache, MySQL → Thread Pool 기반
- **백엔드 처리 (이미지 변환, 동영상 인코딩)**
- **멀티코어 활용 (Node.js Worker Threads, Python concurrent.futures)**

## 정리

- Thread Pool = "스레드 재사용으로 성능 향상 & 리소스 절약"
- 장점: **성능**, **리소스 관리**
- 단점: **풀 크기 조절**이 어렵다 (너무 크면 낭비, 너무 작으면 병목)
