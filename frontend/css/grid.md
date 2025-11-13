---
tags:
  - CSS
  - Frontend
  - Layout
title: CSS Grid (그리드 레이아웃)
---

Grid는 **2차원(행+열)** 레이아웃 엔진으로, 페이지 전체 레이아웃이나 복잡한 카드/대시보드에 최적화되어 있습니다.

## 1) 기본 용어 (짧게)

- **트랙(track)**: 한 열(column) 또는 한 행(row)의 단위.
- **라인(line)**: 트랙 경계(시작/끝).
- **셀(cell)**: 행×열의 한 칸.
- **영역(area)**: 여러 셀을 합친 블록(예: `grid-template-areas`).
- **명시적(explicit) 그리드** vs **암시적(implicit) 그리드**: `grid-template-*`로 만든 영역은 명시적, 아이템이 더 들어오면 브라우저가 자동으로 만드는 트랙은 암시적.

---

## 2) 트랙(열/행) 크기 지정 방식 — 핵심 함수/단위

- `fr` : 남은 공간 비율 단위. (ex. `1fr 2fr`)
- `px`, `%`, `em` 등 고정값 가능.
- `minmax(min, max)` : 트랙 크기를 최소/최대 범위로 지정. (그리드에서 자주 쓰임)
- `min-content` / `max-content` / `auto` : 콘텐츠 기반 크기 결정.
- `fit-content()` : 최대값을 넘어가지 않도록 하면서 콘텐츠에 맞춤.
- `repeat(count, pattern)` : 같은 패턴 반복(짧게 쓰기). (`repeat(auto-fit, ...)` 같은 패턴은 반응형에 자주 사용).

## 3) Grid 컨테이너(부모) 주요 속성 — 무엇을 어떻게 제어하는가

- `display: grid | inline-grid`
- `grid-template-columns`, `grid-template-rows`
  - 예: `grid-template-columns: 200px 1fr 2fr;` 또는 `repeat(3, 1fr)`
- `grid-template-areas`
  - 영역 이름으로 레이아웃을 가독성 있게 선언 가능.
- `grid-auto-rows`, `grid-auto-columns`
  - 암시적 트랙의 크기 지정.
- `grid-auto-flow: row | column | row dense | column dense`
  - **자동 배치 방향**을 제어(기본은 `row`)
  - `dense`는 빈틈을 채우려 아이템 순서를 바꿔 촘촘히 채우려 시도 → 시맨틱/탭 순서 주의.
- `gap | row-gap | column-gap` (추천)
  - 아이템 간격을 깔끔히 제어. `margin` 대신 `gap`을 쓰는 것이 일반적.
- 정렬 관련:
  - `justify-items` (아이템을 **열 방향**에서 개별 정렬),
  - `align-items` (행 방향 정렬),
  - `justify-content` / `align-content` (그리드 전체 블록 정렬 — 컨테이너 내부 여백 분배).
  - `place-items` / `place-content` = 위 두 속성의 축약형.

## 4) Grid 아이템(자식) 주요 속성 — 아이템을 어디에 둘지

- **라인 기반 배치**
  - `grid-column-start`, `grid-column-end`, `grid-row-start`, `grid-row-end`
  - 축약: `grid-column: 1 / 3;` → 1번 라인부터 3번 라인까지(즉 2개의 컬럼 차지).
  - `span` 사용: `grid-column: span 2;` (현재 위치에서 2칸 차지)
- **영역 이름 사용**
  - `grid-area: header;` (부모의 `grid-template-areas`에서 선언한 이름과 매칭)
- **정렬 개별 제어**
  - `justify-self`, `align-self`, `place-self`
- `order`는 Grid에 없으므로 DOM 순서 제어는 디자인보다 접근성에 영향을 줍니다(주의).

## 5) 자동 배치(autoplacement) 규칙 — 브라우저가 아이템을 놓는 법

