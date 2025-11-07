#DesignPattern

# Facade

복잡한 서브시스템(여러 클래스, 모듈)을  **간단한 인터페이스**로 감싸서 클라이언트가 쉽게 사용할 수 있도록 하는 패턴입니다. 흔히 "리모컨" 비유를 많이 합니다. (내부적으로 복잡한 전자 회로 → 버튼 몇 개만 제공)

## 기본 구현

```js
// 복잡한 서브시스템
class CPU {
  freeze() { console.log("CPU: freeze"); }
  jump(position) { console.log(`CPU: jump to ${position}`); }
  execute() { console.log("CPU: execute"); }
}

class Memory {
  load(position, data) {
    console.log(`Memory: load ${data} at ${position}`);
  }
}

class HardDrive {
  read(lba, size) {
    return `data-from-${lba}-${size}`;
  }
}

// Facade
class ComputerFacade {
  constructor() {
    this.cpu = new CPU();
    this.memory = new Memory();
    this.hd = new HardDrive();
  }

  start() {
    console.log("=== 컴퓨터 시작 ===");
    this.cpu.freeze();
    const data = this.hd.read(0, 100);
    this.memory.load(0, data);
    this.cpu.jump(0);
    this.cpu.execute();
  }
}

// 사용
const computer = new ComputerFacade();
computer.start();
```

## 실제 활용 코드 예시

### API 호출

```js
// 복잡한 fetch + 응답 처리
async function rawFetch(url, options) {
  const response = await fetch(url, options);
  if (!response.ok) throw new Error("Network error");
  return await response.json();
}

// Facade
class ApiClient {
  constructor(baseURL) {
    this.baseURL = baseURL;
  }

  getUsers() {
    return rawFetch(`${this.baseURL}/users`, { method: "GET" });
  }

  createUser(data) {
    return rawFetch(`${this.baseURL}/users`, {
      method: "POST",
      body: JSON.stringify(data),
      headers: { "Content-Type": "application/json" }
    });
  }
}

// 사용
const api = new ApiClient("https://api.example.com");
api.getUsers().then(console.log);
api.createUser({ name: "Alice" }).then(console.log);
```

### UI 라이브러리

```js
// 복잡한 DOM 조작
class DOMHelper {
  static create(tag, text, parent) {
    const el = document.createElement(tag);
    if (text) el.textContent = text;
    if (parent) parent.appendChild(el);
    return el;
  }
}

// Facade
class UI {
  static createButton(label, parent, onClick) {
    const btn = DOMHelper.create("button", label, parent);
    btn.addEventListener("click", onClick);
    return btn;
  }

  static createInput(placeholder, parent) {
    const input = DOMHelper.create("input", null, parent);
    input.placeholder = placeholder;
    return input;
  }
}

// 사용
const app = document.getElementById("app");
const input = UI.createInput("Enter text", app);
UI.createButton("Submit", app, () => alert(input.value));
```

## 언제 사용하면 좋은가?

- **복잡한 서브시스템을 단순하게 쓰고 싶을 때**  
    → 컴퓨터 부품, API 호출, DB 연결, DOM 조작
- **클라이언트 코드가 내부 구현을 알 필요 없을 때**  
    → Facade가 필요한 기능만 제공
- **시스템을 계층화하고 싶을 때**  
    → `UI Layer → Facade → Service Layer → Database`

## 정리

- `Facade`는 "**복잡한 내부 로직을 숨기고, 단순한 인터페이스만 제공**"하는 패턴이에요.
- 장점: 사용성 향상, 의존성 감소, 코드 단순화.
- 단점: 너무 많은 기능을 넣으면 "거대한 리모컨(God Facade)"이 될 수 있음.
