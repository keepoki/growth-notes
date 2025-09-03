#CSS 
# Flexbox

## 1) 핵심 개념

* Flex는 **1차원 레이아웃**: 한 축(주축: main axis) 기준으로 배치.
* `flex-direction`이 주축을 결정하고, 교차축(cross axis)은 그에 직각입니다.
* 컨테이너: `display: flex` / `display: inline-flex`
* 아이템: `flex-*`, `order`, `align-self` 등으로 조절

## 2) 컨테이너(부모) 관련 속성

### `display`
* `flex | inline-flex`
* `flex`는 블록 레벨 컨테이너, `inline-flex`는 인라인 레벨(주변 텍스트와 함께 흐름).

### `flex-direction`
* `row | row-reverse | column | column-reverse`
* `row`(기본): 왼→오가 **주축**
* `column`: 위→아래가 주축
* `*-reverse`는 배치 순서 반전

### `flex-wrap`
* `nowrap | wrap | wrap-reverse`
* `nowrap`(기본): 한 줄에 강제 배치(줄바꿈 없음)
* `wrap`: 공간 부족 시 다음 줄로 이동 → 여러 줄 발생
* `wrap-reverse`: 줄 방향 반전

### `flex-flow`
* `<flex-direction> <flex-wrap>`
* 예: `flex-flow: row wrap;` (둘 합친 축약)

### `justify-content`
* `flex-start | flex-end | center | space-between | space-around | space-evenly`
* **주축**(수평/세로 depending)에서의 정렬. 공간 분배 방식.

### `align-items`
* `stretch | flex-start | flex-end | center | baseline`
* **교차축**에서의 아이템 정렬. 기본이 `stretch`라 아이템 높이를 늘려서 맞춤.

### `align-content`
* `flex-start | flex-end | center | space-between | space-around | space-evenly | stretch`
* 여러 줄(`wrap`으로 라인이 여러 개일 때) **교차축**에서 라인 전체를 정렬/간격 조절.
* **한 줄일 때는 무시**됩니다(=`align-items`가 대신).

### `gap`, `row-gap`, `column-gap`
* 아이템 간격을 지정. (이제 flex에서도 `gap` 쓰는 게 권장)
* 마진 대신 `gap`을 쓰면 레이아웃이 깔끔해집니다.

## 3) 아이템(자식) 관련 속성

### `order: <integer>`
* 렌더링 순서(시각적 순서)를 바꿉니다. 기본 `0`. 음수 또는 양수 가능.
* 주의: DOM 순서는 바뀌지 않으므로 스크린리더/키보드 포커스 순서는 영향 받을 수 있음 — 접근성 고려.

### `flex-grow: <number>` (기본 0)
* 남는 공간이 있을 때 **얼만큼 더 늘어날지** 비율로 지정.
* 예: 두 아이템에 `flex-grow: 1`과 `2`이면 나머지 공간을 1:2로 나눔.

### `flex-shrink: <number>` (기본 1)
* 공간이 부족할 때 **얼마나 줄어들지** 비율. 0이면 축소하지 않음(고정).

### `flex-basis: <length|auto>`
* **아이템의 기본 크기(기준 크기)**. 기본값 `auto`(컨텐츠/width에 따라).
* `flex-basis`가 있으면 `width`보다 우선합니다.

### `flex: <grow> <shrink> <basis>` (축약)
* 예: `flex: 1 0 200px;`
* 자주 쓰는 단축형:
  * `flex: 1;` → `1 1 0%` (아이템을 동일 비율로 나눌 때 편함)
  * `flex: auto` → `1 1 auto`
  * `flex: initial` → `0 1 auto`
  * `flex: none` → `0 0 auto`

> **팁:** `flex:1`은 `flex-basis:0%`라 컨텐츠 크기 무시하고 정확히 비율로 나눕니다. 반면 `flex: 1 1 auto`는 컨텐츠 크기를 고려해 배분합니다.

### `align-self`
* `auto | flex-start | flex-end | center | baseline | stretch`
* 해당 아이템만 교차축 정렬을 오버라이드(개별 정렬).

## 4) 동작 원리(간단한 요약)

1. 각 아이템의 **기준 크기**(flex-basis 또는 content/width) 계산
2. 컨테이너 크기에서 모든 기준 크기 합을 뺀 **여유 공간(free space)** 계산
3. free space > 0 → `flex-grow` 비율로 공간 분배
   free space < 0 → `flex-shrink` 비율로 축소 (사실 실제 계산은 `flex-shrink * base-size` 비례 연산)
   (사실 규칙이 더 복잡하지만 실무에선 위 개념으로 충분)

## 5) 실무에서 자주 쓰는 패턴 & 예제

### 가. 중앙 정렬 (수평+수직)

```css
.container {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100vh;
}
```

### 나. 네비 바: 좌우 끝 아이템 고정, 가운데 유동

```html
<div class="nav">
  <div class="logo">Logo</div>
  <div class="menu">메뉴들</div>
  <div class="profile">프로필</div>
</div>
```

```css
.nav { display:flex; align-items:center; gap:16px; }
.logo{ }
.menu { margin: 0 auto; } /* 가운데로 */
```

혹은 `justify-content: space-between;`를 쓰는 방법도 있음.

### 다. 동일 너비 컬럼 (내용 무시, 균등 분배)

```css
.col { flex: 1; } /* 각 .col이 동일 너비로 */
```

### 라. 내용이 길어서 넘칠 때: `min-width:0` 트릭

```css
.flex-item { min-width: 0; } 
/* 자식 내부에 긴 문장/long word 때문에 줄이 안 될 때 이거 안 적어주면 overflow 발생 */
```

## 6) 실무 꿀팁 / 주의사항

* **`gap`을 쓰세요**: margin으로 간격 만들면 마지막 아이템 정렬이 꼬이기 쉽습니다.
* **`align-content`는 wrap(다중 라인)일 때만 의미**: 한 줄이면 무시됨.
* **`order`는 시각적 순서만 바꿉니다**. 스크린리더/탭 순서 고려해서 최소 사용.
* **`flex-basis`가 `auto`면 `width`(또는 콘텐츠 크기)를 참조**합니다. 규칙적으로 크기를 제어하려면 `flex-basis`를 명시하세요.
* \*\*이미지/컨텐츠가 넘칠 때 `min-width:0` / `min-height:0`\*\*으로 허용해주면 예상치 못한 overflow 줄일 수 있음.
* **반응형에서 `flex-wrap` + `gap` 조합**은 간단한 카드 레이아웃에 아주 편합니다.
* Flex는 1차원이라 **복잡한 2차원 레이아웃은 Grid가 더 적합**합니다. (대신 flex는 더 직관적/간단할 때 유리)

## 7) 빠른 요약표 (핵심만)

* 컨테이너: `display`, `flex-direction`, `flex-wrap`, `justify-content`, `align-items`, `align-content`, `gap`
* 아이템: `order`, `flex-grow`, `flex-shrink`, `flex-basis`, `flex`(축약), `align-self`
* 자주 쓰는 조합:
  * 중앙정렬: `justify-content:center; align-items:center;`
  * 균등 분배: `.item { flex:1 }`
  * 라인 나눔: `flex-wrap:wrap; gap:xx;`

## Flex vs Grid 비교 정리
| 항목     | Flexbox             | Grid                   |
| ------ | ------------------- | ---------------------- |
| 레이아웃   | 1차원 (한 줄 또는 한 열)    | 2차원 (행 + 열)            |
| 적합한 상황 | 버튼 그룹, 네비게이션, 간단 정렬 | 전체 페이지 레이아웃, 갤러리, 대시보드 |
