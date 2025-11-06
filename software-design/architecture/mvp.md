# Architecture

# MVP

MVP는 **MVC 패턴의 변형**으로, UI와 비즈니스 로직을 더 철저하게 분리하기 위해 만들어진 패턴입니다.

- **Model**: 데이터와 비즈니스 로직
- **View**: UI, 사용자 입력을 받지만 직접 로직을 처리하지 않음
- **Presenter**: View와 Model 사이를 중개하며 모든 로직을 처리

**MVC와의 차이점**
MVC에서 **Controller**가 View와 Model 양쪽을 직접 다루는 반면, MVP에서는 **View는 단순히 Presenter의 명령을 따르는 수동적 객체**입니다.

## 기본 구현

```js
// Model
class Model {
  constructor() {
    this.data = "Hello MVP!";
  }
  getData() {
    return this.data;
  }
  setData(newData) {
    this.data = newData;
  }
}

// View (수동적 - Presenter가 시키는 대로만 함)
class View {
  show(data) {
    console.log("View Rendering:", data);
  }
}

// Presenter
class Presenter {
  constructor(model, view) {
    this.model = model;
    this.view = view;
  }
  loadData() {
    const data = this.model.getData();
    this.view.show(data);
  }
  updateData(newData) {
    this.model.setData(newData);
    this.view.show(this.model.getData());
  }
}

// 실행
const model = new Model();
const view = new View();
const presenter = new Presenter(model, view);

presenter.loadData();
presenter.updateData("MVP Pattern Example");
```

## 실제 활용 예시 (Frontend Form)

```js
// Model
class UserModel {
  constructor() {
    this.users = [];
  }
  addUser(user) {
    this.users.push(user);
  }
  getUsers() {
    return this.users;
  }
}

// View
class UserView {
  constructor() {
    this.form = document.querySelector("#userForm");
    this.input = document.querySelector("#username");
    this.list = document.querySelector("#userList");
  }

  bindAddUser(handler) {
    this.form.addEventListener("submit", (e) => {
      e.preventDefault();
      handler(this.input.value);
      this.input.value = "";
    });
  }

  render(users) {
    this.list.innerHTML = users.map(u => `<li>${u}</li>`).join("");
  }
}

// Presenter
class UserPresenter {
  constructor(model, view) {
    this.model = model;
    this.view = view;

    this.view.bindAddUser(this.handleAddUser.bind(this));
    this.view.render(this.model.getUsers());
  }

  handleAddUser(user) {
    this.model.addUser(user);
    this.view.render(this.model.getUsers());
  }
}

// 실행
const app = new UserPresenter(new UserModel(), new UserView());
```

## 언제 사용하면 좋은가?

- UI(View)가 단순해야 하고, 로직을 Presenter가 전담해야 할 때
- 테스트 용이성을 높이고 싶을 때 (Presenter는 단위 테스트가 쉬움)
- 복잡한 사용자 입력 이벤트를 관리해야 할 때

## 정리

- **장점**: View는 단순 표시 역할만 하므로 테스트 용이성 ↑, 유지보수성 ↑
- **단점**: Presenter에 로직이 집중되어 코드가 비대해질 수 있음
- **사용 예시**: Android 앱 구조, 복잡한 입력 처리가 많은 UI
