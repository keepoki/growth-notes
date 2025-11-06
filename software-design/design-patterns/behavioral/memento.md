# DesignPattern

# Memento

객체의 상태를 저장해두었다가 필요할 때 다시 복원할 수 있게 하는 패턴입니다.
→ "Undo(되돌리기)" 기능이나 "히스토리 관리"에 자주 사용됩니다.

## 기본 구현

```js
// Memento: 상태를 저장하는 객체
class Memento {
  constructor(state) {
    this.state = state;
  }

  getState() {
    return this.state;
  }
}

// Originator: 상태를 갖고 있는 객체
class Originator {
  constructor() {
    this.state = "";
  }

  setState(state) {
    console.log(`상태 변경: ${state}`);
    this.state = state;
  }

  save() {
    return new Memento(this.state);
  }

  restore(memento) {
    this.state = memento.getState();
    console.log(`상태 복원: ${this.state}`);
  }
}

// Caretaker: Memento를 관리
class Caretaker {
  constructor() {
    this.history = [];
  }

  add(memento) {
    this.history.push(memento);
  }

  get(index) {
    return this.history[index];
  }
}

// 사용
const originator = new Originator();
const caretaker = new Caretaker();

originator.setState("State #1");
caretaker.add(originator.save());

originator.setState("State #2");
caretaker.add(originator.save());

originator.setState("State #3");
console.log("현재 상태:", originator.state);

originator.restore(caretaker.get(0)); // State #1로 복원
```

## 실제 활용 코드 예시

### 텍스트 에디터 Undo 기능

```js
class TextEditor {
  constructor() {
    this.content = "";
  }

  type(words) {
    this.content += words;
  }

  save() {
    return new Memento(this.content);
  }

  restore(memento) {
    this.content = memento.getState();
  }

  getContent() {
    return this.content;
  }
}

class EditorHistory {
  constructor() {
    this.history = [];
  }

  push(memento) {
    this.history.push(memento);
  }

  pop() {
    return this.history.pop();
  }
}

// 사용
const editor = new TextEditor();
const history = new EditorHistory();

editor.type("Hello ");
history.push(editor.save());

editor.type("World!");
history.push(editor.save());

console.log(editor.getContent()); // Hello World!

editor.restore(history.pop()); // Undo
console.log(editor.getContent()); // Hello 
```

### 게임 상태 저장/로드

```js
class Game {
  constructor(level, hp) {
    this.level = level;
    this.hp = hp;
  }

  save() {
    return new Memento({ level: this.level, hp: this.hp });
  }

  restore(memento) {
    const state = memento.getState();
    this.level = state.level;
    this.hp = state.hp;
  }

  showStatus() {
    console.log(`레벨: ${this.level}, HP: ${this.hp}`);
  }
}

// 사용
const game = new Game(1, 100);

game.showStatus(); // 레벨: 1, HP: 100
const checkpoint = game.save();

game.level = 2;
game.hp = 50;
game.showStatus(); // 레벨: 2, HP: 50

game.restore(checkpoint);
game.showStatus(); // 레벨: 1, HP: 100
```

## 언제 사용하면 좋은가?

- **Undo / Redo 기능**이 필요한 경우 (텍스트 편집기, 그래픽 편집기)
- **게임 상태 저장/로드** 같은 기능
- 특정 시점의 **스냅샷을 보관했다가 복원**해야 하는 경우

## 정리

- Memento는 "상태 스냅샷 저장/복원" 패턴입니다.
- Originator(상태 보관 객체) ↔ Memento(스냅샷) ↔ Caretaker(히스토리 관리자) 구조입니다.
- 단점: 상태가 많거나 크면 메모리 사용량이 커집니다.
