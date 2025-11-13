---
tags:
  - DesignPattern
  - Concurrency
title: Active Object Pattern
---

객체의 메서드 호출을 **비동기적으로 실행**하면서,  호출자(Caller)와 실행자(Worker)를 분리하는 패턴입니다.
호출자는 그냥 "메서드 호출"하듯 쓰지만 → 내부적으로는 비동기 큐에 들어가 Worker 스레드(또는 이벤트 루프)가 실행.

## 기본 구현

```js
class ActiveObject {
  constructor() {
    this.queue = [];
    this.isProcessing = false;
  }

  async run() {
    if (this.isProcessing) return;
    this.isProcessing = true;

    while (this.queue.length > 0) {
      const task = this.queue.shift();
      await task(); // 큐에 들어온 작업 실행
    }

    this.isProcessing = false;
  }

  callAsync(fn) {
    return new Promise((resolve) => {
      this.queue.push(async () => {
        const result = await fn();
        resolve(result);
      });
      this.run(); // 큐 실행 시작
    });
  }
}

// 사용
const activeObj = new ActiveObject();

activeObj.callAsync(() => new Promise(res => setTimeout(() => res("작업 1 완료"), 1000)))
  .then(console.log);

activeObj.callAsync(() => Promise.resolve("작업 2 완료"))
  .then(console.log);
```

## 실제 활용 예시

- **UI 이벤트 처리**
  - 버튼 클릭 시 비동기 동작을 ActiveObject 큐에 넣고 순서대로 실행
- **백엔드 작업 큐**
  - Node.js에서 요청이 들어올 때 → ActiveObject 패턴으로 직렬화 실행

## 언제 사용하면 좋은가?

- 호출자가 "비동기 코드"를 직접 관리하지 않고, 동기 호출처럼 쓰고 싶을 때
- 이벤트 루프, 스레드풀 같은 **비동기 실행 모델**을 캡슐화할 때
- 예:
  - 비동기 DB 작업 관리
  - UI에서 긴 작업을 큐잉하여 실행

## 정리

- Active Object = "메서드 호출을 큐에 넣고 비동기적으로 실행"
- 장점: 코드 간단해지고 동기처럼 보임
- 단점: 큐 병목, 작업 우선순위 관리 필요

## 관련 개념

- [[producer-consumer]]: 비동기 작업 큐 패턴
- [[command]]: 요청을 객체로 캡슐화
- [[proxy]]: 비동기 호출을 감싸는 패턴
- [[thread-pool]]: Worker 스레드로 실행
- [[designpattern]]: 디자인 패턴 전체 개요