- 명시적 위치가 없으면 **행(row) 우선**으로 차례대로 빈 셀에 배치(또는 `grid-auto-flow`가 column이면 열 우선).
- `grid-auto-flow: dense`는 가능한 빈틈을 채우기 위해 후속 아이템을 앞으로 당겨 넣으려 시도 → DOM 순서와 시각 순서가 달라짐. 접근성(키보드 포커스/스크린리더)에 영향 가능 → 필요할 때만 사용.

## 6) 반응형/실무에서 가장 많이 쓰는 패턴 (꼭 외우세요)

### 카드/갤러리: 자동 열 수 조정

```css
.grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
  gap: 16px;
}
```

- `auto-fit`(또는 `auto-fill`) + `minmax(최소, 1fr)` 패턴은 브라우저 너비에 따라 **열 개수가 자동 변하는 반응형 그리드**를 만들어 줍니다. (auto-fit vs auto-fill 차이는 빈 트랙 처리 방식 차이)

### 레이아웃(헤더·사이드·메인·푸터) — `grid-template-areas`

```css
.container {
  display: grid;
  grid-template-columns: 240px 1fr;
  grid-template-rows: auto 1fr auto;
  grid-template-areas:
    "header header"
    "aside main"
    "footer footer";
  gap: 12px;
}
.header { grid-area: header; }
.aside  { grid-area: aside; }
.main   { grid-area: main; }
.footer { grid-area: footer; }
```

- 가독성/유지보수에 유리한 방식.

### 대시보드: 복잡한 그리드 트랙

```css
.dashboard {
  display: grid;
  grid-template-columns: repeat(12, 1fr); /* 12-col system */
  grid-auto-rows: minmax(100px, auto);
  gap: 12px;
}
.card { grid-column: span 4; } /* 카드는 4컬럼 폭 */
```

## 7) 실무 팁 & 함정

- **`gap` 사용**: 요소 간 간격은 `gap`이 훨씬 깔끔.
- **overflow 방지**: 긴 텍스트나 긴 단어가 있는 자식은 `min-width: 0`/`min-height: 0`를 줘야 그리드 셀에서 줄바꿈/오버플로우가 잘 처리됩니다(특히 flex에서 자주 보이지만 grid에서도 필요).
- **`dense`는 신중히**: 배치 성능 향상에 도움되지만, DOM 순서를 바꿀 수 있어 접근성 문제 유발.
- **Grid는 2차원, Flex는 1차원**: 전체 레이아웃(행+열)에는 Grid, 버튼 모음/일렬 정렬 등 단일 축 정렬은 Flex를 선호.
- **네이밍/영역 사용**: 작은 UI에는 라인 기반 또는 fr 단위가 더 간단하지만, 복잡한 레이아웃은 `grid-template-areas`로 가독성 확보.

## 8) Subgrid (중요 기능 및 호환성)

- `subgrid`는 **중첩된 그리드가 부모 그리드의 트랙(라인/크기)을 상속**하게 해 주는 기능으로, 중첩된 요소들이 부모의 열/행과 정확히 정렬되도록 도와줍니다. (유용 사례: 부모 그리드 기준으로 하위 카드의 내부 레이아웃을 맞추고 싶을 때)
- **브라우저 호환성**: `subgrid` 지원 여부는 브라우저별로 달라집니다 — 최신 브라우저에서 점차 지원이 더 좋아지고 있으니 실제 사용 전 지원 현황을 확인하세요.

## Flex vs Grid 비교 정리

| 항목     | Flexbox             | Grid                   |
| ------ | ------------------- | ---------------------- |
| 레이아웃   | 1차원 (한 줄 또는 한 열)    | 2차원 (행 + 열)            |
| 적합한 상황 | 버튼 그룹, 네비게이션, 간단 정렬 | 전체 페이지 레이아웃, 갤러리, 대시보드 |

## 관련 개념

- [[css]]: Grid의 기반이 되는 CSS
- [[flexbox]]: 1차원 레이아웃 시스템
- [[html]]: Grid로 배치하는 HTML 구조
