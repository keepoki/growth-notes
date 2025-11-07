#Architecture

# DDD (Domain-Driven Design)

**DDD (Domain-Driven Design)는 도메인 주도 개발**이라고 하며, 복잡한 소프트웨어를 효과적으로 설계하고 구현하기 위한 접근 방식입니다. 2004년 에릭 에반스(Eric Evans)가 쓴 책 _"Domain-Driven Design: Tackling Complexity in the Heart of Software"_ 에서 처음 정리된 개념입니다.

## 도메인 주도 개발 (DDD, Domain-Driven Design)이란?

도메인 주도 개발은 다음과 같은 철학을 가지고 있습니다:

> **"비즈니스의 핵심 개념(=도메인)을 코드의 중심에 두고, 개발자와 도메인 전문가가 협력하여 복잡한 문제를 해결하는 방식"**

즉, 소프트웨어가 해결하고자 하는 **비즈니스 문제(도메인)** 를 깊이 이해하고, 그것을 중심으로 시스템을 **설계, 구조화, 개발** 하자는 접근입니다.

### 1. 도메인(Domain)이란 무엇인가?

**도메인(Domain)** 은 소프트웨어가 해결하려는 **문제 영역**을 말합니다. 즉, 비즈니스의 대상이 되는 세계입니다.

- 예시:
  - 인터넷 쇼핑몰에서는 **상품, 장바구니, 주문, 결제** 등이 도메인입니다.
  - 병원 시스템이라면 **환자, 진료, 예약, 의사** 등이 도메인입니다.
  - 회계 시스템이라면 **회계 계정, 분개, 전표** 등이 도메인이 됩니다.

**도메인 전문가**란? → 실제 그 분야를 잘 아는 사람 (ex. 쇼핑몰 운영자, 의사, 회계사 등)

### 2. DDD의 핵심 요소 요약

| 개념                            | 설명                               |
| ----------------------------- | -------------------------------- |
| **도메인**                       | 해결하려는 문제의 영역 (비즈니스 세계)           |
| **도메인 모델**                    | 도메인을 소프트웨어로 표현한 모델 (클래스, 객체 등으로) |
| **유비쿼터스 언어**                  | 개발자와 도메인 전문가가 공유하는 공통 언어         |
| **Bounded Context (경계 컨텍스트)** | 의미 있는 모델이 적용되는 논리적 범위            |
| **엔티티, 값 객체, 서비스 등**          | 도메인 모델을 구성하는 패턴들                 |

### 3. DDD의 목적은?

- 비즈니스 복잡도를 코드에서 그대로 표현해서 이해와 변경을 쉽게 하자
- 개발자와 도메인 전문가가 **같은 언어(유비쿼터스 언어)** 로 대화하며 협업하자
- 객체지향 설계를 비즈니스 중심으로 제대로 활용하자

### 4. 간단한 예시

쇼핑몰 예시에서 주문(Order)이라는 도메인을 DDD 식으로 설계한다면:

- **Order**: 엔티티 (ID를 갖고, 상태를 가짐)
- **OrderLineItem**: 값 객체 (상품, 수량, 가격)
- **OrderService**: 비즈니스 로직을 포함 (ex. 주문 생성, 결제 처리 등)
- **OrderRepository**: 데이터를 저장하고 조회하는 역할

그리고 개발자와 비즈니스 담당자가 모두 "**주문은 승인될 수 있고, 취소될 수 있다**" 같은 언어로 이야기하고, 이것이 코드에 그대로 반영되게끔 합니다.

### 5. DDD를 언제 적용하면 좋을까?

- 비즈니스 로직이 복잡하거나 자주 변할 때
- 여러 팀이 협업하며 도메인이 커질 때
- 단순 CRUD가 아니라, 비즈니스 정책과 규칙이 중요한 경우

### 정리

- DDD는 **비즈니스의 핵심을 코드 중심에 두는 설계 방법**입니다.
- 도메인이란, 소프트웨어가 해결하고자 하는 **비즈니스 문제 영역**입니다.
- DDD는 "비즈니스 전문가 + 개발자"가 **공통 언어**를 가지고 **협력**하여 **복잡한 시스템을 만들기 쉽게 하려는 전략**입니다.

