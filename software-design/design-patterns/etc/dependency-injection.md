#DesignPattern

# Dependency Injection

객체가 스스로 의존성을 생성하지 않고,  **외부에서 필요한 의존성을 주입받는 패턴**입니다.

- Service Locator는 "찾아온다(get)",
- DI는 "외부에서 넣어준다(inject)" 가 차이점.

## 기본 구현

```js
// 의존성: Logger
class Logger {
  log(message) {
    console.log("[LOG]", message);
  }
}

// 의존성을 직접 만들면 강한 결합
class UserServiceTight {
  constructor() {
    this.logger = new Logger(); // ❌ 직접 생성
  }
  getUser(id) {
    this.logger.log(`사용자 ${id} 조회`);
    return { id, name: "경준" };
  }
}

// DI를 적용한 경우
class UserService {
  constructor(logger) {
    this.logger = logger; // ✅ 외부에서 주입
  }
  getUser(id) {
    this.logger.log(`사용자 ${id} 조회`);
    return { id, name: "경준" };
  }
}

// 사용
const logger = new Logger();
const userService = new UserService(logger); // 주입
console.log(userService.getUser(1));
```

## 실제 활용 코드 예시

### Express 라우터에서 서비스 주입

```js
// services/logger.js
export class Logger {
  log(msg) {
    console.log("[LOG]", msg);
  }
}

// routes/userRoutes.js
export function createUserRoutes(userService) {
  return (req, res) => {
    const user = userService.getUser(req.params.id);
    res.json(user);
  };
}

// app.js
import express from "express";
import { Logger } from "./services/logger.js";
import { createUserRoutes } from "./routes/userRoutes.js";

class UserService {
  constructor(logger) {
    this.logger = logger;
  }
  getUser(id) {
    this.logger.log(`Fetching user ${id}`);
    return { id, name: "경준" };
  }
}

const app = express();
const logger = new Logger();
const userService = new UserService(logger);

app.get("/user/:id", createUserRoutes(userService));
```

### React 컴포넌트에서 Context 기반 DI

```js
import React, { createContext, useContext } from "react";

const LoggerContext = createContext();

function LoggerProvider({ children }) {
  const logger = { log: (msg) => console.log("[LOG]", msg) };
  return (
    <LoggerContext.Provider value={logger}>{children}</LoggerContext.Provider>
  );
}

function UserProfile() {
  const logger = useContext(LoggerContext);
  logger.log("UserProfile 렌더링됨");
  return <div>사용자 프로필</div>;
}

// App
function App() {
  return (
    <LoggerProvider>
      <UserProfile />
    </LoggerProvider>
  );
}
```

## 언제 사용하면 좋은가?

- **테스트 용이성**: Mock 객체를 쉽게 주입 가능 → 단위 테스트 편리
- **결합도 낮춤**: 클래스/모듈이 구체적인 의존성에 묶이지 않음
- **대규모 애플리케이션**: 서비스가 많아지고 의존성이 복잡할 때 DI 컨테이너 사용

## 정리

- Service Locator = "필요하면 중앙 저장소에서 꺼내옴"
- DI = "외부에서 의존성을 넣어줌"
- 현대 프레임워크(Angular, NestJS, Spring 등)는 DI 컨테이너를 내장
