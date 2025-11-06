# DesignPattern

# Lazy Initialization

객체나 자원의 생성을 **필요할 때까지 미루는 패턴**입니다. "쓰일 때 초기화" → 불필요한 리소스 낭비 방지.

## 기본 구현

```js
class HeavyResource {
  constructor() {
    console.log("⚙️ 무거운 리소스 생성됨");
    this.data = Array(1000000).fill("데이터");
  }
}

class ResourceManager {
  constructor() {
    this._resource = null;
  }

  get resource() {
    if (!this._resource) {
      this._resource = new HeavyResource(); // 실제 필요할 때만 생성
    }
    return this._resource;
  }
}

// 사용
const manager = new ResourceManager();
console.log("아직 리소스 없음");

const r1 = manager.resource; // 이 시점에 생성됨
console.log("리소스 접근 완료");
```

## 실제 활용 코드 예시

### React 컴포넌트 (동적 import)

```js
import React, { lazy, Suspense } from "react";

const HeavyChart = lazy(() => import("./HeavyChart")); // 필요할 때 로드

function Dashboard({ showChart }) {
  return (
    <div>
      <h1>Dashboard</h1>
      {showChart && (
        <Suspense fallback={<div>로딩중...</div>}>
          <HeavyChart />
        </Suspense>
      )}
    </div>
  );
}
```

### Express 라우트에서 모듈 지연 로드

```js
import express from "express";
const app = express();

app.get("/report", async (req, res) => {
  const { generateReport } = await import("./report.js"); // 필요할 때 import
  const report = generateReport();
  res.send(report);
});
```

## 언제 사용하면 좋은가?

- 객체 생성 비용이 크고, 항상 쓰이지는 않을 때
- 초기 로딩 시간을 줄이고 싶을 때 (예: 웹앱 번들 사이즈 최적화)
- 예:
  - 무거운 데이터 로딩 (차트, 대용량 파일, 이미지)
  - 일부 기능이 옵션일 때 (관리자 페이지 등)
  - 서버 사이드에서 특정 요청이 있을 때만 모듈 불러오기

## 정리

- **장점**: 초기 성능 최적화, 메모리 절약
- **단점**: 최초 접근 시 지연 발생, 동시 접근 시 Race Condition 주의 필요
