# DesignPattern

# Mutation

Singleton이 "하나의 인스턴스만 유지"한다면, **Multiton은 Key마다 하나의 인스턴스만 유지**하는 패턴입니다.
예: DB 커넥션, 로거(Logger), 설정 객체 등에서 "종류별 단일 인스턴스" 필요할 때

## 기본 구현

```js
class Logger {
  constructor(level) {
    this.level = level;
  }

  log(message) {
    console.log(`[${this.level}] ${message}`);
  }
}

class LoggerFactory {
  static instances = new Map();

  static getLogger(level) {
    if (!this.instances.has(level)) {
      this.instances.set(level, new Logger(level));
    }
    return this.instances.get(level);
  }
}

// 사용
const errorLogger = LoggerFactory.getLogger("ERROR");
const infoLogger = LoggerFactory.getLogger("INFO");

errorLogger.log("DB 연결 실패");
infoLogger.log("서버 시작됨");

// 같은 key 요청 → 동일 인스턴스 반환
console.log(errorLogger === LoggerFactory.getLogger("ERROR")); // true
```

## 실제 활용 코드 예시

### DB 커넥션 관리

```js
class DBConnection {
  constructor(database) {
    this.database = database;
    console.log(`${database} DB 연결 생성`);
  }
}

class DBConnectionPool {
  static connections = new Map();

  static getConnection(database) {
    if (!this.connections.has(database)) {
      this.connections.set(database, new DBConnection(database));
    }
    return this.connections.get(database);
  }
}

// 사용
const userDB = DBConnectionPool.getConnection("users");
const orderDB = DBConnectionPool.getConnection("orders");
const againUserDB = DBConnectionPool.getConnection("users");

console.log(userDB === againUserDB); // true
```

### 테마별 Config 객체

```js
class Config {
  constructor(theme) {
    this.theme = theme;
  }
}

class ConfigManager {
  static configs = {};

  static getConfig(theme) {
    if (!this.configs[theme]) {
      this.configs[theme] = new Config(theme);
    }
    return this.configs[theme];
  }
}

// 사용
const darkConfig = ConfigManager.getConfig("dark");
const lightConfig = ConfigManager.getConfig("light");
```

## 언제 사용하면 좋은가?

- **종류별로 하나만 있어야 하는 객체**가 있을 때
- Singleton보다 유연하게, **Key 단위로 인스턴스를 관리**해야 할 때
- 예:
  - 로그 레벨별 로거(Logger)
  - 여러 DB/캐시 연결 객체
  - 사용자별/테마별 환경 설정

## 정리

- **Singleton → 전역 1개**
- **Multiton → Key별 1개**
- 단점: 전역 상태 관리이므로 남용하면 코드 테스트/디버깅 어려워짐
