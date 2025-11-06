# DesignPattern

# Abstract Factory

관련 있는 객체들을 **하나의 “제품군(product family)”** 으로 묶어서 생성하는 팩토리를 제공하는 패턴입니다.
클라이언트는 구체적인 클래스(`new`)를 몰라도, **추상적인 팩토리 인터페이스**를 통해 관련 객체들을 일관성 있게 생성할 수 있습니다.

## 기본 구현

```js
// 추상 팩토리 인터페이스
class UIFactory {
  createButton() {}
  createCheckbox() {}
}

// 구체 팩토리 (Mac 스타일 UI)
class MacUIFactory extends UIFactory {
  createButton() {
    return new MacButton();
  }
  createCheckbox() {
    return new MacCheckbox();
  }
}

// 구체 팩토리 (Windows 스타일 UI)
class WindowsUIFactory extends UIFactory {
  createButton() {
    return new WindowsButton();
  }
  createCheckbox() {
    return new WindowsCheckbox();
  }
}

// 제품군
class MacButton {
  render() {
    return "Rendering Mac Button 🍎";
  }
}
class MacCheckbox {
  render() {
    return "Rendering Mac Checkbox 🍎";
  }
}
class WindowsButton {
  render() {
    return "Rendering Windows Button 🪟";
  }
}
class WindowsCheckbox {
  render() {
    return "Rendering Windows Checkbox 🪟";
  }
}

// 클라이언트 코드
function app(factory) {
  const button = factory.createButton();
  const checkbox = factory.createCheckbox();

  console.log(button.render());
  console.log(checkbox.render());
}

// 실행
app(new MacUIFactory());
// Rendering Mac Button 🍎
// Rendering Mac Checkbox 🍎

app(new WindowsUIFactory());
// Rendering Windows Button 🪟
// Rendering Windows Checkbox 🪟
```

## 실제 활용 코드 예시

### 다국어 번역 라이브러리 (Locale Factory)

```js
class EnLocaleFactory {
  createDateFormatter() {
    return new Intl.DateTimeFormat("en-US");
  }
  createCurrencyFormatter() {
    return new Intl.NumberFormat("en-US", { style: "currency", currency: "USD" });
  }
}

class KoLocaleFactory {
  createDateFormatter() {
    return new Intl.DateTimeFormat("ko-KR");
  }
  createCurrencyFormatter() {
    return new Intl.NumberFormat("ko-KR", { style: "currency", currency: "KRW" });
  }
}

// 클라이언트
function printSample(factory) {
  const dateFormatter = factory.createDateFormatter();
  const currencyFormatter = factory.createCurrencyFormatter();

  console.log(dateFormatter.format(new Date())); // 날짜 현지화
  console.log(currencyFormatter.format(1234567)); // 통화 현지화
}

printSample(new EnLocaleFactory());
printSample(new KoLocaleFactory());
```

### DB 드라이버 팩토리

```js
class MySQLFactory {
  createConnection() {
    return { connect: () => "Connected to MySQL" };
  }
  createQueryBuilder() {
    return { build: () => "MySQL Query Built" };
  }
}

class MongoFactory {
  createConnection() {
    return { connect: () => "Connected to MongoDB" };
  }
  createQueryBuilder() {
    return { build: () => "MongoDB Query Built" };
  }
}

function runDB(factory) {
  const conn = factory.createConnection();
  const qb = factory.createQueryBuilder();

  console.log(conn.connect());
  console.log(qb.build());
}

runDB(new MySQLFactory());
runDB(new MongoFactory());
```

## 언제 사용하면 좋은가?

- **서로 관련된 객체들을 일관성 있게 생성해야 할 때**  
    → 예: UI 테마(버튼, 체크박스, 입력창 등), 다국어 지원(날짜/통화 포맷), DB 드라이버
- **클라이언트 코드가 구체적인 구현체에 의존하지 않고, 추상화된 인터페이스만 사용해야 할 때**
- **제품군이 확장될 가능성이 있을 때**

## 정리

- `Factory Method`는 **하나의 객체 생성 로직**에 초점을 두고,
- `Abstract Factory`는 **서로 관련된 객체들의 집합(제품군)을** 생성하는 데 초점이 있어요.
