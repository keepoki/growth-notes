#DesignPattern

# DesignPattern

**디자인 패턴(Design Pattern)은** 소프트웨어 설계에서 **자주 반복되는 문제를 해결하기 위해 검증된 재사용 가능한 설계 방법**을 말합니다.

즉, 패턴이란 "특정 상황에서 문제를 해결하는 데 효과적인 형식화된 설계 방법"이에요.  
개발자가 처음부터 모든 설계를 고민할 필요 없이, 이미 많은 사람들이 검증해둔 설계 아이디어를 참고할 수 있도록 정리된 **공통의 해결책**이라고 보면 됩니다.

## 특징
- **재사용성**: 비슷한 문제에 반복적으로 적용 가능
- **가독성 향상**: 개발자끼리 “이건 싱글톤 패턴이네”라고 말하면 바로 이해 가능
- **유지보수 용이**: 구조가 명확해서 코드 확장과 변경이 쉬움
- **표준화된 의사소통**: 개발자들 사이에 공통 언어처럼 사용됨

## 분류
GoF(“Gang of Four”) 책 _Design Patterns: Elements of Reusable Object-Oriented Software_ 에서 23가지 패턴을 제시했고, 크게 3가지로 나눠요:

### **생성(Creational) 패턴**
객체 생성 방식을 캡슐화 → 코드 유연성과 재사용성 증가

- **Singleton** : 객체를 하나만 생성
- **Factory Method** : 객체 생성을 서브클래스에 위임
- **Abstract Factory** : 관련 객체들을 묶어서 생성
- **Builder** : 복잡한 객체를 단계별로 생성
- **Prototype** : 기존 객체를 복제해서 생성

### 구조(Structural) 패턴
클래스와 객체를 조합 → 더 큰 구조 형성

- **Adapter** : 인터페이스 변환 (호환성 해결)
- **Bridge** : 구현과 추상을 분리
- **Composite** : 트리 구조 (부분-전체 계층 표현)
- **Decorator** : 객체에 동적으로 기능 추가
- **Facade** : 복잡한 시스템을 단순 인터페이스로 감싸기
- **Flyweight** : 공유 객체로 메모리 절약
- **Proxy** : 대리 객체를 통해 접근 제어

### 행위(Behavioral) 패턴
객체 간 책임 분산 및 협력 방식 정의

- **Chain of Responsibility** : 요청을 책임 연쇄로 처리
- **Command** : 요청을 객체로 캡슐화 (실행/취소/로그)
- **Interpreter** : 문법 해석기 구현
- **Iterator** : 컬렉션 요소 순차 접근
- **Mediator** : 객체 간 중재자 (직접 통신 제거)
- **Memento** : 객체 상태 저장/복원
- **Observer** : 상태 변화를 구독-알림
- **State** : 상태에 따른 행동 변경
- **Strategy** : 알고리즘을 캡슐화하여 교체 가능
- **Template Method** : 알고리즘 뼈대 정의, 일부 단계 오버라이드
- **Visitor** : 구조 변경 없이 새로운 연산 추가

### 동시성(Concurrency) 패턴
멀티스레드, 비동기 환경에서 자주 사용

- **Active Object** : 메서드 호출을 비동기적으로 실행
- **Half-Sync/Half-Async** : 동기/비동기 계층 분리
- **Leader-Followers** : 리소스 관리 스레드 풀
- **Producer-Consumer** : 작업 생성자와 처리자 분리
- **Thread Pool** : 스레드 재사용

### 엔터프라이즈 애플리케이션 패턴 (Martin Fowler)

대규모 웹/비즈니스 앱에서 흔히 등장

- **Repository** : 데이터 접근 추상화
- **Unit of Work** : 트랜잭션 단위로 객체 변경 추적
- **Active Record** : DB 레코드를 객체로 매핑
- **Data Mapper** : 객체와 DB 매핑을 별도 계층에서 관리
- **Identity Map** : 같은 객체를 중복 조회하지 않음
- **Service Layer** : 비즈니스 로직을 서비스 계층에 두기

### 아키텍처 (Architecture) 패턴
시스템 전체 설계 구조에서 사용

