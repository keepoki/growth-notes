---
tags:
  - DesignPattern
  - Behavioral
title: Command Pattern
---

요청을 객체(Command)로 캡슐화해서, 실행하는 객체(Invoker)와 실제 작업을 수행하는 객체(Receiver)를 분리하는 패턴입니다.

## 기본 구현

```js
// Command 인터페이스
class Command {
  execute() {}
  undo() {}
}

// Receiver (실제 작업자)
class Light {
  on() { console.log("💡 불 켜짐"); }
  off() { console.log("💡 불 꺼짐"); }
}

// Concrete Commands
class LightOnCommand extends Command {
  constructor(light) {
    super();
    this.light = light;
  }
  execute() { this.light.on(); }
  undo() { this.light.off(); }
}

class LightOffCommand extends Command {
  constructor(light) {
    super();
    this.light = light;
  }
  execute() { this.light.off(); }
  undo() { this.light.on(); }
}

// Invoker (명령 실행자)
class RemoteControl {
  setCommand(command) {
    this.command = command;
  }
  pressButton() {
    this.command.execute();
  }
  pressUndo() {
    this.command.undo();
  }
}

// 사용
const light = new Light();
const lightOn = new LightOnCommand(light);
const lightOff = new LightOffCommand(light);

const remote = new RemoteControl();

remote.setCommand(lightOn);
remote.pressButton(); // 💡 불 켜짐
remote.pressUndo();   // 💡 불 꺼짐

remote.setCommand(lightOff);
remote.pressButton(); // 💡 불 꺼짐
remote.pressUndo();   // 💡 불 켜짐
```

## 실제 활용 코드 예시

### 에디터 Undo/Redo

```js
class Editor {
  constructor() {
    this.content = "";
  }
  write(text) {
    this.content += text;
  }
  getContent() {
    return this.content;
  }
}

class WriteCommand {
  constructor(editor, text) {
    this.editor = editor;
    this.text = text;
  }
  execute() {
    this.editor.write(this.text);
  }
  undo() {
    this.editor.content = this.editor.content.slice(0, -this.text.length);
  }
}

// 사용
const editor = new Editor();
const history = [];

const cmd1 = new WriteCommand(editor, "Hello ");
cmd1.execute();
history.push(cmd1);

const cmd2 = new WriteCommand(editor, "World!");
cmd2.execute();
history.push(cmd2);

console.log(editor.getContent()); // Hello World!

// Undo
history.pop().undo();
console.log(editor.getContent()); // Hello
```

### 게임 입력 처리 (키 입력 -> 명령 객체)

```js
class MoveCommand {
  constructor(player, direction) {
    this.player = player;
    this.direction = direction;
  }
  execute() {
    console.log(`${this.player} moves ${this.direction}`);
  }
}

const commands = {
  w: new MoveCommand("Player1", "up"),
  s: new MoveCommand("Player1", "down"),
};

function handleInput(key) {
  const command = commands[key];
  if (command) command.execute();
}

handleInput("w"); // Player1 moves up
handleInput("s"); // Player1 moves down
```

## 언제 사용하면 좋은가?

- **작업을 실행하는 객체와 명령을 요청하는 객체를 분리하고 싶을 때**
- **Undo / Redo 기능이 필요한 경우** (텍스트 에디터, Photoshop 등)
- **명령을 큐에 저장하거나, 로그로 기록하고 싶을 때**
- **매크로(여러 명령을 하나로 묶기)** 기능이 필요할 때

## 정리

- Command 패턴은 **요청을 객체로 만들어 저장하고 실행할 수 있게 해줍니다.**
- 실무에서는 **Undo/Redo, 요청 큐잉, 매크로 명령** 같은 곳에서 자주 쓰입니다.

## 관련 개념

- [[memento]]: 객체 상태를 저장하는 패턴 (Undo/Redo와 함께 사용)
- [[observer]]: 이벤트 기반 명령 실행
- [[chain-of-responsibility]]: 명령 처리를 연쇄적으로 전달
- [[event-driven]]: Command를 이벤트로 처리하는 아키텍처
- [[designpattern]]: 디자인 패턴 전체 개요