---

## DDD의 주요 구성요소

| 요소                            | 설명                                  | 예시 (쇼핑몰: 주문 도메인)                              |
| ----------------------------- | ----------------------------------- | --------------------------------------------- |
| **엔티티 (Entity)**              | 고유 ID를 가진 객체. 상태가 시간에 따라 변함         | `Order`, `User`                               |
| **값 객체 (Value Object)**       | 고유 ID 없음. 속성 값 자체로 동일성 판단           | `Money`, `Address`, `OrderItem`               |
| **도메인 서비스 (Domain Service)**  | 특정 엔티티/값 객체에 속하지 않는 비즈니스 로직         | 결제 처리, 할인 정책 계산                               |
| **애그리게잇 (Aggregate)**         | 연관된 엔티티와 값 객체들의 집합. 일관성 있는 단위       | `Order`(루트), 그 안의 `OrderItem`                 |
| **애그리게잇 루트 (Aggregate Root)** | 애그리게잇의 대표 엔티티. 외부에서 접근하는 유일한 진입점    | `Order`                                       |
| **리포지토리 (Repository)**        | 애그리게잇을 저장/조회하는 추상화                  | `OrderRepository`                             |
| **유비쿼터스 언어**                  | 개발자 + 도메인 전문가가 공유하는 언어              | “주문(Order)은 생성(Create)되고, 결제(Payment)될 수 있다.” |
| **Bounded Context (경계 컨텍스트)** | 특정 모델이 적용되는 경계. 도메인이 크면 여러 컨텍스트로 나눔 | `주문 컨텍스트`, `결제 컨텍스트` 등                        |

---

## NestJS로 간단히 표현한 예시 (Order 도메인)

> 상황: 사용자가 상품을 담아 주문을 생성하는 기능

### 1. 값 객체 (OrderItem)

```ts
// domain/order-item.vo.ts
export class OrderItem {
  constructor(
    public readonly productId: string,
    public readonly quantity: number,
    public readonly price: number,
  ) {
    if (quantity <= 0) {
      throw new Error('수량은 1개 이상이어야 합니다.');
    }
  }

  get total(): number {
    return this.quantity * this.price;
  }
}
```

### 2. 엔티티 + 애그리게잇 루트 (Order)

```ts
// domain/order.entity.ts
import { OrderItem } from './order-item.vo';

export class Order {
  private items: OrderItem[] = [];
  private isPaid: boolean = false;

  constructor(
    public readonly id: string,
    public readonly userId: string,
  ) {}

  addItem(item: OrderItem) {
    this.items.push(item);
  }

  get totalAmount(): number {
    return this.items.reduce((sum, i) => sum + i.total, 0);
  }

  pay() {
    if (this.isPaid) throw new Error('이미 결제된 주문입니다.');
    this.isPaid = true;
  }
}
```

### 3. 리포지토리 인터페이스

```ts
// domain/order.repository.ts
import { Order } from './order.entity';

export interface OrderRepository {
  save(order: Order): Promise<void>;
  findById(id: string): Promise<Order | null>;
}
```

### 4. 인프라 구현 (DB 연동)

```ts
// infrastructure/order.repository.impl.ts
import { Injectable } from '@nestjs/common';
import { OrderRepository } from '../domain/order.repository';
import { Order } from '../domain/order.entity';

@Injectable()
export class OrderRepositoryImpl implements OrderRepository {
  private db: Map<string, Order> = new Map(); // 예시용 in-memory DB

  async save(order: Order): Promise<void> {
    this.db.set(order.id, order);
  }

  async findById(id: string): Promise<Order | null> {
    return this.db.get(id) ?? null;
  }
}
```

### 5. 애플리케이션 서비스 (Use Case)

