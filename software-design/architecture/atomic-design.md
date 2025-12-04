---
tags:
  - Architecture
title: Atomic Design
---
### 아토믹 디자인(Atomic Design)이란?

**아토믹 디자인**은 UI(사용자 인터페이스)를 체계적으로 만들기 위한 디자인 방법론입니다. 화학에서 모든 물질이 **원자(Atoms)** 로 이루어져 있고, 이 원자들이 모여 **분자(Molecules)** 가 되고, 분자들이 모여 더 큰 **유기체(Organisms)** 가 되는 것처럼, UI도 가장 작은 단위부터 조립해 나가는 방식입니다.

이 패턴은 총 5가지 단계로 구성됩니다.

#### 1. 원자 (Atoms) ⚛️
- UI를 구성하는 가장 작은 기본 단위입니다. 더 이상 쪼갤 수 없는 요소들이죠.
- **React 예시:** `<button>`, `<input>`, `<label>`, 텍스트 스타일 등
- 이 자체로는 큰 의미가 없지만, 모든 UI의 시작점이 됩니다.

```jsx
// 예시: Button Atom
function Button({ label }) {
  return <button>{label}</button>;
}
```

#### 2. 분자 (Molecules) 🧪
- 여러 개의 원자들이 모여 하나의 작은 기능을 수행하는 단위입니다.
- **React 예시:** 검색창을 만들 때, `<label>`, `<input>`, `<button>` 원자를 조합하여 하나의 '검색 폼' 분자를 만드는 것.
- 이제부터는 구체적인 목적을 가진 컴포넌트가 됩니다.

```jsx
// 예시: SearchForm Molecule
import Label from './atoms/Label';
import Input from './atoms/Input';
import Button from './atoms/Button';

function SearchForm() {
  return (
    <div>
      <Label>검색</Label>
      <Input placeholder="검색어를 입력하세요" />
      <Button>찾기</Button>
    </div>
  );
}
```

#### 3. 유기체 (Organisms) 🧬
- 분자(Molecules)나 원자(Atoms)들이 모여서 만들어진 더 크고 복잡한 UI 섹션입니다.
- **React 예시:** 웹사이트 상단의 '헤더(Header)' 영역. 여기에는 로고(원자), 메뉴(분자), 검색 폼(분자) 등이 포함될 수 있습니다.
- 유기체는 독립적으로도 하나의 완전한 섹션 역할을 할 수 있습니다.

```jsx
// 예시: Header Organism
import Logo from './atoms/Logo';
import Navigation from './molecules/Navigation';
import SearchForm from './molecules/SearchForm';

function Header() {
  return (
    <header>
      <Logo />
      <Navigation />
      <SearchForm />
    </header>
  );
}
```

#### 4. 템플릿 (Templates) 📄
- 여러 유기체와 분자들이 모여 페이지의 전체적인 '레이아웃'과 '구조'를 보여주는 단계입니다.
- 실제 데이터나 콘텐츠는 없고, "여기에 헤더가 오고, 여기엔 본문이, 여기엔 푸터가 온다"와 같이 뼈대만 잡아놓은 상태입니다.
- **React 예시:** 블로그 게시물 페이지의 레이아웃을 정의한 `PostPageTemplate` 컴포넌트.

#### 5. 페이지 (Pages) 🖥️
- 템플릿에 실제 콘텐츠(텍스트, 이미지 등)를 채워 넣어 사용자에게 보여지는 최종 완성본입니다.
- **React 예시:** `PostPageTemplate`에 '오늘의 일기'라는 제목과 실제 내용을 넣어 완성된 `TodayDiaryPage`.

---

### 👦 '레고' 비유!

아토믹 디자인을 레고로 자동차를 만드는 과정에 비유해 볼게요.

- **1. 원자 (Atoms) = 레고 브릭 하나하나** 🧱
  - 가장 작은 네모 브릭, 동그란 브릭, 투명한 브릭 하나하나를 말해요. 이 브릭 하나만으로는 아무것도 아니지만, 모든 것의 시작이죠.

- **2. 분자 (Molecules) = 바퀴 조립 부품** 🚗
  - 타이어 브릭, 휠 브릭, 축 브릭을 합쳐서 '바퀴' 하나를 만들었어요. 이제 '굴러간다'는 작은 기능을 가진 부품이 되었죠.

- **3. 유기체 (Organisms) = 자동차의 큰 덩어리들** 🚙
  - 바퀴 4개(분자)와 차체 밑판(분자)을 조립해서 '자동차 하부'를 만들었어요. 또, 운전석, 핸들, 의자 등을 조립해서 '운전석 모듈'을 만들었죠. 이렇게 각각 독립적인 기능을 하는 큰 덩어리가 유기체예요.

- **4. 템플릿 (Templates) = 자동차 설계도** 📜
  - 레고 설명서를 보면, "여기에 자동차 하부를 놓고, 그 위에 운전석 모듈을 얹고, 앞에는 엔진을, 뒤에는 트렁크를 붙이세요"라고 그림으로 구조만 나와 있죠? 이게 바로 템플릿이에요. 아직 완성되진 않았지만 전체적인 모양을 알 수 있어요.

- **5. 페이지 (Pages) = 완성된 레고 자동차!** 🏎️
  - 설계도(템플릿)에 따라 모든 부품(유기체)을 제자리에 끼우고, 스티커까지 붙여서 마침내 멋진 빨간색 스포츠카가 완성되었어요! 이게 바로 사용자가 보게 되는 최종 페이지입니다.

이렇게 아토믹 디자인을 사용하면, 작은 부품(Atoms)부터 체계적으로 만들어 나가기 때문에 **재사용하기 쉽고, 유지보수가 편리하며, 전체적인 디자인의 일관성을 지키기 좋다**는 장점이 있습니다. React의 컴포넌트 기반 개발 방식과 아주 잘 맞는 찰떡궁합 패턴이라고 할 수 있죠
