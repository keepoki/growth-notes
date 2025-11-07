#DesignPattern

# Active Record

- 데이터베이스 테이블(Row)을 **객체(클래스 인스턴스)로** 표현하고, 그 객체가 **자신의 CRUD(Create, Read, Update, Delete) 기능을 직접 수행**하는 패턴입니다.
- 즉, **데이터와 동작이 같은 클래스에 결합**됩니다.

마치 "학생 객체가 스스로 DB에서 저장·수정·삭제를 담당"하는 느낌

## 기본 구현

```js
class User {
  constructor(id, name) {
    this.id = id;
    this.name = name;
  }

  save(db) {
    db.push(this);
    console.log(`💾 저장됨: ${this.name}`);
  }

  update(db, newName) {
    this.name = newName;
    console.log(`✏️ 수정됨: ${this.name}`);
  }

  delete(db) {
    const index = db.findIndex(u => u.id === this.id);
    if (index > -1) {
      db.splice(index, 1);
      console.log(`🗑 삭제됨: ${this.name}`);
    }
  }

  static findById(db, id) {
    return db.find(u => u.id === id);
  }
}

// 사용 예시
const db = [];
const alice = new User(1, "Alice");
alice.save(db);

const bob = new User(2, "Bob");
bob.save(db);

alice.update(db, "Alice Cooper");
bob.delete(db);

console.log(db); // [ User { id: 1, name: 'Alice Cooper' } ]
```

## 실제 활용 예시 (Node.js + Sequelize ORM)

```js
const { Sequelize, DataTypes, Model } = require("sequelize");
const sequelize = new Sequelize("sqlite::memory:");

class User extends Model {}
User.init(
  {
    name: DataTypes.STRING,
  },
  { sequelize, modelName: "User" }
);

(async () => {
  await sequelize.sync();

  const alice = await User.create({ name: "Alice" }); // INSERT
  console.log(alice.name); // Alice

  alice.name = "Alice Cooper";
  await alice.save(); // UPDATE

  const found = await User.findByPk(1); // SELECT
  console.log(found.name); // Alice Cooper

  await alice.destroy(); // DELETE
})();
```

## 언제 사용하면 좋은가?

- 단순한 CRUD 중심의 애플리케이션
- ORM을 사용할 때 가장 직관적이고 개발 속도가 빠름
- Rails, Laravel(Eloquent), Sequelize 등에서 기본적으로 채택

## 비교

- **Active Record**
  - 데이터와 DB 접근 로직이 한 클래스에 있음
  - 코드 직관적, 빠른 개발 가능
  - 대규모 시스템에서는 복잡해질 수 있음 (비즈니스 로직과 DB 로직 혼재)
- **Repository + Unit of Work**
  - DB 로직과 비즈니스 로직 분리
  - 테스트 용이, 유지보수성 ↑
  - 코드가 다소 장황해짐

## 정리

- Active Record = "객체가 자기 데이터를 직접 DB에 저장/조회/삭제"
- 장점: 직관적, 빠른 개발
- 단점: 규모가 커지면 관심사 분리 어려움
