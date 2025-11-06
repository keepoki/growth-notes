# Architecture

# MVVM

MVVM은 **MVC와 MVP의 진화된 형태**로, UI(View)와 로직(ViewModel)을 **데이터 바인딩(Data Binding)** 으로 연결하는 아키텍처 패턴입니다.

- **Model**: 애플리케이션 데이터와 비즈니스 로직
- **View**: UI, 사용자와 상호작용하는 화면
- **ViewModel**: View와 Model 사이에서 데이터를 가공하고, View에 자동으로 반영되도록 하는 중간 계층

특징은 **양방향 데이터 바인딩** → View에서 데이터가 바뀌면 ViewModel에 자동 반영되고, ViewModel 값이 바뀌면 View도 자동 업데이트됨.

## 기본 구현

```js
// Model
class Model {
  constructor() {
    this.text = "Hello MVVM!";
  }
}

// ViewModel
class ViewModel {
  constructor(model) {
    this.model = model;
    this.bindings = {};
  }

  bind(property, callback) {
    this.bindings[property] = callback;
    callback(this.model[property]);
  }

  set(property, value) {
    this.model[property] = value;
    if (this.bindings[property]) {
      this.bindings[property](value);
    }
  }

  get(property) {
    return this.model[property];
  }
}

// View + 실행
const model = new Model();
const viewModel = new ViewModel(model);

// View와 ViewModel 바인딩
viewModel.bind("text", (value) => {
  console.log("View Rendering:", value);
});

// 값 변경 → View 자동 업데이트
viewModel.set("text", "MVVM Pattern Example");
```

## 실제 활용 예시 (Vue)

```html
<div id="app">
  <input v-model="message" />
  <p>{{ message }}</p>
</div>

<script src="https://cdn.jsdelivr.net/npm/vue@2"></script>
<script>
  new Vue({
    el: "#app",
    data: {
      message: "Hello MVVM!"
    }
  });
</script>
```

- Vue.js, Angular, Knockout.js 등은 **MVVM 기반 프레임워크**
- 위 예제에서 `message` 값이 바뀌면 `<p>` 내용도 자동으로 업데이트됨 (양방향 데이터 바인딩).

### React 예시

```js
import React, { useState } from "react";
import ReactDOM from "react-dom/client";

// Model
class Model {
  constructor(text) {
    this.text = text;
  }
}

// ViewModel
class ViewModel {
  constructor(model) {
    this.model = model;
    this.listeners = [];
  }

  bind(listener) {
    this.listeners.push(listener);
    listener(this.model.text);
  }

  setText(newText) {
    this.model.text = newText;
    this.listeners.forEach((listener) => listener(newText));
  }

  getText() {
    return this.model.text;
  }
}

// React View
function App() {
  const [text, setText] = useState(""); // View의 상태

  // MVVM 구성
  const model = new Model("Hello React MVVM!");
  const viewModel = new ViewModel(model);

  // ViewModel -> View 바인딩
  viewModel.bind(setText);

  return (
    <div>
      <h1>MVVM in React</h1>
      <input
        value={text}
        onChange={(e) => viewModel.setText(e.target.value)} // View -> ViewModel
      />
      <p>{text}</p>
    </div>
  );
}

const root = ReactDOM.createRoot(document.getElementById("root"));
root.render(<App />);
```

## 언제 사용하면 좋은가?

- UI와 데이터의 동기화를 자동으로 하고 싶을 때
- 데이터 양이 많고, UI가 복잡하며, 사용자와 상호작용이 잦을 때
- 프론트엔드 프레임워크(Vue, Angular, React+MobX 등) 사용 시

## 정리

- **장점**: 양방향 바인딩으로 생산성 ↑, View와 로직의 결합도 ↓
- **단점**: 데이터 바인딩이 과도하면 성능 이슈 발생 가능, 디버깅 난이도 ↑
- **사용 예시**: Vue.js, Angular, Knockout.js, React(상태관리 라이브러리와 조합)
