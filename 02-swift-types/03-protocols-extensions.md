# 프로토콜과 익스텐션

> protocol, extension, 프로토콜 지향 프로그래밍

## 개요

객체지향 프로그래밍에서는 "이 객체가 무엇인가?"(상속)에 집중하지만, Swift는 **"이 타입이 무엇을 할 수 있는가?"**(프로토콜)에 집중합니다. **프로토콜(Protocol)** 은 타입이 갖춰야 할 능력을 정의하고, **익스텐션(Extension)** 은 기존 타입에 새로운 기능을 추가합니다. 이 두 가지를 합치면 Swift의 핵심 철학인 **프로토콜 지향 프로그래밍(Protocol-Oriented Programming)** 이 완성됩니다.

**선수 지식**: [구조체와 클래스](./01-struct-class.md), [프로퍼티와 메서드](./02-properties-methods.md)
**학습 목표**:
- 프로토콜을 정의하고 채택하는 방법을 익힌다
- 익스텐션으로 기존 타입을 확장하는 방법을 안다
- 프로토콜 기본 구현(default implementation)을 이해한다
- 프로토콜 지향 프로그래밍의 장점을 파악한다

## 왜 알아야 할까?

SwiftUI의 `View` 프로토콜, SwiftData의 `@Model`, `Codable`, `Hashable`, `Equatable`... Swift에서 마주치는 거의 모든 중요한 개념이 프로토콜입니다. 프로토콜을 모르면 SwiftUI의 `struct ContentView: View`가 왜 그렇게 생겼는지조차 이해할 수 없어요. Swift를 쓴다면 프로토콜은 **반드시** 알아야 합니다.

## 핵심 개념

### 개념 1: 프로토콜 — 능력의 약속

> 💡 **비유**: 프로토콜은 **자격증**입니다. "운전면허가 있다"고 하면, 그 사람이 남자든 여자든, 학생이든 직장인이든 상관없이 "운전할 수 있다"는 능력이 보장되죠. 프로토콜도 마찬가지로, 타입이 특정 능력을 갖추고 있음을 보증합니다.

```run:swift
// 프로토콜 정의 — "이런 것들을 할 수 있어야 한다"는 약속
protocol Describable {
    var description: String { get }     // 읽기 가능한 프로퍼티
    func summarize() -> String          // 메서드
}

// struct가 프로토콜을 채택(conform)
struct Book: Describable {
    var title: String
    var author: String
    var pages: Int

    // 프로토콜이 요구한 프로퍼티 구현
    var description: String {
        "『\(title)』 — \(author) 지음"
    }

    // 프로토콜이 요구한 메서드 구현
    func summarize() -> String {
        "\(title) (\(pages)쪽)"
    }
}

struct Movie: Describable {
    var title: String
    var director: String
    var runtime: Int

    var description: String {
        "🎬 \(title) — \(director) 감독"
    }

    func summarize() -> String {
        "\(title) (\(runtime)분)"
    }
}

// 프로토콜 타입으로 다양한 타입을 동일하게 다룰 수 있습니다
let items: [any Describable] = [
    Book(title: "Swift 마스터", author: "김스위프트", pages: 400),
    Movie(title: "인터스텔라", director: "크리스토퍼 놀란", runtime: 169)
]

for item in items {
    print(item.description)
}
```

```output
『Swift 마스터』 — 김스위프트 지음
🎬 인터스텔라 — 크리스토퍼 놀란 감독
```

> ⚠️ **흔한 오해**: "프로토콜은 Java의 인터페이스와 같다" — 비슷하지만, Swift 프로토콜은 **기본 구현**을 가질 수 있고, **값 타입(struct, enum)** 에도 적용되며, **연관 타입(associated type)** 같은 고급 기능도 있어요. 훨씬 강력합니다.

### 개념 2: 익스텐션 — 기존 타입 확장하기

> 💡 **비유**: 익스텐션은 **리모델링**입니다. 이미 지어진 집(기존 타입)에 방을 추가하거나 새 기능을 덧붙이는 것처럼, 원래 코드를 수정하지 않고 기능을 확장합니다.

