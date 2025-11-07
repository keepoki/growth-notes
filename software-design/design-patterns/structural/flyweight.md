#DesignPattern

# Flyweight

대량의 객체를 만들 때, **공유할 수 있는 부분(내부 상태)** 은 공유하고,  **변하는 부분(외부 상태)** 만 개별적으로 관리해서  메모리 사용을 최소화하는 패턴입니다.

- **내부 상태 (Intrinsic State)** → 변하지 않고 공유 가능한 것
- **외부 상태 (Extrinsic State)** → 객체마다 달라지는 것

## 기본 구현

```js
// Flyweight (공유 가능한 부분)
class Character {
  constructor(symbol, font) {
    this.symbol = symbol;
    this.font = font; // 공유 가능
  }

  draw(position) {
    console.log(`${this.symbol} (${this.font}) at ${position}`);
  }
}

// Flyweight Factory
class CharacterFactory {
  constructor() {
    this.characters = {};
  }

  getCharacter(symbol, font) {
    const key = symbol + font;
    if (!this.characters[key]) {
      this.characters[key] = new Character(symbol, font);
    }
    return this.characters[key];
  }
}

// 사용
const factory = new CharacterFactory();
const c1 = factory.getCharacter("A", "Arial");
const c2 = factory.getCharacter("A", "Arial");

console.log(c1 === c2); // true → 같은 객체 공유

c1.draw(1);
c2.draw(2);
```

## 실제 활용 코드 예시

### 게임 캐릭터 스킨

```js
// 공유할 데이터 (스킨)
class Sprite {
  constructor(image) {
    this.image = image; // 무거운 데이터
  }
  draw(x, y) {
    console.log(`Draw ${this.image} at (${x}, ${y})`);
  }
}

class SpriteFactory {
  constructor() {
    this.sprites = {};
  }
  getSprite(image) {
    if (!this.sprites[image]) {
      this.sprites[image] = new Sprite(image);
    }
    return this.sprites[image];
  }
}

// 사용
const spriteFactory = new SpriteFactory();
const orc1 = spriteFactory.getSprite("orc.png");
const orc2 = spriteFactory.getSprite("orc.png");

orc1.draw(10, 20);
orc2.draw(30, 40);

console.log(orc1 === orc2); // true → 같은 스킨 공유
```

### 웹에서 아이콘 관리 (React 느낌)

```js
class Icon {
  constructor(type) {
    this.type = type; // 공유 가능
  }
  render(x, y) {
    console.log(`Render ${this.type} icon at (${x}, ${y})`);
  }
}

class IconFactory {
  constructor() {
    this.icons = {};
  }
  getIcon(type) {
    if (!this.icons[type]) {
      this.icons[type] = new Icon(type);
    }
    return this.icons[type];
  }
}

// 사용
const iconFactory = new IconFactory();
const saveIcon1 = iconFactory.getIcon("save");
const saveIcon2 = iconFactory.getIcon("save");

saveIcon1.render(100, 200);
saveIcon2.render(300, 400);

console.log(saveIcon1 === saveIcon2); // true
```

## 언제 사용하면 좋은가?

- **대량의 유사한 객체가 필요할 때**  
    → 글자 렌더링, 게임 캐릭터, 아이콘 표시
- **객체가 무겁고, 공유 가능한 상태가 많을 때**  
    → 이미지, 글꼴, 데이터 캐시
- **메모리 최적화가 중요한 경우**

## 정리

- Flyweight는 **공유 가능한 객체를 캐싱해 재사용**하는 패턴.
- 장점: 메모리 절약, 성능 개선.
- 단점: 외부 상태를 잘 관리하지 않으면 코드가 복잡해질 수 있음.
