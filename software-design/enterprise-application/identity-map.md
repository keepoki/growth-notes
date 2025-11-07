#DesignPattern

# Identity Map

- 동일한 객체가 메모리에 여러 번 로드되지 않도록 **캐싱(객체 맵)을 유지**하는 패턴
- 즉, 한 번 조회한 객체는 맵(Map)에 저장해 두고, 같은 ID로 다시 조회할 때는 **DB 대신 캐시에서 반환**
- 불필요한 DB 쿼리를 줄이고, 동일 객체를 항상 동일 인스턴스로 다룰 수 있음

같은 학생을 두 번 DB에서 꺼내면, 같은 객체 참조가 리턴되도록 하는 캐시

## 기본 구현

```js
class User {
  constructor(id, name) {
    this.id = id;
    this.name = name;
  }
}

class IdentityMap {
  constructor() {
    this.map = new Map(); // { id: User }
  }

  get(id) {
    return this.map.get(id);
  }

  add(user) {
    this.map.set(user.id, user);
  }

  has(id) {
    return this.map.has(id);
  }
}

// 가짜 DB
const fakeDB = [
  { id: 1, name: "Alice" },
  { id: 2, name: "Bob" },
];

// Repository with Identity Map
class UserRepository {
  constructor(db) {
    this.db = db;
    this.identityMap = new IdentityMap();
  }

  findById(id) {
    if (this.identityMap.has(id)) {
      console.log("⚡ 캐시에서 반환");
      return this.identityMap.get(id);
    }
    console.log("🔍 DB 조회");
    const row = this.db.find(r => r.id === id);
    if (!row) return null;
    const user = new User(row.id, row.name);
    this.identityMap.add(user);
    return user;
  }
}

// 사용 예시
const repo = new UserRepository(fakeDB);

const user1 = repo.findById(1); // 🔍 DB 조회
const user2 = repo.findById(1); // ⚡ 캐시에서 반환

console.log(user1 === user2); // true (같은 객체)
```

## 실제 활용 예시 (ORM 내부)

많은 ORM (예: **Hibernate, TypeORM**)에서 Identity Map은 기본적으로 사용됩니다.

- `EntityManager` 또는 `Session`이 Identity Map을 유지
- 같은 트랜잭션 내에서 동일한 객체는 항상 같은 인스턴스로 반환

예시 (TypeORM):

```js
const user1 = await manager.findOne(User, { where: { id: 1 } });
const user2 = await manager.findOne(User, { where: { id: 1 } });

console.log(user1 === user2); // true (Identity Map 덕분)
```

## 언제 사용하면 좋은가?

- 동일한 엔티티를 여러 번 로드하는 대규모 시스템
- DB 트래픽 최소화 필요할 때
- "같은 객체는 같은 인스턴스"라는 참조 동등성을 보장해야 할 때

## 비교

- **캐싱 일반**
  - 단순히 데이터만 저장
  - Identity Map은 "객체 참조 동일성" 보장 목적
- **Identity Map**
  - 같은 ID는 항상 같은 객체 반환
  - ORM, Repository에서 자주 사용

## 정리

- Identity Map = "같은 객체는 한 번만 메모리에 존재"
- 장점: 성능 ↑, 객체 참조 동일성 보장
- 단점: 캐시 관리(동기화, 무효화)가 필요
