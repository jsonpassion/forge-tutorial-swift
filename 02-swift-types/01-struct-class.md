# 구조체와 클래스

> struct vs class, 값 타입 vs 참조 타입

## 개요

Ch1에서 `Int`, `String`, `Array` 같은 기본 타입을 배웠죠? 그런데 실제 앱을 만들려면 "사용자", "상품", "게시글" 같은 **나만의 타입**이 필요합니다. Swift에서 새로운 타입을 만드는 두 가지 핵심 도구가 바로 **구조체(struct)** 와 **클래스(class)** 입니다. 둘은 비슷해 보이지만, 근본적으로 다른 철학을 가지고 있어요.

**선수 지식**: [Ch1. Swift 시작하기](../01-swift-basics/01-introduction.md) 전체 (변수, 컬렉션, 함수, 옵셔널)
**학습 목표**:
- struct와 class의 문법과 차이를 이해한다
- 값 타입과 참조 타입의 동작 방식을 구분한다
- 상황에 맞게 struct 또는 class를 선택할 수 있다

## 왜 알아야 할까?

Swift에서 타입을 직접 정의하지 않고는 앱을 만들 수 없습니다. SwiftUI의 모든 뷰는 `struct`이고, SwiftData의 모델은 `class`입니다. 어떤 상황에서 무엇을 써야 하는지 모르면 예상치 못한 버그가 생기거든요. 특히 "값을 복사했는데 원본이 바뀌었다!" 같은 문제는 값 타입과 참조 타입의 차이를 모르면 절대 해결할 수 없습니다.

## 핵심 개념

### 개념 1: 구조체(struct) 기본

> 💡 **비유**: 구조체는 **복사한 서류**입니다. 원본 서류를 복사기로 복사하면, 복사본에 메모를 해도 원본은 깨끗하죠. 각각이 독립적인 사본입니다.

```swift
// 구조체 정의
struct User {
    var name: String
    var age: Int
    var email: String
}

// 인스턴스 생성 — Swift가 자동으로 초기화 메서드를 만들어줍니다
let user1 = User(name: "민수", age: 25, email: "minsu@email.com")
print(user1.name)   // "민수"
print(user1.age)    // 25

// 값 타입 — 복사하면 독립적인 사본이 됩니다
var user2 = user1
user2.name = "지영"

print(user1.name)   // "민수" — 원본은 그대로!
print(user2.name)   // "지영" — 복사본만 변경됨
```

```output
민수
25
민수
지영
```

구조체의 핵심은 **값 타입(Value Type)** 이라는 것입니다. 변수에 대입하거나 함수에 전달하면 **복사본**이 만들어져요. 앞서 [컬렉션 타입](../01-swift-basics/03-collections.md)에서 배운 Array, Dictionary도 사실 구조체입니다!

### 개념 2: 클래스(class) 기본

> 💡 **비유**: 클래스는 **공유 문서 링크**입니다. 구글 문서 링크를 여러 명에게 보내면, 누가 수정하든 모두가 같은 문서를 보게 되죠. 복사본이 아니라 같은 원본을 가리키는 겁니다.

```swift
// 클래스 정의
class Account {
    var owner: String
    var balance: Int

    // 클래스는 초기화 메서드(init)를 직접 작성해야 합니다
    init(owner: String, balance: Int) {
        self.owner = owner
        self.balance = balance
    }
}

// 인스턴스 생성
let account1 = Account(owner: "민수", balance: 100000)

// 참조 타입 — 대입하면 같은 객체를 가리킵니다
let account2 = account1
account2.balance = 50000

print(account1.balance)   // 50000 — 원본도 바뀌었습니다!
print(account2.balance)   // 50000 — 같은 객체를 보고 있으니까요
```

```output
50000
50000
```

클래스는 **참조 타입(Reference Type)** 입니다. 변수에 대입하면 값을 복사하는 게 아니라, **같은 객체를 가리키는 참조(포인터)를 공유**합니다.

### 개념 3: 값 타입 vs 참조 타입 — 핵심 차이

두 타입의 차이를 한눈에 비교해 봅시다.

| 특성 | struct (값 타입) | class (참조 타입) |
|------|-----------------|-------------------|
| **대입/전달** | 복사본 생성 | 참조(주소) 공유 |
| **변경 영향** | 각 사본 독립적 | 한쪽 변경 → 모두 반영 |
| **let 선언** | 프로퍼티 변경 불가 | 프로퍼티 변경 가능 |
| **상속** | 불가 | 가능 |
| **init** | 자동 생성 (memberwise) | 직접 작성 필요 |
| **메모리** | 스택 (빠름) | 힙 + ARC (느림) |
| **Sendable** | 기본 Sendable | 별도 처리 필요 |

