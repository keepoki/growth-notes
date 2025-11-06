# Optimization

# Debouncing (디바운싱), Throttling (쓰로틀링)

자바스크립트에서 **디바운싱(Debouncing)과 쓰로틀링(Throttling)은** 연속적으로 발생하는 이벤트를 제어하여 성능을 최적화하는 기술입니다. 두 기술 모두 이벤트 핸들러의 실행 횟수를 줄이는 것이 목적이지만, 동작 방식에 차이가 있습니다.

## 디바운싱 (Debouncing)

**디바운싱**은 특정 이벤트가 **일정 시간 동안 다시 발생하지 않을 때 마지막 이벤트를 한 번만 실행하는 기술**입니다. 주로 입력창에 텍스트를 입력할 때와 같이 사용자가 이벤트 발생을 멈춘 후에만 최종 결과를 처리하고 싶을 때 사용됩니다.

- **동작 방식:** 이벤트가 발생할 때마다 타이머를 초기화합니다. 설정한 시간(예: 300ms) 내에 이벤트가 다시 발생하면 이전 타이머는 취소되고 새로운 타이머가 설정됩니다. 이 과정을 반복하다가 이벤트가 더 이상 발생하지 않아 타이머가 만료되면, 그제서야 이벤트 핸들러가 한 번 실행됩니다.
- **주요 사용 사례:**
  - **검색창 자동완성:** 사용자가 입력을 멈췄을 때만 검색 API를 호출하여 불필요한 네트워크 요청을 줄입니다.
  - **윈도우 리사이즈:** 창 크기 조절이 끝났을 때만 최종 크기에 맞게 UI를 다시 그립니다.

### 디바운싱(Debouncing) 예제 코드

```js
// 디바운싱 함수
function debounce(callback, delay) {
  let timerId;
  
  return (...args) => {
    // delay가 지나기 전에 이벤트가 다시 발생하면 이전 타이머를 취소
    clearTimeout(timerId);
    
    // 새로운 타이머 시작
    timerId = setTimeout(() => {
      callback(...args);
    }, delay);
  };
}

// 예시: 입력창에 텍스트를 입력할 때마다 검색 API를 호출하는 대신, 
// 500ms 동안 입력이 없으면 한 번만 호출
const searchInput = document.getElementById('search-input');

function handleSearch(event) {
  console.log(`검색어: ${event.target.value}`);
  // 여기에 실제 검색 API 호출 로직이 들어갑니다.
}

// searchInput에 'input' 이벤트가 발생할 때 debounce 함수를 적용
searchInput.addEventListener('input', debounce(handleSearch, 500));
```

## 쓰로틀링 (Throttling)

**쓰로틀링**은 이벤트가 **일정 시간 간격으로만 실행되도록 하는 기술**입니다. 이벤트가 연속적으로 발생하더라도, 설정한 시간(예: 300ms) 동안에는 이벤트 핸들러가 다시 실행되지 않습니다.

- **동작 방식:** 이벤트가 발생하면 타이머를 시작하고 이벤트 핸들러를 실행합니다. 타이머가 동작 중일 때는 이후에 발생하는 모든 이벤트는 무시됩니다. 타이머가 만료되면 다시 이벤트 핸들러를 실행할 수 있는 상태가 됩니다.
- **주요 사용 사례:**
  - **스크롤 이벤트:** 스크롤 위치를 계산하는 이벤트를 일정 시간마다 한 번씩만 실행하여 잦은 DOM 조작을 방지합니다.
  - **버튼 연속 클릭 방지:** 버튼을 빠르게 여러 번 눌러도 첫 번째 클릭만 처리하고, 일정 시간 동안은 추가적인 클릭이 무시되게 만듭니다.

### 쓰로틀링(Throttling) 예제 코드

```js
// 쓰로틀링 함수
function throttle(callback, delay) {
  let timerId;

  return (...args) => {
    // 타이머가 이미 실행 중이면 새로운 호출을 무시
    if (timerId) {
      return;
    }

    // 콜백 함수를 실행하고 타이머 시작
    callback(...args);

    timerId = setTimeout(() => {
      // delay 시간 후 타이머를 해제하여 다시 함수를 호출할 수 있도록 함
      timerId = null;
    }, delay);
  };
}

// 예시: 스크롤 이벤트를 200ms 간격으로 한 번씩만 처리
const container = document.getElementById('scroll-container');

function handleScroll() {
  console.log('스크롤 이벤트 발생!');
  // 여기에 스크롤 위치에 따른 UI 업데이트 로직이 들어갑니다.
}

// container에 'scroll' 이벤트가 발생할 때 throttle 함수를 적용
container.addEventListener('scroll', throttle(handleScroll, 200));
```

## 디바운싱과 쓰로틀링의 비교

| =======   | **디바운싱 (Debouncing)**               | **쓰로틀링 (Throttling)**                           |
| --------- | ----------------------------------- | ----------------------------------------------- |
| **실행 시점** | 마지막 이벤트 발생 후 일정 시간이 지난 뒤 한 번 실행     | 일정 시간 간격으로 실행                                   |
| **실행 횟수** | 이벤트가 멈춘 후 딱 한 번                     | 설정한 시간 간격마다 한 번                                 |
| **주요 용도** | 최종 결과가 중요한 경우 (입력 완료, 창 크기 조절 완료 등) | 연속적인 이벤트 발생 빈도를 제한해야 하는 경우 (스크롤, 리사이즈, 게임 조작 등) |