```run:swift
// Int에 새로운 기능 추가
extension Int {
    var isEven: Bool {
        self % 2 == 0
    }

    var squared: Int {
        self * self
    }

    func times(_ action: () -> Void) {
        for _ in 0..<self {
            action()
        }
    }
}

// 이제 모든 Int에서 사용 가능!
print(42.isEven)      // true
print(5.squared)      // 25

3.times {
    print("안녕!")     // "안녕!" 3번 출력
}
```

```output
true
25
안녕!
안녕!
안녕!
```

```run:swift
// String에 유용한 기능 추가
extension String {
    var trimmed: String {
        self.trimmingCharacters(in: .whitespacesAndNewlines)
    }

    var isValidEmail: Bool {
        self.contains("@") && self.contains(".")
    }
}

let email = "  user@email.com  "
print(email.trimmed)               // "user@email.com"
print(email.trimmed.isValidEmail)  // true
```

```output
user@email.com
true
```

### 개념 3: 프로토콜 기본 구현 — 익스텐션의 진짜 힘

프로토콜과 익스텐션을 결합하면, 프로토콜에 **기본 구현(default implementation)** 을 제공할 수 있습니다. 이게 Swift 프로토콜의 진짜 무기입니다.

```run:swift
protocol Greetable {
    var name: String { get }
    func greet() -> String
}

// 프로토콜 익스텐션으로 기본 구현 제공
extension Greetable {
    func greet() -> String {
        "안녕하세요, \(name)입니다!"    // 기본 인사말
    }
}

struct Student: Greetable {
    var name: String
    // greet()를 구현하지 않아도 됩니다 — 기본 구현이 있으니까!
}

struct Teacher: Greetable {
    var name: String

    // 필요하면 기본 구현을 오버라이드할 수 있습니다
    func greet() -> String {
        "안녕하세요, \(name) 선생님입니다."
    }
}

let student = Student(name: "민수")
let teacher = Teacher(name: "김영희")

print(student.greet())   // "안녕하세요, 민수입니다!" — 기본 구현 사용
print(teacher.greet())   // "안녕하세요, 김영희 선생님입니다." — 커스텀 구현
```

```output
안녕하세요, 민수입니다!
안녕하세요, 김영희 선생님입니다.
```

### 개념 4: Swift의 대표적인 프로토콜들

Swift 표준 라이브러리에는 매우 유용한 프로토콜들이 내장되어 있습니다.

```run:swift
// Equatable — == 비교 가능
struct Point: Equatable {
    var x: Double
    var y: Double
    // 모든 프로퍼티가 Equatable이면 자동 구현!
}

let p1 = Point(x: 1, y: 2)
let p2 = Point(x: 1, y: 2)
print(p1 == p2)   // true — 자동 비교
```

```output
true
```

```run:swift
// Comparable — <, >, <=, >= 비교 가능
struct Score: Comparable {
    var value: Int
    var name: String

    static func < (lhs: Score, rhs: Score) -> Bool {
        lhs.value < rhs.value
    }
}

let scores = [Score(value: 85, name: "민수"), Score(value: 92, name: "지영")]
let sorted = scores.sorted()    // value 기준 정렬됨

// Codable — JSON 변환 가능 (매우 중요!)
struct User: Codable {
    var name: String
    var age: Int
    var email: String
}

let user = User(name: "민수", age: 25, email: "minsu@email.com")
let jsonData = try! JSONEncoder().encode(user)
let jsonString = String(data: jsonData, encoding: .utf8)!
print(jsonString)  // {"name":"민수","age":25,"email":"minsu@email.com"}
```

```output
{"name":"민수","age":25,"email":"minsu@email.com"}
```

> 💡 **알고 계셨나요?**: `Equatable`, `Hashable`, `Codable`은 조건이 맞으면 Swift가 **자동으로 구현**을 생성해줍니다. 모든 저장 프로퍼티가 해당 프로토콜을 따르면, 직접 코드를 작성할 필요가 없어요. 이것을 **자동 합성(automatic synthesis)** 이라고 합니다.