```swift
// let으로 선언했을 때의 차이
struct Point {
    var x: Int
    var y: Int
}

let structPoint = Point(x: 1, y: 2)
// structPoint.x = 10  // ❌ 에러! let 구조체는 프로퍼티도 변경 불가

class Circle {
    var radius: Int
    init(radius: Int) { self.radius = radius }
}

let classCircle = Circle(radius: 5)
classCircle.radius = 10  // ✅ let 클래스지만 프로퍼티 변경 가능!
// (참조 자체가 상수일 뿐, 가리키는 객체의 내용은 바꿀 수 있습니다)
```

### 개념 4: mutating — 구조체의 값 변경

구조체는 값 타입이기 때문에, 메서드 안에서 자신의 프로퍼티를 변경하려면 `mutating` 키워드가 필요합니다.

```swift
struct Counter {
    var count = 0

    // mutating: "이 메서드는 자기 자신을 변경합니다"라고 선언
    mutating func increment() {
        count += 1
    }

    mutating func reset() {
        count = 0
    }
}

var counter = Counter()
counter.increment()
counter.increment()
print(counter.count)   // 2

counter.reset()
print(counter.count)   // 0
```

```output
2
0
```

> ⚠️ **흔한 오해**: "`let` 구조체에서 `mutating` 메서드를 호출할 수 있다" — 불가능합니다! `let`으로 선언한 구조체는 변경할 수 없으므로 `mutating` 메서드를 호출하면 컴파일 에러가 납니다.

### 개념 5: struct vs class, 뭘 써야 할까?

Apple의 공식 가이드라인은 명확합니다: **기본적으로 struct를 사용하세요.**

- **struct를 쓰세요** (기본 선택):
  - 데이터를 담는 모델 (사용자, 상품, 좌표 등)
  - SwiftUI 뷰 (`View` 프로토콜을 따르는 모든 것)
  - 독립적인 복사본이 필요한 경우
  - Swift Concurrency에서 안전하게 공유할 때

- **class를 쓰세요** (특별한 이유가 있을 때만):
  - 참조를 공유해야 할 때 (여러 곳에서 같은 객체를 보고 수정)
  - 상속이 필요할 때
  - SwiftData의 `@Model` 클래스
  - Objective-C와 연동할 때 (`NSObject` 상속)

```swift
// ✅ 좋은 예: 데이터 모델은 struct
struct Product {
    var name: String
    var price: Int
    var category: String
}

// ✅ 좋은 예: 공유 상태가 필요하면 class
class ShoppingCart {
    var items: [Product] = []

    init() {}

    func add(_ product: Product) {
        items.append(product)
    }
}
```

## 실습: 직접 해보기

값 타입과 참조 타입의 차이를 체감해 봅시다.

```swift
import Foundation

// 구조체: 음식 주문 (각 주문은 독립적)
struct FoodOrder {
    var menu: String
    var quantity: Int
    var isDelivery: Bool

    func totalDescription() -> String {
        let method = isDelivery ? "배달" : "포장"
        return "\(menu) \(quantity)개 (\(method))"
    }
}

// 클래스: 주문 관리자 (하나만 존재, 공유)
class OrderManager {
    var orders: [FoodOrder] = []
    var orderCount: Int { orders.count }

    init() {}

    func addOrder(_ order: FoodOrder) {
        orders.append(order)
        print("📝 주문 추가: \(order.totalDescription())")
    }

    func printSummary() {
        print("\n📋 전체 주문 (\(orderCount)건)")
        print("────────────────")
        for (i, order) in orders.enumerated() {
            print("  \(i + 1). \(order.totalDescription())")
        }
    }
}

// 구조체는 복사됩니다
var order1 = FoodOrder(menu: "비빔밥", quantity: 2, isDelivery: true)
var order2 = order1              // 복사본 생성
order2.menu = "김치찌개"          // 복사본만 변경
order2.quantity = 1

print(order1.totalDescription()) // "비빔밥 2개 (배달)" — 원본 그대로
print(order2.totalDescription()) // "김치찌개 1개 (배달)" — 복사본만 변경

// 클래스는 참조를 공유합니다
let manager = OrderManager()
let sameManager = manager        // 같은 객체를 가리킴

manager.addOrder(order1)
sameManager.addOrder(order2)     // 같은 매니저에 추가됨

manager.printSummary()           // 2건 모두 표시 — 같은 객체니까!
```