```ts
// application/order.service.ts
import { Injectable } from '@nestjs/common';
import { Order } from '../domain/order.entity';
import { OrderItem } from '../domain/order-item.vo';
import { OrderRepository } from '../domain/order.repository';

@Injectable()
export class OrderService {
  constructor(private readonly orderRepo: OrderRepository) {}

  async createOrder(userId: string, productId: string, price: number, quantity: number): Promise<Order> {
    const order = new Order(Date.now().toString(), userId);
    order.addItem(new OrderItem(productId, quantity, price));
    await this.orderRepo.save(order);
    return order;
  }

  async payOrder(orderId: string): Promise<void> {
    const order = await this.orderRepo.findById(orderId);
    if (!order) throw new Error('주문을 찾을 수 없습니다.');
    order.pay();
    await this.orderRepo.save(order);
  }
}
```

### 6. 컨트롤러 (Interface Layer)

```ts
// interfaces/order.controller.ts
import { Controller, Post, Body } from '@nestjs/common';
import { OrderService } from '../application/order.service';

@Controller('orders')
export class OrderController {
  constructor(private readonly orderService: OrderService) {}

  @Post()
  async create(@Body() dto: { userId: string; productId: string; price: number; quantity: number }) {
    return this.orderService.createOrder(dto.userId, dto.productId, dto.price, dto.quantity);
  }
}
```

### NestJS 예시 정리

- **도메인 (Entity/Value Object/Service/Repository)** → 비즈니스 로직의 중심
- **애플리케이션 서비스** → 유즈케이스를 실행하는 계층
- **인프라 계층** → DB, 외부 시스템과 연결
- **인터페이스 계층 (Controller)** → 외부 요청을 받고 응답

DDD를 NestJS에 적용하면 **도메인 로직이 깔끔하게 분리**되어 유지보수가 쉬워지고, 팀 내에서 **"주문"이라는 도메인 언어**를 그대로 코드에 반영할 수 있습니다.

---

## NestJS에서 DDD 스타일 폴더 구조 예시

```scss
src
 └── modules
      └── order               # 주문 도메인 (Bounded Context)
           ├── domain         # 순수한 도메인 모델 (비즈니스 로직)
           │    ├── order.entity.ts
           │    ├── order-item.vo.ts
           │    ├── order.repository.ts
           │    └── exceptions.ts
           │
           ├── application    # 유즈케이스 (도메인 조합, 서비스 계층)
           │    └── order.service.ts
           │
           ├── infrastructure # 기술 세부사항 (DB, 외부 API 등)
           │    └── order.repository.impl.ts
           │
           ├── interfaces    # 입출력 계층 (Controller, DTO)
           │    ├── order.controller.ts
           │    └── dto
           │         └── create-order.dto.ts
           │
           └── order.module.ts # NestJS 모듈 등록
```

### 각 계층 역할

1. **domain**
    - 순수한 비즈니스 로직만 포함 (Entity, Value Object, Repository Interface 등)
    - 외부 의존성 없음 → 테스트하기 쉬움
2. **application**
    - 유즈케이스(Service) 구현
    - 도메인 모델을 활용해 시나리오 수행 (예: 주문 생성, 주문 결제)
3. **infrastructure**
    - DB, 메시지 브로커, 외부 API 같은 기술적 세부사항 구현
    - Repository 구현체 포함
4. **interfaces**
    - Controller, DTO, GraphQL Resolver 등 **입출력 담당**
    - REST API, gRPC, GraphQL 등 어떤 인터페이스든 이 계층에서 처리
5. **module.ts**
    - NestJS 모듈 단위로 등록 (의존성 주입 관리)

### NestJS 모듈 예시

```ts
// modules/order/order.module.ts
import { Module } from '@nestjs/common';
import { OrderController } from './interfaces/order.controller';
import { OrderService } from './application/order.service';
import { OrderRepository } from './domain/order.repository';
import { OrderRepositoryImpl } from './infrastructure/order.repository.impl';

@Module({
  controllers: [OrderController],
  providers: [
    OrderService,
    {
      provide: OrderRepository,
      useClass: OrderRepositoryImpl,
    },
  ],
})
export class OrderModule {}
```

이렇게 구조를 나누면:

- **도메인**은 독립적이고 깔끔하게 유지
- **인프라 변경(DB, API 교체 등)** 이 생겨도 도메인에 영향이 최소화
- 팀원이 "도메인 용어" 중심으로 대화 → 유지보수성과 협업 효율 상승
