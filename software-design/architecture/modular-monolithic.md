---
tags:
  - Architecture
title: Modular Monolithic
---
## 모놀리식과 차이점

2025-12 기준 최근 많은 팀들이 마이크로서비스 아키텍처(MSA)의 높은 복잡도와 운영 비용에 지쳐 **모듈형 모놀리식(Modular Monolithic)** 으로 회귀하거나 이를 선택하는 추세입니다.

모든 코드가 하나로 뭉쳐 있는 일반적인 **모놀리식**과, 구조를 개선한 **모듈형 모놀리식**의 핵심 차이점

| **구분**     | **일반 모놀리식 (Monolithic)** | **모듈형 모놀리식 (Modular Monolithic)** |
| ---------- | ------------------------ | --------------------------------- |
| **내부 구조**  | 기능들이 서로 얽혀 있는 '스파게티' 구조  | 기능별로 경계(Boundary)가 명확히 나뉜 구조      |
| **의존성**    | 어디서든 무엇이든 호출 가능 (강결합)    | 정해진 인터페이스(API)를 통해서만 소통 (약결합)     |
| **데이터베이스** | 모든 데이터가 하나의 스키마에 혼재      | 모듈별로 테이블 권한을 분리하거나 논리적 격리         |
| **난이도**    | 설계가 쉽지만 유지보수가 어려움        | 초기 설계 비용이 높지만 유지보수가 용이함           |

### 주요 차이점 상세 분석

#### 구조적 응집도와 결합도

- **일반 모놀리식:** 주문(Order) 클래스에서 직접 재고(Stock) 클래스의 내부 변수를 수정하거나, 결제 로직이 여기저기 흩어져 있는 경우가 많습니다. 시간이 흐를수록 코드 수정이 다른 곳에 어떤 영향을 줄지 예측하기 힘듭니다.
- **모듈형 모놀리식:** 코드를 성격에 따라 독립된 모듈로 나눕니다. 각 모듈은 **독립적인 API(Interface)** 를 가집니다. 예를 들어 주문 모듈이 재고를 확인하려면 재고 모듈이 열어둔 'Public API'만 사용해야 하며, 재고 모듈 내부의 복잡한 로직은 알 필요가 없습니다.

#### 데이터베이스 관리

- **일반 모놀리식:** 하나의 커다란 DB에 수백 개의 테이블이 있고, 여러 서비스가 서로의 테이블을 `JOIN`해서 가져다 씁니다. 테이블 구조 하나 바꾸려면 전체 시스템을 다 뒤져봐야 합니다.
- **모듈형 모놀리식:** DB는 하나를 쓰더라도, **논리적으로 영역을 나눕니다.** 특정 모듈은 자신에게 할당된 테이블만 건드리는 것이 원칙입니다. 다른 모듈의 데이터가 필요하면 코드를 통해 요청합니다. 이렇게 하면 나중에 특정 모듈만 떼어내서 MSA로 전환하기가 매우 쉽습니다.

### 왜 "모듈형"이 대세인가요?

MSA로 가기에는 **운영 리소스(배포, 네트워크 지연, 트랜잭션 관리)** 가 너무 부족하고, 그렇다고 일반 모놀리식으로 짜기엔 **코드 스파게티**가 두려운 팀들에게 최고의 절충안이기 때문입니다.

- **배포가 쉽다:** MSA처럼 여러 서비스를 따로 배포할 필요 없이 한 번만 배포하면 됩니다.
- **성능이 좋다:** 서비스 간 통신이 네트워크(HTTP/gRPC)가 아닌 메모리 내 함수 호출이므로 빠릅니다.
- **MSA로의 징검다리:** 모듈화가 잘 된 시스템은 나중에 트래픽이 몰리는 특정 모듈만 쏙 뽑아서 독립된 서비스(Microservice)로 만들기 매우 유리합니다.

### 요약: 무엇을 선택해야 할까?

> "애플리케이션이 하나로 배포되느냐(Monolith), 여러 개로 쪼개져 배포되느냐(MSA)보다 중요한 것은 **코드 내부가 얼마나 잘 격리되어 있는가**입니다."

결국 모듈형 모놀리식은 **"배포는 하나로 편하게 하되, 코드는 마이크로서비스처럼 깔끔하게 관리하자"** 는 전략입니다.

## 예시

쇼핑몰 시스템을 예로 들어 **Node.js** 환경에서 **모듈형 모놀리식(Modular Monolithic)** 아키텍처를 어떻게 구성하는지 DB와 API 구조를 중심으로 설명해 드리겠습니다.

