#DesignPattern

# Chain of Responsibility

요청을 처리하는 여러 객체들이 **체인(연결 고리)** 으로 묶여 있어서, 요청이 해당 객체에서 처리되거나, **다음 객체로 넘겨지는 방식**의 패턴입니다.
→ 클라이언트는 어떤 객체가 요청을 처리하는지 몰라도 됩니다.

## 기본 구현

```js
class Handler {
  setNext(handler) {
    this.next = handler;
    return handler;
  }

  handle(request) {
    if (this.next) {
      return this.next.handle(request);
    }
    return null;
  }
}

// 구체 핸들러
class AuthHandler extends Handler {
  handle(request) {
    if (!request.user) {
      console.log("Auth 실패: 사용자 없음");
      return "401 Unauthorized";
    }
    console.log("Auth 성공");
    return super.handle(request);
  }
}

class RoleHandler extends Handler {
  handle(request) {
    if (request.user.role !== "admin") {
      console.log("권한 부족");
      return "403 Forbidden";
    }
    console.log("권한 확인 완료");
    return super.handle(request);
  }
}

class DataHandler extends Handler {
  handle(request) {
    console.log("데이터 처리 완료");
    return "200 OK";
  }
}

// 사용
const auth = new AuthHandler();
const role = new RoleHandler();
const data = new DataHandler();

auth.setNext(role).setNext(data);

console.log(auth.handle({ user: { role: "admin" } }));
// Auth 성공 → 권한 확인 완료 → 데이터 처리 완료 → "200 OK"
```

## 실제 활용 코드 예시

### Express 미들웨어 (실제 체인 구조)

```js
function authMiddleware(req, res, next) {
  if (!req.user) return res.status(401).send("Unauthorized");
  next();
}

function roleMiddleware(req, res, next) {
  if (req.user.role !== "admin") return res.status(403).send("Forbidden");
  next();
}

function dataMiddleware(req, res) {
  res.send("Data response");
}

// Express는 미들웨어 체인을 자동으로 실행
app.get("/data", authMiddleware, roleMiddleware, dataMiddleware);
```

### UI 이벤트 처리 (DOM 이벤트 버블링도 CoR 성격을 가짐)

```js
document.getElementById("child").addEventListener("click", (e) => {
  console.log("Child handler");
  // e.stopPropagation(); → 여기서 멈추면 부모로 안 올라감
});

document.getElementById("parent").addEventListener("click", () => {
  console.log("Parent handler");
});
```

## 언제 사용하면 좋은가?

- **여러 처리 단계가 순차적으로 이어지는 경우**  
    → 인증 → 권한 확인 → 데이터 처리
- **요청을 처리할 객체를 유연하게 바꿔야 하는 경우**  
    → 체인 구성을 런타임에 변경 가능
- **조건에 따라 요청을 다르게 처리해야 하는 경우**  
    → 이벤트 핸들링, 필터 파이프라인

## 정리

- Chain of Responsibility는 **요청을 처리할 기회를 여러 객체에 주는 패턴**.
- 클라이언트는 내부 구조를 몰라도 되고, 유연하게 체인을 변경할 수 있습니다.
