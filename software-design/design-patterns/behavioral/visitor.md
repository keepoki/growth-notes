# DesignPattern

# Visitor

객체 구조(Elements)를 변경하지 않고, 새로운 동작(Operations)을 추가할 수 있게 하는 패턴입니다.
→ 즉, **객체는 그대로 두고, 방문자(Visitor)가 행동을 정의**합니다.
→ "데이터 구조와 기능을 분리"할 때 유용합니다.

## 기본 구현

```js
// 방문자 (Visitor)
class Visitor {
  visitElementA(element) {
    console.log("ElementA 처리");
  }

  visitElementB(element) {
    console.log("ElementB 처리");
  }
}

// 요소 (Element)
class Element {
  accept(visitor) {
    throw new Error("accept() 구현 필요");
  }
}

class ElementA extends Element {
  accept(visitor) {
    visitor.visitElementA(this);
  }
}

class ElementB extends Element {
  accept(visitor) {
    visitor.visitElementB(this);
  }
}

// 사용
const elements = [new ElementA(), new ElementB()];
const visitor = new Visitor();

elements.forEach(el => el.accept(visitor));
```

## 실제 활용 코드 예시

### 문서 렌더링 (PDF, HTML, Text)

```js
class DocumentPart {
  accept(visitor) {}
}

class Text extends DocumentPart {
  constructor(content) {
    super();
    this.content = content;
  }
  accept(visitor) {
    visitor.visitText(this);
  }
}

class Image extends DocumentPart {
  constructor(src) {
    super();
    this.src = src;
  }
  accept(visitor) {
    visitor.visitImage(this);
  }
}

class HtmlExportVisitor {
  visitText(text) {
    console.log(`<p>${text.content}</p>`);
  }
  visitImage(image) {
    console.log(`<img src="${image.src}" />`);
  }
}

class PlainTextExportVisitor {
  visitText(text) {
    console.log(text.content);
  }
  visitImage(image) {
    console.log(`[이미지: ${image.src}]`);
  }
}

// 사용
const document = [
  new Text("안녕하세요"),
  new Image("logo.png"),
];

const htmlExporter = new HtmlExportVisitor();
document.forEach(part => part.accept(htmlExporter));

const textExporter = new PlainTextExportVisitor();
document.forEach(part => part.accept(textExporter));
```

### 게임 캐릭터 상호작용

```js
class Character {
  accept(visitor) {}
}

class Warrior extends Character {
  accept(visitor) {
    visitor.visitWarrior(this);
  }
}

class Mage extends Character {
  accept(visitor) {
    visitor.visitMage(this);
  }
}

class InteractionVisitor {
  visitWarrior(warrior) {
    console.log("⚔️ 전사와 악수");
  }
  visitMage(mage) {
    console.log("✨ 마법사와 대화");
  }
}

// 사용
const party = [new Warrior(), new Mage()];
const interaction = new InteractionVisitor();

party.forEach(char => char.accept(interaction));
```

## 언제 사용하면 좋은가?

- 객체 구조는 자주 변하지 않지만, **새로운 기능(연산)을 자주 추가**해야 할 때
- 데이터 구조와 기능을 분리하고 싶을 때
- 예:
  - 컴파일러(구문 트리 탐색)
  - 문서 변환기(HTML, PDF, Text 등으로 내보내기)
  - 게임 캐릭터 간 상호작용

## 정리

- Visitor 패턴은 **객체 구조를 변경하지 않고, 새로운 기능을 추가**할 수 있게 합니다.
- "이중 디스패치(Double Dispatch)" 개념 활용.
- 단점: 객체 구조 자체를 변경하면 Visitor도 다 바꿔야 해서 유지보수 비용이 큽니다.
