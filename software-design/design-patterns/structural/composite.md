#DesignPattern 

# Composite

객체들을 트리 구조(부분-전체 계층 구조)로 구성해서,  **개별 객체와 복합 객체를 동일하게 다룰 수 있도록** 하는 패턴입니다. 흔히 **디렉토리-파일 구조** 예시로 설명합니다.

## 기본 구현

```js
// Component (공통 인터페이스)
class Graphic {
  draw() {}
}

// Leaf (단일 객체)
class Circle extends Graphic {
  draw() {
    console.log("⚪ 원 그리기");
  }
}

class Square extends Graphic {
  draw() {
    console.log("⬛ 사각형 그리기");
  }
}

// Composite (복합 객체)
class GraphicGroup extends Graphic {
  constructor() {
    super();
    this.children = [];
  }

  add(child) {
    this.children.push(child);
  }

  draw() {
    this.children.forEach(child => child.draw());
  }
}

// 사용
const circle = new Circle();
const square = new Square();

const group = new GraphicGroup();
group.add(circle);
group.add(square);

group.draw();
// ⚪ 원 그리기
// ⬛ 사각형 그리기
```

## 실제 활용 코드 예시

### 파일 시스템 구조
```js
class FileSystemComponent {
  show(indent = "") {}
}

class File extends FileSystemComponent {
  constructor(name) {
    super();
    this.name = name;
  }
  show(indent = "") {
    console.log(indent + "📄 " + this.name);
  }
}

class Directory extends FileSystemComponent {
  constructor(name) {
    super();
    this.name = name;
    this.children = [];
  }
  add(component) {
    this.children.push(component);
  }
  show(indent = "") {
    console.log(indent + "📂 " + this.name);
    this.children.forEach(child => child.show(indent + "  "));
  }
}

// 사용
const root = new Directory("root");
const src = new Directory("src");
const file1 = new File("index.js");
const file2 = new File("app.js");

src.add(file1);
root.add(src);
root.add(file2);

root.show();
/*
📂 root
  📂 src
    📄 index.js
  📄 app.js
*/
```

### UI 컴포넌트 트리
```js
class UIComponent {
  render() {}
}

class Button extends UIComponent {
  render() {
    return "<button>Click Me</button>";
  }
}

class TextBox extends UIComponent {
  render() {
    return "<input type='text' />";
  }
}

class Panel extends UIComponent {
  constructor() {
    super();
    this.children = [];
  }
  add(component) {
    this.children.push(component);
  }
  render() {
    return `<div class="panel">${this.children.map(c => c.render()).join("")}</div>`;
  }
}

// 사용
const panel = new Panel();
panel.add(new Button());
panel.add(new TextBox());

console.log(panel.render());
// <div class="panel"><button>Click Me</button><input type='text' /></div>
```

## 언제 사용하면 좋은가?

- **부분-전체 계층 구조를 동일하게 다루고 싶을 때**  
    → 디렉토리-파일, UI 컴포넌트 트리, 조직도, 수학식 트리 등
- **클라이언트가 개별 객체와 복합 객체를 구분하지 않고 사용해야 할 때**  
    → `draw()` 같은 동일한 메서드로 접근 가능
- **재귀적인 구조가 자연스러운 경우**

## 정리

Composite은 **트리 구조를 편하게 관리**하기 위한 패턴으로, Leaf(단일 객체)와 Composite(복합 객체)를 같은 인터페이스로 다룰 수 있게 합니다.