### 개념 5: 프로토콜 조합과 any / some

여러 프로토콜을 동시에 요구하거나, 프로토콜 타입을 다룰 때의 문법입니다.

```run:swift
protocol Identifiable {
    var id: String { get }
}

protocol Displayable {
    var displayName: String { get }
}

// 프로토콜 조합 — 여러 프로토콜을 동시에 요구
func showProfile(item: any Identifiable & Displayable) {
    print("[\(item.id)] \(item.displayName)")
}

struct Member: Identifiable, Displayable {
    var id: String
    var displayName: String
}

showProfile(item: Member(id: "001", displayName: "민수"))
```

```output
[001] 민수
```

```swift
// some — 불투명 타입 (SwiftUI에서 매일 만나게 됩니다)
func makeGreeting() -> some Describable {
    Book(title: "Hello", author: "World", pages: 1)
}
// some Describable = "어떤 Describable인지는 컴파일러만 알고, 호출자는 모른다"
```

> 🔥 **실무 팁**: Swift 6에서는 `any` 키워드가 더 중요해졌습니다. 프로토콜을 타입으로 사용할 때 `any Describable`처럼 명시적으로 써야 합니다. `some`은 SwiftUI의 `var body: some View`에서 매일 만나게 돼요.

## 실습: 직접 해보기

프로토콜을 활용한 간단한 도형 계산기를 만들어 봅시다.

```run:swift
import Foundation

// 도형 프로토콜 정의
protocol Shape {
    var name: String { get }
    func area() -> Double
    func perimeter() -> Double
}

// 프로토콜 기본 구현
extension Shape {
    func printInfo() {
        print("📐 \(name)")
        print("   넓이: \(String(format: "%.1f", area()))")
        print("   둘레: \(String(format: "%.1f", perimeter()))")
    }
}

// 원
struct CircleShape: Shape {
    var radius: Double
    var name: String { "원 (반지름 \(radius))" }

    func area() -> Double {
        Double.pi * radius * radius
    }

    func perimeter() -> Double {
        2 * Double.pi * radius
    }
}

// 직사각형
struct RectangleShape: Shape {
    var width: Double
    var height: Double
    var name: String { "직사각형 (\(width) × \(height))" }

    func area() -> Double {
        width * height
    }

    func perimeter() -> Double {
        2 * (width + height)
    }
}

// 정삼각형
struct TriangleShape: Shape {
    var side: Double
    var name: String { "정삼각형 (한 변 \(side))" }

    func area() -> Double {
        (sqrt(3) / 4) * side * side
    }

    func perimeter() -> Double {
        side * 3
    }
}

// 프로토콜 타입 배열로 다양한 도형을 동일하게 처리
let shapes: [any Shape] = [
    CircleShape(radius: 5),
    RectangleShape(width: 10, height: 5),
    TriangleShape(side: 6)
]

print("🔷 도형 정보")
print("────────────────")
for shape in shapes {
    shape.printInfo()    // 기본 구현 사용
    print()
}

// 넓이 기준 정렬
let sortedByArea = shapes.sorted { $0.area() > $1.area() }
print("📊 넓이 순위:")
for (i, shape) in sortedByArea.enumerated() {
    print("  \(i + 1)위: \(shape.name) — \(String(format: "%.1f", shape.area()))")
}
```

```output
🔷 도형 정보
────────────────
📐 원 (반지름 5.0)
   넓이: 78.5
   둘레: 31.4

📐 직사각형 (10.0 × 5.0)
   넓이: 50.0
   둘레: 30.0

📐 정삼각형 (한 변 6.0)
   넓이: 15.6
   둘레: 18.0

📊 넓이 순위:
  1위: 원 (반지름 5.0) — 78.5
  2위: 직사각형 (10.0 × 5.0) — 50.0
  3위: 정삼각형 (한 변 6.0) — 15.6
```

