# DesignPattern

# DAO (Data Access object)

- 데이터 접근 로직(DB CRUD)을 **전담하는 객체(DAO 클래스)를** 따로 두고, 비즈니스 로직에서는 DAO를 통해서만 DB를 접근하도록 하는 패턴입니다.
- **데이터 접근 계층(Data Access Layer, DAL)을** 명확히 분리

서비스 로직은 DAO를 호출만 하고, 실제 DB 작업은 DAO가 담당

## 기본 구현

```js
// DAO 클래스
class UserDAO {
  constructor(db) {
    this.db = db;
  }

  create(user) {
    this.db.push(user);
    console.log(`💾 저장됨: ${user.name}`);
  }

  findById(id) {
    return this.db.find(u => u.id === id);
  }

  update(user) {
    const idx = this.db.findIndex(u => u.id === user.id);
    if (idx > -1) {
      this.db[idx] = user;
      console.log(`✏️ 수정됨: ${user.name}`);
    }
  }

  delete(id) {
    const idx = this.db.findIndex(u => u.id === id);
    if (idx > -1) {
      const removed = this.db.splice(idx, 1);
      console.log(`🗑 삭제됨: ${removed[0].name}`);
    }
  }
}

// 사용 예시
const db = [];
const userDAO = new UserDAO(db);

userDAO.create({ id: 1, name: "Alice" });
userDAO.create({ id: 2, name: "Bob" });

const alice = userDAO.findById(1);
console.log(alice);

alice.name = "Alice Cooper";
userDAO.update(alice);

userDAO.delete(2);
console.log(db);
```

## 실제 활용 예시 (Node.js + MongoDB)

```js
// user.dao.js
const { ObjectId } = require("mongodb");

class UserDAO {
  constructor(collection) {
    this.collection = collection;
  }

  async create(user) {
    return await this.collection.insertOne(user);
  }

  async findById(id) {
    return await this.collection.findOne({ _id: new ObjectId(id) });
  }

  async update(id, data) {
    return await this.collection.updateOne(
      { _id: new ObjectId(id) },
      { $set: data }
    );
  }

  async delete(id) {
    return await this.collection.deleteOne({ _id: new ObjectId(id) });
  }
}

module.exports = UserDAO;

// service.js
const UserDAO = require("./user.dao");

async function userService(db) {
  const userDAO = new UserDAO(db.collection("users"));

  const alice = await userDAO.create({ name: "Alice" });
  console.log("저장된 사용자:", alice.insertedId);

  const found = await userDAO.findById(alice.insertedId);
  console.log("조회:", found);
}
```

## 언제 사용하면 좋은가?

- 데이터베이스 관련 코드가 비즈니스 로직과 섞이는 걸 피하고 싶을 때
- 여러 데이터 소스(DB, 캐시, API 등)를 추상화해서 교체 가능성을 높이고 싶을 때
- 테스트 용이성 확보 (DAO를 Mock 객체로 교체 가능)

## 비교

- **Active Record**
  - 객체가 스스로 DB CRUD 수행
  - 직관적, 빠른 개발
  - 규모 커지면 로직 혼재
- **DAO**
  - 데이터 접근 전담 계층을 따로 둠
  - 관심사 분리, 테스트 용이
  - 코드량 증가

## 정리

- DAO = "DB 작업을 전담하는 계층"
- 장점: 관심사 분리, 테스트/유지보수 용이
- 단점: 코드가 장황해짐
