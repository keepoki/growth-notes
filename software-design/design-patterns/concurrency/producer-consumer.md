---
tags:
  - DesignPattern
  - Concurrency
title: Producer-Consumer Pattern
---

- **Producer**: 데이터를 생성하는 쪽
- **Consumer**: 데이터를 소비(처리)하는 쪽 둘 사이에 **공유 버퍼(Queue)를 두고 비동기적으로 분리**하는 패턴.

**생산자와 소비자의 속도를 독립적으로 유지**할 수 있어서, 웹 서버, 메시지 큐, 스트리밍 시스템 등에서 많이 사용됩니다.

## 기본 구현

```js
class ProducerConsumer {
  constructor() {
    this.queue = [];
    this.consumers = [];
  }

  // Producer
  produce(item) {
    console.log(`📦 생산됨: ${item}`);
    this.queue.push(item);
    this.notify();
  }

  // Consumer
  consume() {
    return new Promise((resolve) => {
      if (this.queue.length > 0) {
        const item = this.queue.shift();
        console.log(`🙋 소비됨: ${item}`);
        resolve(item);
      } else {
        this.consumers.push(resolve); // 대기
      }
    });
  }

  // 대기 중인 Consumer에게 알림
  notify() {
    if (this.queue.length > 0 && this.consumers.length > 0) {
      const consumer = this.consumers.shift();
      const item = this.queue.shift();
      console.log(`🙋 소비됨: ${item}`);
      consumer(item);
    }
  }
}

// 사용 예시
const pc = new ProducerConsumer();

// 소비자 (비동기 소비)
(async () => {
  while (true) {
    const item = await pc.consume();
    await new Promise((res) => setTimeout(res, 500)); // 처리 시간
    console.log(`✅ 처리 완료: ${item}`);
  }
})();

// 생산자 (데이터 생산)
setInterval(() => {
  const item = `Task-${Math.floor(Math.random() * 100)}`;
  pc.produce(item);
}, 300);
```

## 실제 활용 예시 코드 (Node.js + `bullmq` -> Redis 기반 큐)

```js
// 설치: npm install bullmq ioredis
const { Queue, Worker } = require("bullmq");
const connection = { host: "127.0.0.1", port: 6379 };

// Producer
const taskQueue = new Queue("tasks", { connection });

async function produceTask(data) {
  await taskQueue.add("job", data);
  console.log(`📦 작업 등록: ${JSON.stringify(data)}`);
}

// Consumer
const worker = new Worker(
  "tasks",
  async (job) => {
    console.log(`🙋 소비자 처리 중: ${job.id}, data=${JSON.stringify(job.data)}`);
    await new Promise((res) => setTimeout(res, 1000));
    console.log(`✅ 완료: ${job.id}`);
  },
  { connection }
);

// 테스트
produceTask({ type: "email", to: "user@example.com" });
produceTask({ type: "report", id: 123 });
```

## 언제 사용하면 좋은가?

- **비동기 작업 큐** (이메일 발송, 이미지 처리, 결제 처리)
- **스트리밍 시스템** (Kafka, RabbitMQ)
- **백엔드 마이크로서비스 간 메시지 교환**
- **속도 불균형 완화**
  - Producer가 빠르고 Consumer가 느려도 버퍼가 조정해줌

## 정리

- Producer-Consumer = "생산자 ↔ Queue ↔ 소비자"
- 장점: **속도 독립성**, **버퍼링**, **확장성**
- 단점: 큐 관리 필요 (메모리, 장애 처리)

## 관련 개념

- [[thread-pool]]: Consumer를 Worker 스레드로 구현
- [[event-driven]]: 이벤트 기반 Producer-Consumer
- [[saga]]: 분산 트랜잭션에서의 비동기 메시지 처리
- [[observer]]: 이벤트 생성과 소비 패턴
- [[designpattern]]: 디자인 패턴 전체 개요
