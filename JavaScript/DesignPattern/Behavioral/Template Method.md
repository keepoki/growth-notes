#DesignPattern 

# Template Method

상위 클래스에서 **알고리즘의 뼈대(템플릿)를** 정의하고, 일부 단계는 하위 클래스에서 오버라이드하여 구현하는 패턴입니다.
→ 공통 로직은 상위 클래스에 두고, 세부 구현은 하위 클래스에서 맡김.

## 기본 구현

```js
class AbstractClass {
  templateMethod() {
    this.stepOne();
    this.stepTwo();
    this.stepThree();
  }

  stepOne() {
    console.log("Step 1: 공통 로직");
  }

  stepTwo() {
    throw new Error("stepTwo() must be implemented");
  }

  stepThree() {
    console.log("Step 3: 공통 로직");
  }
}

class ConcreteClassA extends AbstractClass {
  stepTwo() {
    console.log("ClassA: 구체적 구현");
  }
}

class ConcreteClassB extends AbstractClass {
  stepTwo() {
    console.log("ClassB: 다른 구현");
  }
}

// 사용
const a = new ConcreteClassA();
a.templateMethod();
// Step 1 → ClassA → Step 3

const b = new ConcreteClassB();
b.templateMethod();
// Step 1 → ClassB → Step 3
```

## 실제 활용 코드 예시

### 데이터 처리 파이프라인

```js
class DataProcessor {
  process() {
    this.loadData();
    this.transformData();
    this.saveData();
  }

  loadData() {
    console.log("데이터 로드 (공통)");
  }

  transformData() {
    throw new Error("transformData() 구현 필요");
  }

  saveData() {
    console.log("데이터 저장 (공통)");
  }
}

class CSVProcessor extends DataProcessor {
  transformData() {
    console.log("CSV 데이터를 변환");
  }
}

class JSONProcessor extends DataProcessor {
  transformData() {
    console.log("JSON 데이터를 변환");
  }
}

// 사용
const csv = new CSVProcessor();
csv.process();

const json = new JSONProcessor();
json.process();
```

### 게임 캐릭터 공격 패턴

```js
class Character {
  attack() {
    this.prepare();
    this.hit();
    this.finish();
  }

  prepare() {
    console.log("무기 준비");
  }

  hit() {
    throw new Error("hit() 구현 필요");
  }

  finish() {
    console.log("공격 마무리");
  }
}

class Warrior extends Character {
  hit() {
    console.log("⚔️ 검으로 베기!");
  }
}

class Mage extends Character {
  hit() {
    console.log("✨ 마법 발사!");
  }
}

// 사용
const warrior = new Warrior();
warrior.attack();

const mage = new Mage();
mage.attack();
```

## 언제 사용하면 좋은가?

- **알고리즘 구조는 동일하지만, 일부 단계만 다를 때**
- 코드 중복을 줄이고, 공통 부분은 상위 클래스에 모으고 싶을 때
- "후크 메서드(hook)" 제공으로 기본 동작을 바꿀 여지를 주고 싶을 때

## 정리

- Template Method는 **알고리즘 틀을 상위 클래스에서 정의**하고,  **세부 구현은 하위 클래스에서 담당**하는 패턴입니다.
- 예시: 데이터 처리 파이프라인, UI 렌더링 단계, 게임 캐릭터 행동 등.
- 단점: 상속을 강제하므로 **클래스 간 결합도↑**, 유연성↓