- **MVC (Model-View-Controller)** : UI 분리
- **MVP (Model-View-Presenter)**
- **MVVM (Model-View-ViewModel)**
- **Layered Architecture (계층형 아키텍처)**
- **Hexagonal Architecture (Ports & Adapters)**
- **Clean Architecture**
- **Microservices Architecture**
- **Event-Driven Architecture**
- **CQRS (Command Query Responsibility Segregation)**

### 분산 시스템 패턴

마이크로서비스나 클라우드 환경에서 유용

- **Circuit Breaker** : 장애 전파 방지 (넷플릭스 Hystrix로 유명)
- **Service Discovery** : 마이크로서비스 위치 동적 관리
- **API Gateway** : 요청 라우팅/인증/로깅 중앙 처리
- **Saga Pattern** : 분산 트랜잭션 관리
- **Bulkhead Pattern** : 장애 격리
- **Event Sourcing** : 상태를 이벤트 로그로 저장

### 기타 패턴

- **Null Object** : null 대신 기본 동작 객체 사용
- **Object Pool** : 재사용 가능한 객체 풀 관리
- **Lazy Initialization** : 필요할 때까지 객체 생성 지연
- **Specification** : 비즈니스 규칙을 객체로 캡슐화
- **Dependency Injection (DI)** : 의존성 주입
- **Publish–Subscribe (Pub/Sub)** : 메시지 브로커 기반 알림

## 상황별 디자인 패턴 선택 가이드

### 객체 생성 관련

- **하나의 인스턴스만 필요** → `Singleton`
- **객체 생성 로직을 감추고 싶음** → `Factory Method`
- **서로 관련된 객체 군을 묶어서 생성해야 함** → `Abstract Factory`
- **복잡한 객체를 단계적으로 조립** → `Builder`
- **기존 객체를 복제해서 재사용** → `Prototype`

### 구조 설계 관련

- **서로 다른 인터페이스를 연결** → `Adapter`
- **추상과 구현을 독립적으로 확장** → `Bridge`
- **트리 구조(부분-전체 계층) 표현** → `Composite`
- **기능을 동적으로 추가/확장** → `Decorator`
- **복잡한 시스템을 단순한 API로 제공** → `Facade`
- **많은 객체를 공유해 메모리 절약** → `Flyweight`
- **대리 객체를 두어 접근 제어/캐싱/로깅** → `Proxy`

### 행동/로직 제어 관련

- **여러 처리자 중 누가 처리할지 모름** → `Chain of Responsibility`
- **요청을 실행/취소/로그로 관리** → `Command`
- **문법 해석기/DSL 구현** → `Interpreter`
- **컬렉션 반복 처리** → `Iterator`
- **객체 간 직접 통신 제거 (중재자 추가)** → `Mediator`
- **객체 상태를 저장/복구** → `Memento`
- **변경 사항을 여러 객체에게 알림** → `Observer`
- **객체 상태에 따라 행동이 바뀜** → `State`
- **알고리즘을 교체 가능하게 캡슐화** → `Strategy`
- **알고리즘 뼈대는 같지만 일부 단계만 다름** → `Template Method`
- **구조 변경 없이 기능 추가** → `Visitor`

### 한눈에 보기 (키워드 매핑)

- **인스턴스 관리** → Singleton
- **객체 생성 유연성** → Factory / Abstract Factory / Builder / Prototype
- **인터페이스 맞추기** → Adapter
- **구조 확장** → Composite / Bridge / Decorator
- **복잡한 시스템 단순화** → Facade
- **리소스 최적화** → Flyweight / Proxy
- **행동 위임** → Strategy / State / Template Method
- **행동 확장** → Visitor / Observer / Command / Chain of Responsibility
- **객체 관계 단순화** → Mediator / Iterator / Memento / Interpreter

### 요약
- "객체 생성 문제"라면 **생성 패턴**
- "클래스/객체 구조 문제"라면 **구조 패턴**
- "행동/로직 문제"라면 **행위 패턴**

## 학습 사이트
https://github.com/RefactoringGuru/design-patterns-typescript
