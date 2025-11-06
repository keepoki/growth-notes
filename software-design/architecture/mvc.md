# Architecture

# MVC

MVC는 애플리케이션을 **Model, View, Controller** 세 부분으로 나누어 역할을 분리하는 아키텍처 패턴입니다.

- **Model**: 데이터와 비즈니스 로직을 관리
- **View**: UI, 사용자에게 보여지는 부분
- **Controller**: 사용자의 입력을 받아 Model과 View를 연결

## 기본 구현

```js
// Model
class Model {
  constructor() {
    this.data = "Hello MVC!";
  }
  getData() {
    return this.data;
  }
  setData(newData) {
    this.data = newData;
  }
}

// View
class View {
  render(data) {
    console.log("View Rendering:", data);
  }
}

// Controller
class Controller {
  constructor(model, view) {
    this.model = model;
    this.view = view;
  }
  updateData(newData) {
    this.model.setData(newData);
    this.view.render(this.model.getData());
  }
}

// 실행
const model = new Model();
const view = new View();
const controller = new Controller(model, view);

controller.updateData("MVC Pattern Example");
```

## 실제 활용 예시 (Express.js)

```js
// Model (User.js)
const users = [];
class UserModel {
  static addUser(name) {
    users.push(name);
  }
  static getUsers() {
    return users;
  }
}

// View (userView.js)
function renderUsers(users) {
  return `<ul>${users.map(u => `<li>${u}</li>`).join("")}</ul>`;
}

// Controller (userController.js)
const express = require("express");
const router = express.Router();

router.get("/users", (req, res) => {
  res.send(renderUsers(UserModel.getUsers()));
});

router.post("/users", (req, res) => {
  UserModel.addUser(req.body.name);
  res.redirect("/users");
});

module.exports = router;
```

## 언제 사용하면 좋은가?

- UI와 데이터 로직을 명확히 분리해야 할 때
- 웹 애플리케이션의 규모가 커지고, 역할 분리가 필요할 때
- 유지보수성과 확장성을 고려해야 하는 경우

## 정리

- **장점**: 역할 분리, 코드 재사용성 ↑, 유지보수 쉬움
- **단점**: 규모가 작을 경우 오히려 복잡해질 수 있음
- **사용 예시**: 웹 프레임워크 (Express.js, Ruby on Rails, ASP.NET MVC 등)