```output
비빔밥 2개 (배달)
김치찌개 1개 (배달)
📝 주문 추가: 비빔밥 2개 (배달)
📝 주문 추가: 김치찌개 1개 (배달)

📋 전체 주문 (2건)
────────────────
  1. 비빔밥 2개 (배달)
  2. 김치찌개 1개 (배달)
```

## 더 깊이 알아보기

### Swift는 왜 struct를 기본으로 밀까?

Swift 이전의 Apple 주력 언어인 Objective-C에서는 거의 모든 것이 **클래스(참조 타입)** 였습니다. `NSString`, `NSArray`, `NSDictionary`... 전부 클래스였죠. 그런데 참조 타입은 **공유되는 가변 상태(shared mutable state)** 라는 골치 아픈 문제를 만듭니다. A가 넘겨준 객체를 B가 몰래 수정하면, A는 그 사실을 모른 채 변경된 데이터를 사용하게 되거든요.

Swift 팀은 이 문제를 해결하기 위해 **값 타입 중심 설계**를 택했습니다. WWDC 2015의 유명한 세션 "Protocol-Oriented Programming in Swift"에서 Dave Abrahams는 이렇게 선언했죠: **"Start with a struct."** 이 한마디가 Swift 커뮤니티의 설계 철학이 되었습니다.

실제로 Swift 표준 라이브러리에서 `Int`, `String`, `Array`, `Dictionary`, `Bool` 등 우리가 매일 쓰는 거의 모든 타입이 struct입니다. Swift 6에서는 Strict Concurrency 검사가 기본이 되면서, 값 타입의 장점이 더욱 부각되고 있어요. 값 타입은 복사되기 때문에 **여러 스레드에서 동시에 접근해도 안전**하거든요.

## 흔한 오해와 팁

> ⚠️ **흔한 오해**: "struct는 간단한 것, class는 복잡한 것에 쓴다" — 이건 Java 세계의 관습이고, Swift에서는 **반대**입니다. 복잡한 데이터 모델도 struct로 만들고, class는 참조 공유가 필요할 때만 씁니다. SwiftUI의 `View`도 struct인데, 절대 단순하지 않죠!

> 🔥 **실무 팁**: 동일한 클래스 인스턴스인지 확인하려면 `===` (동일성 연산자)를 사용하세요. `==`는 값이 같은지, `===`는 같은 객체인지를 비교합니다.

```swift
let a = Account(owner: "민수", balance: 1000)
let b = a
let c = Account(owner: "민수", balance: 1000)

print(a === b)   // true  — 같은 객체
print(a === c)   // false — 값은 같지만 다른 객체
```

```output
true
false
```

> 💡 **알고 계셨나요?**: Swift의 `String`은 struct입니다! C++이나 Java에서 문자열이 참조 타입인 것과 대조적이죠. 그래서 문자열을 함수에 넘겨도 원본이 변경될 걱정이 없습니다. 내부적으로는 Copy-On-Write 최적화로 성능도 챙겼어요.

## 핵심 정리

| 개념 | 설명 |
|------|------|
| **struct** | 값 타입. 대입 시 복사. 기본 선택 |
| **class** | 참조 타입. 대입 시 참조 공유. 상속 가능 |
| **값 타입** | 복사본이 독립적. 안전하지만 변경 시 mutating 필요 |
| **참조 타입** | 같은 객체를 공유. let이어도 프로퍼티 변경 가능 |
| **mutating** | 구조체 메서드에서 자신의 프로퍼티를 변경할 때 필요 |
| **===** | 두 참조가 같은 객체를 가리키는지 확인 (클래스만) |
| **memberwise init** | 구조체가 자동 생성하는 초기화 메서드 |

## 다음 섹션 미리보기

구조체와 클래스의 기본을 배웠으니, 이제 그 안을 채울 **프로퍼티와 메서드**를 더 깊이 알아봅시다. 연산 프로퍼티, 프로퍼티 감시자, 타입 메서드 등 [프로퍼티와 메서드](./02-properties-methods.md)에서 만나요.

## 참고 자료

- [Structures and Classes — The Swift Programming Language](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/classesandstructures/) - Swift 공식 문서의 구조체와 클래스 비교
- [Choosing Between Structures and Classes — Apple Developer](https://developer.apple.com/documentation/swift/choosing-between-structures-and-classes) - Apple의 struct vs class 선택 가이드
- [Value and Reference Types — Swift.org](https://www.swift.org/documentation/articles/value-and-reference-types.html) - 값 타입과 참조 타입 심화 설명
- [Protocol-Oriented Programming in Swift — WWDC 2015](https://developer.apple.com/videos/play/wwdc2015/408/) - "Start with a struct" 명언이 나온 세션
