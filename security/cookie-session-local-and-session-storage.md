#Web

# Cookie, Session, Local/Session Storage

## 1. 쿠키 (Cookie)

* **저장 위치**: 클라이언트(브라우저)에 저장 (텍스트 파일 형태)
* **용도**: 서버가 사용자를 식별하기 위해 사용 (로그인 상태 유지, 장바구니 등)
* **특징**:
  * 서버와 클라이언트가 **HTTP 요청/응답 시 자동으로 주고받음**
  * 보통 **4KB 정도 크기 제한**
  * 만료시간을 정할 수 있음 (지속 쿠키 vs 세션 쿠키)
* **보안 위험**: 브라우저에 저장되므로 탈취될 수 있음 → 중요한 정보 직접 저장하면 안 됨.

### 쿠키 예시

* "내가 어제 본 상품 목록"이 쿠키에 저장되어 있어서 다음에 접속했을 때 바로 보여줌.
* PC방에서 로그인했는데 로그아웃 안 하면, 쿠키가 남아 있어서 다른 사람이 내 계정으로 들어갈 수도 있음.

```js
// 쿠키 저장 (key=value; expires=날짜; path=경로)
document.cookie = "username=jun; expires=Fri, 31 Dec 2025 23:59:59 GMT; path=/";

// 쿠키 조회 (모든 쿠키를 문자열로 반환)
console.log(document.cookie); 
// 예시 출력: "username=jun; theme=dark"

// 쿠키 수정 (같은 key로 덮어쓰기)
document.cookie = "username=kyo; path=/";

// 쿠키 삭제 (expires를 과거로 설정)
document.cookie = "username=; expires=Thu, 01 Jan 1970 00:00:00 UTC; path=/;";
```

---

## 2. 세션 (Session)

* **저장 위치**: 서버에 저장
* **용도**: 로그인 같은 **사용자 상태 관리**
* **특징**:
  * 서버에서 사용자별로 공간(Session ID 기반)을 만들어 관리
  * 클라이언트는 **세션 ID만 쿠키에 저장**해서 서버와 연결
  * 브라우저 종료하거나 일정 시간 지나면 세션 만료
  * 서버 자원을 차지한다는 단점

### 세션 예시

* 내가 로그인하면 서버가 `Session ID = 1234`라는 방을 하나 만들어두고, 내 로그인 정보를 거기에 보관.
* 브라우저는 "나는 1234번이에요!"라고 쿠키로 알려주면, 서버는 그 방에서 내 정보를 찾아서 로그인 상태를 유지해줌.

```js
const express = require("express");
const session = require("express-session");

const app = express();
const PORT = 3000;

// 세션 미들웨어 설정
app.use(
  session({
    secret: "mySecretKey", // 세션 암호화 키 (꼭 환경변수로 관리할 것!)
    resave: false,         // 요청마다 세션 다시 저장할지 여부
    saveUninitialized: true, // 초기화되지 않은 세션도 저장할지 여부
    cookie: {
      maxAge: 1000 * 60 * 5, // 쿠키 유효기간 (5분)
      httpOnly: true,        // JS에서 접근 불가 (보안 ↑)
    },
  })
);

// 로그인 시 세션에 사용자 정보 저장
app.get("/login", (req, res) => {
  req.session.user = { username: "jun", role: "admin" };
  res.send("로그인 성공, 세션에 사용자 정보 저장됨!");
});

// 세션에서 사용자 정보 확인
app.get("/profile", (req, res) => {
  if (req.session.user) {
    res.send(`안녕하세요, ${req.session.user.username}님!`);
  } else {
    res.send("로그인 필요!");
  }
});

// 로그아웃 (세션 삭제)
app.get("/logout", (req, res) => {
  req.session.destroy((err) => {
    if (err) {
      return res.send("로그아웃 실패");
    }
    res.clearCookie("connect.sid"); // 기본 세션 쿠키 이름: connect.sid
    res.send("로그아웃 완료!");
  });
});

app.listen(PORT, () => {
  console.log(`✅ Server running at http://localhost:${PORT}`);
});
```

1. `/login` 접속 → 서버가 세션 공간을 만들고 `connect.sid`라는 세션 ID를 쿠키에 넣어서 브라우저에 전송
2. 이후 요청 시 브라우저는 `connect.sid` 쿠키를 자동 전송
3. 서버는 그 세션 ID를 보고 `req.session.user`에 있는 로그인 정보를 찾아서 유지
4. `/logout` 요청 → 세션 삭제 + 쿠키 제거

즉, **세션은 실제 데이터는 서버에 저장하고, 클라이언트(브라우저)는 세션 ID만 쿠키로 갖고 있는 구조**

---

## 3. 로컬 스토리지 (LocalStorage)

* **저장 위치**: 브라우저 (클라이언트)
* **용도**: 장기적으로 데이터를 저장 (사용자 설정, 캐시 등)
* **특징**:
  * **자동으로 서버에 전송되지 않음** (쿠키와 다름!)
  * **5MB 정도 저장 가능** (쿠키보다 훨씬 큼)
  * 브라우저를 닫아도 남아 있음 (영구 저장)
* **보안 주의**: JS로 접근 가능하므로 민감한 정보 저장하면 위험.

### 로컬 스토리지 예시

* 다크 모드/라이트 모드 설정 저장.
* 한 번 본 공지사항은 다시 안 보이게 체크 → 이 상태를 LocalStorage에 저장.

```js
// 데이터 저장
localStorage.setItem("theme", "dark");