### 1. DB 구조: 논리적 격리 (Logical Separation)

모듈형 모놀리식의 핵심은 **"물리적으로는 하나의 DB를 쓰지만, 논리적으로는 남의 테이블을 직접 참조하지 않는다"** 는 것입니다.
- **기존 모놀리식:** `Orders` 테이블이 `Users`와 `Products` 테이블을 직접 `JOIN`해서 가져옴.
- **모듈형 모놀리식:** 각 모듈은 자신의 스키마(또는 테이블 접두사)를 소유하며, 외래키(FK) 제약조건을 최소화하거나 애플리케이션 레벨에서 관리함.

|**모듈 (Module)**|**소유 테이블 예시**|**설명**|
|---|---|---|
|**User (회원)**|`users`, `addresses`|사용자 프로필 및 배송지 정보|
|**Catalog (상품)**|`products`, `categories`|상품 정보, 가격, 카테고리|
|**Order (주문)**|`orders`, `order_items`|주문 이력 (상품 ID만 저장, JOIN 지양)|
|**Stock (재고)**|`stocks`, `stock_logs`|상품별 실시간 재고 수량|
### 2. NodeJS 프로젝트 구조 (Directory Structure)

`src/modules` 디렉토리 아래에 각 도메인을 독립적인 폴더로 격리합니다. 각 모듈은 마치 작은 마이크로서비스처럼 동작합니다.

```text
src/
├── modules/
│   ├── user/             # 회원 모듈
│   │   ├── user.controller.ts
│   │   ├── user.service.ts
│   │   ├── user.repository.ts
│   │   └── index.ts      # 모듈의 Public API(Interface) 정의
│   ├── catalog/          # 상품 모듈
│   │   ├── catalog.service.ts
│   │   └── ...
│   └── order/            # 주문 모듈
│       ├── order.service.ts
│       └── ...
├── shared/               # 공통 유틸리티 (에러 핸들러, DB 설정 등)
└── app.ts                # 모든 모듈을 하나로 합쳐 실행하는 진입점
```

### 3. API 구조 예시 (Code)

주문(Order) 모듈에서 상품(Catalog) 정보를 가져와 주문을 생성하는 로직의 흐름입니다.
#### ① 상품 모듈 (Catalog Module) - 내부 Interface 제공

`src/modules/catalog/index.ts`
```ts
// 다른 모듈에서 상품 정보가 필요할 때 사용할 수 있도록 외부로 노출하는 함수
import { CatalogService } from './catalog.service';

export const CatalogModule = {
  getProductForOrder: async (productId: string) => {
    const service = new CatalogService();
    return await service.getProductById(productId);
  }
};
```

#### ② 주문 모듈 (Order Module) - 타 모듈 호출

`src/modules/order/order.service.ts`
```ts
import { CatalogModule } from '../catalog'; // 중요: 상품 모듈의 '공개 인터페이스'만 참조

export class OrderService {
  async createOrder(userId: string, productId: string) {
    // 1. DB JOIN 대신 다른 모듈의 Interface를 통해 데이터 조회
    const product = await CatalogModule.getProductForOrder(productId);
    
    if (!product) throw new Error("상품이 존재하지 않습니다.");

    // 2. 주문 로직 수행
    const order = await this.orderRepository.save({
      userId,
      productId: product.id,
      price: product.price, // 조회해온 가격 정보 사용
      status: 'CREATED'
    });

    return order;
  }
}
```

### 4. 이 구조의 장점 (쇼핑몰 시나리오)

1. **영향도 분리:** 이벤트 프로모션 때문에 `Catalog` 모듈의 로직을 대폭 수정해도, `Order` 모듈의 코드는 `CatalogModule.getProductForOrder`라는 인터페이스만 유지된다면 수정할 필요가 없습니다.
2. **테스트 용이성:** 주문 로직을 테스트할 때 실제 DB를 복잡하게 엮지 않고, `CatalogModule`만 모킹(Mocking)하여 독립적으로 테스트할 수 있습니다.
3. **부분적 MSA 전환:** 만약 블래프라이데이 이벤트로 인해 '상품 조회' 트래픽만 감당하기 힘들어진다면, 이미 분리된 `catalog` 폴더를 그대로 떼어내서 별도의 서버(Microservice)로 배포하기가 매우 수월합니다.

이처럼 모듈형 모놀리식은 **"개발 생산성(Monolith)"** 과 **"시스템의 유연성(MSA)"** 사이에서 균형을 잡는 매우 현실적인 대안입니다.