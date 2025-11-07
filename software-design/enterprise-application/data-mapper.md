#DesignPattern

# Data Mapper

- **도메인 객체(Domain Model)와** **DB 엔티티(Table Row)를** 서로 매핑해주는 전용 계층(Mapper)을 둠
- 도메인 객체는 **비즈니스 로직만 담당**하고, DB 저장/조회 로직은 전혀 알지 못함
- DB 접근은 Mapper가 대신 처리

객체는 DB를 몰라도 되고, Mapper가 객체 ↔ DB 사이에서 번역(매핑)을 담당

## 기본 구현

```js
// 도메인 모델 (DB와 전혀 무관)
class User {
  constructor(id, name) {
    this.id = id;
    this.name = name;
  }

  changeName(newName) {
    this.name = newName;
  }
}

// Data Mapper
class UserMapper {
  constructor(db) {
    this.db = db;
  }

  insert(user) {
    this.db.push({ id: user.id, name: user.name });
    console.log(`💾 저장됨: ${user.name}`);
  }

  findById(id) {
    const row = this.db.find(r => r.id === id);
    return row ? new User(row.id, row.name) : null;
  }

  update(user) {
    const idx = this.db.findIndex(r => r.id === user.id);
    if (idx > -1) {
      this.db[idx].name = user.name;
      console.log(`✏️ 수정됨: ${user.name}`);
    }
  }

  delete(user) {
    const idx = this.db.findIndex(r => r.id === user.id);
    if (idx > -1) {
      this.db.splice(idx, 1);
      console.log(`🗑 삭제됨: ${user.name}`);
    }
  }
}

// 사용 예시
const db = [];
const mapper = new UserMapper(db);

const alice = new User(1, "Alice");
mapper.insert(alice);

const found = mapper.findById(1);
console.log(found instanceof User); // true

found.changeName("Alice Cooper");
mapper.update(found);

mapper.delete(found);
console.log(db);
```

## 실제 활용 예시 (Node.js + TypeORM)

```js
import { Entity, PrimaryGeneratedColumn, Column, DataSource } from "typeorm";

// 도메인 모델 (엔티티 정의)
@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id!: number;

  @Column()
  name!: string;

  changeName(newName: string) {
    this.name = newName;
  }
}

// Repository (실제로는 Mapper 역할)
const AppDataSource = new DataSource({
  type: "sqlite",
  database: ":memory:",
  entities: [User],
  synchronize: true,
});

(async () => {
  await AppDataSource.initialize();
  const userRepo = AppDataSource.getRepository(User);

  const alice = new User();
  alice.name = "Alice";
  await userRepo.save(alice); // INSERT

  const found = await userRepo.findOneBy({ id: 1 });
  console.log(found);

  found!.changeName("Alice Cooper");
  await userRepo.save(found!); // UPDATE

  await userRepo.remove(found!); // DELETE
})();
```

## 언제 사용하면 좋은가?

- 대규모 시스템 (비즈니스 로직과 DB 로직을 철저히 분리하고 싶을 때)
- 테스트 코드에서 DB 의존성을 제거하고 싶을 때
- 복잡한 도메인 모델(풍부한 비즈니스 규칙 포함)을 갖는 애플리케이션

## 비교

- **Active Record**
  - 도메인 객체가 DB CRUD 직접 수행
  - 단순 CRUD 시스템에 적합
- **DAO**
  - DB 작업 전담 계층이 있음, 하지만 도메인 객체는 DB 구조와 유사할 수 있음
- **Data Mapper**
  - 도메인 객체는 DB를 전혀 모름
  - Mapper가 중간 번역을 담당
  - 가장 깔끔하지만 구현 복잡도 ↑

## 정리

- Data Mapper = "객체와 DB 사이의 번역가"
- 장점: 관심사 완전 분리, 대규모 시스템 적합
- 단점: 구현 복잡도 ↑