// 데이터 조회
console.log(localStorage.getItem("theme")); // "dark"

// 데이터 수정
localStorage.setItem("theme", "light");

// 데이터 삭제
localStorage.removeItem("theme");

// 모든 데이터 삭제
localStorage.clear();
```

---

## 4. 세션 스토리지 (SessionStorage)

* **저장 위치**: 브라우저 (클라이언트)
* **용도**: 한 브라우저 탭에서만 사용하는 데이터
* **특징**:
  * **자동으로 서버에 전송되지 않음**
  * **5MB 정도 저장 가능**
  * 브라우저 탭을 닫으면 데이터 사라짐
  * 탭 간 공유 안 됨 (로컬 스토리지와 차이점)

### 세션 스토리지 예시

* 쇼핑몰 주문 페이지에서, 결제 단계까지 가는 동안만 필요한 데이터 저장.
* 새 탭 열면 다시 비어 있음 (주문 데이터 안 넘어감).

```js
// 데이터 저장
sessionStorage.setItem("cart", JSON.stringify(["apple", "banana"]));

// 데이터 조회
console.log(sessionStorage.getItem("cart")); 
// 출력: "[\"apple\",\"banana\"]"

// 데이터 수정
sessionStorage.setItem("cart", JSON.stringify(["apple", "banana", "grape"]));

// 데이터 삭제
sessionStorage.removeItem("cart");

// 모든 데이터 삭제
sessionStorage.clear();
```

---

## 비교 요약

| 구분          | 저장 위치 | 서버 전송 여부    | 만료 시점          | 저장 용량  | 대표 용도          |
| ----------- | ----- | ----------- | -------------- | ------ | -------------- |
| **쿠키**      | 브라우저  | ✅ 자동 전송     | 설정된 만료 시간      | \~4KB  | 로그인 유지, 사용자 추적 |
| **세션**      | 서버    | ✅ (세션ID 쿠키) | 브라우저 종료 / 만료시간 | 서버 메모리 | 로그인 상태 관리      |
| **로컬 스토리지** | 브라우저  | ❌           | 사용자가 지우기 전까지   | \~5MB  | 사용자 설정, 캐시     |
| **세션 스토리지** | 브라우저  | ❌           | 탭 닫을 때         | \~5MB  | 탭 내 임시 데이터     |

---

## 알기 쉽게 비유

* **쿠키** → “도서관 회원증에 적힌 내 번호” (가방에 넣어두고, 어디 가든 보여주면 됨. 하지만 분실 위험 있음)
* **세션** → “도서관에 있는 내 사물함” (회원증 번호로 찾아 들어감. 사물함은 도서관이 관리)
* **로컬 스토리지** → “내 집 책장” (내가 영구적으로 보관. 언제든 다시 꺼내봄)
* **세션 스토리지** → “내 책상 위에 잠깐 올려둔 메모” (자리 떠나면 치워짐. 옆자리랑 공유 안 됨)
