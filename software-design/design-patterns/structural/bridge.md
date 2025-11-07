#DesignPattern

# Bridge

**추상화(Abstraction)** 와 **구현(Implementation)** 을 분리하여, 두 부분을 독립적으로 확장할 수 있게 해주는 패턴입니다.
쉽게 말해, "상속 대신 위임(조합)"을 활용해서 **확장성을 높이는 구조**입니다.

## 기본 구현

```js
// 구현부 (Implementation)
class Device {
  turnOn() {}
  turnOff() {}
}

class TV extends Device {
  turnOn() {
    console.log("TV 켜짐");
  }
  turnOff() {
    console.log("TV 꺼짐");
  }
}

class Radio extends Device {
  turnOn() {
    console.log("라디오 켜짐");
  }
  turnOff() {
    console.log("라디오 꺼짐");
  }
}

// 추상화 (Abstraction)
class RemoteControl {
  constructor(device) {
    this.device = device;
  }

  togglePower() {
    this.device.turnOn();
  }
}

// 확장된 추상화 (Refined Abstraction)
class AdvancedRemoteControl extends RemoteControl {
  mute() {
    console.log("음소거 처리");
  }
}

// 사용
const tvRemote = new AdvancedRemoteControl(new TV());
tvRemote.togglePower(); // TV 켜짐
tvRemote.mute();        // 음소거 처리

const radioRemote = new RemoteControl(new Radio());
radioRemote.togglePower(); // 라디오 켜짐
```

## 실제 활용 코드 예시

### 결제 시스템 (다양한 결제 수단 + 플랫폼)

```js
// 구현부: 결제 방법
class PaymentMethod {
  pay(amount) {}
}

class CreditCardPayment extends PaymentMethod {
  pay(amount) {
    console.log(`💳 카드로 ${amount}원 결제`);
  }
}

class PayPalPayment extends PaymentMethod {
  pay(amount) {
    console.log(`💻 PayPal로 ${amount}원 결제`);
  }
}

// 추상화: 결제 플랫폼
class PaymentPlatform {
  constructor(paymentMethod) {
    this.paymentMethod = paymentMethod;
  }

  process(amount) {
    this.paymentMethod.pay(amount);
  }
}

// 확장된 추상화
class OnlineShop extends PaymentPlatform {
  refund(amount) {
    console.log(`${amount}원 환불 처리`);
  }
}

// 사용
const shop1 = new OnlineShop(new CreditCardPayment());
shop1.process(5000); // 💳 카드로 5000원 결제

const shop2 = new OnlineShop(new PayPalPayment());
shop2.process(12000); // 💻 PayPal로 12000원 결제
shop2.refund(12000);  // 12000원 환불 처리
```

### 알림 시스템 (다양한 메시지 형태 + 전송 채널)

```js
// 구현부: 채널
class Notifier {
  send(message) {}
}

class EmailNotifier extends Notifier {
  send(message) {
    console.log(`📧 이메일 전송: ${message}`);
  }
}

class SMSNotifier extends Notifier {
  send(message) {
    console.log(`📱 SMS 전송: ${message}`);
  }
}

// 추상화: 알림
class Notification {
  constructor(notifier) {
    this.notifier = notifier;
  }

  notify(message) {
    this.notifier.send(message);
  }
}

// 확장된 알림
class UrgentNotification extends Notification {
  notify(message) {
    console.log("🚨 긴급 알림!");
    this.notifier.send(message);
  }
}

// 사용
const emailAlert = new UrgentNotification(new EmailNotifier());
emailAlert.notify("서버 다운!");

const smsAlert = new Notification(new SMSNotifier());
smsAlert.notify("새로운 메시지 도착");
```

## 언제 사용하면 좋은가?

- **클래스 계층이 복잡해질 때 (조합 폭발 문제)**  
    → 예: `RemoteControl + TV`, `RemoteControl + Radio`, `AdvancedRemoteControl + TV`, …  
    상속으로 하면 클래스가 기하급수적으로 늘어나지만, Bridge를 쓰면 조합으로 해결 가능
- **추상화와 구현을 독립적으로 확장하고 싶을 때**  
    → 새로운 디바이스를 추가해도 RemoteControl은 수정 필요 없음
- **다양한 차원의 확장이 동시에 필요한 경우**  
    → 알림(일반/긴급) + 채널(Email, SMS, Slack 등)

## 정리

- Adapter: "인터페이스를 변환" → 호환성 문제 해결
- Bridge: "추상화와 구현을 분리" → 확장성 문제 해결
