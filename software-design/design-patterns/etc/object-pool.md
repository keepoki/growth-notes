#DesignPattern 

# Object Pool

비용이 큰 객체를 매번 새로 생성하지 않고,  **재사용 가능한 객체 풀(pool)을** 만들어 관리하는 패턴입니다.
예: DB 커넥션, 스레드, 네트워크 소켓, 캐싱 가능한 대규모 객체

## 기본 구현

```js
class ObjectPool {
  constructor(createFn, size = 5) {
    this.createFn = createFn;
    this.pool = Array.from({ length: size }, () => this.createFn());
    this.available = [...this.pool];
  }

  acquire() {
    if (this.available.length === 0) {
      console.log("⚠️ 풀에 객체가 없습니다. 새로 생성합니다.");
      return this.createFn();
    }
    return this.available.pop();
  }

  release(obj) {
    this.available.push(obj);
  }
}

// 사용 예시: DB 연결 객체 풀
const pool = new ObjectPool(() => ({ id: Date.now() }), 2);

const obj1 = pool.acquire();
const obj2 = pool.acquire();
const obj3 = pool.acquire(); // 풀 부족 → 새 객체 생성

console.log(obj1, obj2, obj3);

pool.release(obj1); // 다시 풀에 반환
const obj4 = pool.acquire(); // 재사용됨
console.log(obj4);
```

## 실제 활용 코드 예시

### DB 커넥션 풀 (Express + MySQL)

```js
import mysql from "mysql2/promise";

const pool = mysql.createPool({
  host: "localhost",
  user: "root",
  password: "1234",
  database: "test",
  connectionLimit: 10, // 풀 크기
});

// 사용
async function getUsers() {
  const conn = await pool.getConnection();
  const [rows] = await conn.query("SELECT * FROM users");
  conn.release(); // 연결 반환
  return rows;
}
```

### 게임 오브젝트 풀링 (React/Canvas)

```js
class Bullet {
  constructor() {
    this.active = false;
  }
  fire(x, y) {
    this.active = true;
    this.x = x;
    this.y = y;
  }
  reset() {
    this.active = false;
  }
}

class BulletPool extends ObjectPool {
  constructor(size) {
    super(() => new Bullet(), size);
  }
}

// 게임에서 사용
const bullets = new BulletPool(10);
const b1 = bullets.acquire();
b1.fire(100, 200);
bullets.release(b1); // 재사용
```

## 언제 사용하면 좋은가?

- 객체 생성 비용이 비싸고, **자주 생성/파괴**되는 경우
- 리소스를 제한적으로 관리해야 할 때 (DB 커넥션, 네트워크 소켓, 스레드 등)
- 게임 개발에서 **총알, 이펙트, NPC**처럼 짧게 쓰이고 자주 재사용되는 오브젝트

## 정리

- **장점**: 성능 최적화, GC 부담 줄임, 리소스 효율 ↑
- **단점**: 풀 관리 로직이 복잡해질 수 있음, 객체 상태 초기화 누락 시 버그 발생
