---
tags:
  - DesignPattern
  - EnterpriseApplication
  - Transaction
title: Unit of Work Pattern
---

- 여러 개의 데이터 변경 작업(Insert, Update, Delete)을 **한 트랜잭션 단위로 묶어서 처리**하는 패턴
- 즉, **변경 사항을 추적**하다가,
  - `commit()` 시점에 한꺼번에 DB 반영
  - `rollback()` 시점에 취소 가능

마치 "장바구니에 담아뒀다가, 결제 시 한꺼번에 반영"하는 느낌.

## 기본 구현

```js
class UnitOfWork {
  constructor() {
    this.newObjects = [];
    this.dirtyObjects = [];
    this.removedObjects = [];
  }

  registerNew(obj) {
    this.newObjects.push(obj);
  }

  registerDirty(obj) {
    this.dirtyObjects.push(obj);
  }

  registerRemoved(obj) {
    this.removedObjects.push(obj);
  }

  commit(repository) {
    this.newObjects.forEach(obj => repository.add(obj));
    this.dirtyObjects.forEach(obj => repository.update(obj));
    this.removedObjects.forEach(obj => repository.remove(obj));

    console.log("✅ 모든 변경 사항 커밋 완료");
    this.clear();
  }

  rollback() {
    console.log("↩️ 롤백: 변경 사항 취소");
    this.clear();
  }

  clear() {
    this.newObjects = [];
    this.dirtyObjects = [];
    this.removedObjects = [];
  }
}

// Repository
class UserRepository {
  constructor() {
    this.db = [];
  }

  add(user) {
    this.db.push(user);
    console.log(`➕ 추가됨: ${user.name}`);
  }

  update(user) {
    console.log(`✏️ 수정됨: ${user.name}`);
  }

  remove(user) {
    this.db = this.db.filter(u => u.id !== user.id);
    console.log(`🗑 삭제됨: ${user.name}`);
  }

  getAll() {
    return this.db;
  }
}

// 사용 예시
const userRepo = new UserRepository();
const uow = new UnitOfWork();

const alice = { id: 1, name: "Alice" };
const bob = { id: 2, name: "Bob" };

uow.registerNew(alice);
uow.registerNew(bob);
uow.registerRemoved(alice);
uow.commit(userRepo);
console.log(userRepo.getAll()); // [ { id: 2, name: 'Bob' } ]
```

## 실제 활용 예시 (Node.js + TypeORM)

```js
// service/UserService.js
class UserService {
  constructor(dataSource) {
    this.dataSource = dataSource; // TypeORM DataSource
  }

  async registerUsers(users) {
    const queryRunner = this.dataSource.createQueryRunner();
    await queryRunner.connect();
    await queryRunner.startTransaction();

    try {
      for (const user of users) {
        await queryRunner.manager.save("User", user);
      }
      await queryRunner.commitTransaction();
      console.log("✅ 트랜잭션 커밋 완료");
    } catch (err) {
      await queryRunner.rollbackTransaction();
      console.error("↩️ 롤백: ", err.message);
    } finally {
      await queryRunner.release();
    }
  }
}

// 사용 예시
// userService.registerUsers([ { name: "Alice" }, { name: "Bob" } ]);
```

## 언제 사용하면 좋은가?

- **트랜잭션 단위로 일괄 처리**해야 할 때
  - 예: 주문 + 결제 + 재고 차감 → 하나라도 실패하면 전부 취소
- **객체 변경 추적**을 자동화하고 싶을 때 (ORM에서 자주 사용)
- DB 작업의 **일관성과 무결성** 보장 필요할 때

## 정리

- Unit of Work = "변경 사항을 모아두고 한꺼번에 Commit/Rollback"
- 장점: **트랜잭션 관리, 무결성 보장**
- 단점: 단순 CRUD에는 오버엔지니어링

## 관련 개념

- [[repository]]: Unit of Work와 함께 사용되는 패턴
- [[data-mapper]]: 객체와 DB 매핑
- [[identity-map]]: 객체 캐싱 패턴
- [[service-layer]]: Unit of Work를 사용하는 계층
- [[ddd-(domain-driven-design)]]: 애그리게잇 트랜잭션 관리
