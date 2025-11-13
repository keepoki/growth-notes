---
tags:
  - DesignPattern
  - Concurrency
title: Half-Sync/Half-Async Pattern
---

시스템을 **비동기 계층(Async Layer)**과 **동기 계층(Sync Layer)으로** 분리하고,  그 사이에 **Queue Layer**를 두어 양쪽을 연결하는 구조입니다.

- **Async 계층** → 이벤트 기반, 빠른 I/O 처리 (ex. 네트워크 소켓)
- **Sync 계층** → 직관적인 비즈니스 로직 처리 (동기적으로 코딩)
- **Queue 계층** → 두 계층 사이에서 작업 전달

## 기본 구조

```js
class HalfSyncHalfAsync {
  constructor() {
    this.queue = [];
    this.isProcessing = false;
  }

  // Async 계층 (예: I/O 이벤트)
  async asyncRequest(data) {
    console.log(`📥 Async Layer: 이벤트 수신 -> ${data}`);
    this.queue.push(data);
    this.processQueue();
  }

  // Sync 계층 (비즈니스 로직)
  async syncProcess(task) {
    console.log(`⚙️ Sync Layer: 처리 중... ${task}`);
    return new Promise((res) =>
      setTimeout(() => res(`✅ 처리 완료: ${task}`), 1000)
    );
  }

  // Queue 계층 (중재자)
  async processQueue() {
    if (this.isProcessing) return;
    this.isProcessing = true;

    while (this.queue.length > 0) {
      const task = this.queue.shift();
      const result = await this.syncProcess(task);
      console.log(result);
    }

    this.isProcessing = false;
  }
}

// 사용 예시
const system = new HalfSyncHalfAsync();

system.asyncRequest("파일 읽기 이벤트");
system.asyncRequest("네트워크 패킷 수신");
system.asyncRequest("DB 응답");
```

## 실제 활용 예시

1. **Node.js 서버 아키텍처**
   - Async Layer: `libuv` 이벤트 루프 (I/O 관리)
   - Sync Layer: JS 애플리케이션 로직
   - Queue Layer: 이벤트 큐

2. **운영체제 커널**
    - Async Layer: Non-blocking I/O
    - Sync Layer: 사용자 모드 애플리케이션
    - Queue Layer: 커널 이벤트 큐

## 언제 사용하면 좋은가?

- **비동기 I/O는 빠르지만, 비즈니스 로직은 동기적으로 다루고 싶을 때**
- 네트워크 서버, DB, 파일 시스템 같은 **고성능 시스템**에서 자주 사용됨
- 복잡한 비동기 처리를 "큐 + 계층 분리"로 단순화 가능

## 정리

- Half-Sync/Half-Async = "Async(I/O) + Queue + Sync(로직)"
- 장점: 성능(Async) + 직관적 코드(Sync) 둘 다 얻음
- 단점: 큐 병목 가능성

## 관련 개념

- [[producer-consumer]]: Async 레이어에서 Queue로 전달
- [[event-driven]]: 비동기 이벤트 처리
- [[thread-pool]]: Sync 레이어의 Worker 구현
- [[designpattern]]: 디자인 패턴 전체 개요
