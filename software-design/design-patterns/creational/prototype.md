#DesignPattern

# Prototype

기존 객체(프로토타입)를 복제(clone)해서 새로운 객체를 생성하는 패턴입니다.
객체를 **직접 생성(new)** 하는 대신, **이미 존재하는 객체를 복제**하여 성능 최적화와 구조 일관성을 유지할 수 있습니다.

## 기본 구현

```js
class Prototype {
  constructor(name, data) {
    this.name = name;
    this.data = data;
  }

  clone() {
    return new Prototype(this.name, { ...this.data });
  }
}

// 사용
const original = new Prototype("Original", { x: 10, y: 20 });
const copy = original.clone();

console.log(original); // Prototype { name: 'Original', data: { x: 10, y: 20 } }
console.log(copy);     // Prototype { name: 'Original', data: { x: 10, y: 20 } }

console.log(original === copy); // false (서로 다른 객체)
console.log(original.data === copy.data); // false (깊은 복제라 별도 객체)
```

## 실제 활용 코드 예시

### 게임 캐릭터 복제

```js
class Character {
  constructor(type, health, attack) {
    this.type = type;
    this.health = health;
    this.attack = attack;
  }

  clone() {
    return new Character(this.type, this.health, this.attack);
  }
}

// 기본 프로토타입
const warrior = new Character("Warrior", 100, 20);

// 복제해서 새로운 캐릭터 생성
const warrior1 = warrior.clone();
const warrior2 = warrior.clone();

warrior1.health = 80; // 일부 수정

console.log(warrior);  // { type: 'Warrior', health: 100, attack: 20 }
console.log(warrior1); // { type: 'Warrior', health: 80, attack: 20 }
console.log(warrior2); // { type: 'Warrior', health: 100, attack: 20 }
```

### UI 컴포넌트 템플릿 복제

```js
class Button {
  constructor(label, color) {
    this.label = label;
    this.color = color;
  }

  clone() {
    return new Button(this.label, this.color);
  }
}

// 기본 버튼 템플릿
const primaryButton = new Button("Submit", "blue");

// 복제 후 커스터마이징
const cancelButton = primaryButton.clone();
cancelButton.label = "Cancel";
cancelButton.color = "red";

console.log(primaryButton); // Button { label: 'Submit', color: 'blue' }
console.log(cancelButton);  // Button { label: 'Cancel', color: 'red' }
```

## 언제 사용하면 좋은가?

- **객체 생성 비용이 클 때**  
    → DB에서 로딩, 복잡한 초기화 과정이 필요한 경우
- **비슷한 객체를 반복적으로 생성해야 할 때**  
    → 게임 캐릭터, 문서 템플릿, UI 컴포넌트 등
- **런타임에 객체를 동적으로 복제해서 변형해야 할 때**

## 정리

- `Prototype`은 **복잡한 객체를 매번 새로 만드는 대신, 기존 객체를 복사(clone)하여 효율적으로 재사용**하는 방법이에요.
- 자바스크립트는 원래 프로토타입 기반 언어라서, 이 패턴이 다른 언어보다 더 자연스럽게 어울립니다.