## 더 깊이 알아보기

### "프로토콜 지향 프로그래밍"의 탄생

2015년 WWDC에서 Apple의 Dave Abrahams가 발표한 **"Protocol-Oriented Programming in Swift"** 세션은 Swift 커뮤니티에 큰 파장을 일으켰습니다. 그는 "Swift is a protocol-oriented programming language"라고 선언하며, 클래스 상속 중심의 전통적인 객체지향 프로그래밍의 한계를 지적했어요.

상속의 문제점은 뭘까요? **다이아몬드 문제**(두 부모로부터 같은 메서드를 상속받으면?), **깊은 상속 계층**(할아버지 클래스를 수정하면 모든 후손이 영향받음), **단일 상속 제한**(한 번에 하나만 상속 가능) 등이 있죠. 프로토콜은 이 모든 문제를 우아하게 해결합니다. 타입은 여러 프로토콜을 동시에 채택할 수 있고, 프로토콜 익스텐션으로 코드를 재사용할 수 있으니까요.

이 철학 덕분에 SwiftUI는 `View` **프로토콜**을 기반으로 설계되었고, struct가 화면을 구성하는 기본 단위가 되었습니다.

## 흔한 오해와 팁

> ⚠️ **흔한 오해**: "익스텐션에서 저장 프로퍼티를 추가할 수 있다" — 불가능합니다! 익스텐션에서는 **연산 프로퍼티와 메서드만** 추가할 수 있어요. 저장 프로퍼티는 타입의 메모리 레이아웃을 바꿔야 하기 때문에 원래 정의에서만 추가할 수 있습니다.

> 🔥 **실무 팁**: 코드를 정리할 때 익스텐션을 "기능별 그룹"으로 활용하세요. 프로토콜 채택을 별도의 extension으로 분리하면 가독성이 크게 올라갑니다.

```swift
struct Player {
    var name: String
    var score: Int
}

// 프로토콜 채택을 extension으로 분리
extension Player: CustomStringConvertible {
    var description: String {
        "\(name) (\(score)점)"
    }
}

extension Player: Comparable {
    static func < (lhs: Player, rhs: Player) -> Bool {
        lhs.score < rhs.score
    }
}
```

## 핵심 정리

| 개념 | 설명 |
|------|------|
| **protocol** | 타입이 갖춰야 할 능력(프로퍼티, 메서드)을 정의하는 약속 |
| **extension** | 기존 타입에 새 기능(연산 프로퍼티, 메서드)을 추가 |
| **기본 구현** | 프로토콜 익스텐션으로 제공하는 기본 동작 |
| **any** | 프로토콜을 타입으로 사용할 때 명시 (Swift 6) |
| **some** | 불투명 반환 타입. `some View`처럼 사용 |
| **Equatable** | `==` 비교를 가능하게 하는 프로토콜 |
| **Codable** | JSON 등의 인코딩/디코딩을 가능하게 하는 프로토콜 |
| **자동 합성** | 조건 충족 시 컴파일러가 프로토콜 구현을 자동 생성 |

## 다음 섹션 미리보기

프로토콜로 타입의 능력을 정의하는 법을 배웠습니다. 다음은 Swift에서 가장 강력한 타입 중 하나인 **열거형(enum)** 과, 이를 활용한 **패턴 매칭**입니다. [열거형과 패턴 매칭](./04-enums-pattern.md)에서 Swift가 왜 "switch의 천국"이라 불리는지 직접 확인해 보세요.

## 참고 자료

- [Protocols — The Swift Programming Language](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/protocols/) - Swift 공식 문서의 프로토콜 설명
- [Extensions — The Swift Programming Language](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/extensions/) - 익스텐션 공식 문서
- [Protocol-Oriented Programming in Swift — WWDC 2015](https://developer.apple.com/videos/play/wwdc2015/408/) - POP의 시작점이 된 전설적인 세션
- [Protocols and Generics — WWDC 2022](https://developer.apple.com/videos/play/wwdc2022/110353/) - some과 any 키워드의 사용법
