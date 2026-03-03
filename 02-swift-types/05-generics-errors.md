# 제네릭과 에러 처리

> Generic 타입, Result, throw/try/catch

## 개요

지금까지 Swift의 타입 시스템을 차근차근 배워왔습니다. 이번 섹션은 Ch2의 마지막이자, Swift 타입 시스템의 완성편입니다. **제네릭(Generic)** 은 "어떤 타입이든 유연하게 다루는 코드"를 만들게 해주고, **에러 처리(Error Handling)** 는 "예상치 못한 상황에 우아하게 대처하는 방법"을 제공합니다. 이 둘을 알면 진짜 실전 코드를 작성할 준비가 된 겁니다.

**선수 지식**: [프로토콜과 익스텐션](./03-protocols-extensions.md), [열거형과 패턴 매칭](./04-enums-pattern.md)
**학습 목표**:
- 제네릭 함수와 제네릭 타입을 정의하고 사용한다
- 타입 제약(type constraint)을 활용한다
- throw/try/catch로 에러를 처리한다
- Result 타입의 용도와 사용법을 안다

## 왜 알아야 할까?

이미 제네릭을 쓰고 있었다는 사실, 알고 계셨나요? `Array<String>`, `Dictionary<String, Int>`, `Optional<Int>` — 이것들이 전부 제네릭입니다! 그리고 네트워크 요청, 파일 읽기, JSON 파싱 등 "실패할 수 있는 작업"을 할 때는 에러 처리가 필수죠. 이 두 가지를 모르면 실전 앱 코드를 읽는 것조차 어렵습니다.

## 핵심 개념

### 개념 1: 제네릭 — 타입을 매개변수로

> 💡 **비유**: 제네릭은 **만능 틀(몰드)** 입니다. 같은 틀인데, 초콜릿을 부으면 초콜릿 모양이 나오고, 젤리를 부으면 젤리 모양이 나오죠. 틀의 구조는 같지만 재료(타입)에 따라 결과가 달라집니다.

제네릭이 없다면, 타입마다 같은 함수를 반복 작성해야 합니다.

```swift
// 제네릭 없이 — 타입마다 함수를 따로 만들어야 합니다
func swapInts(_ a: inout Int, _ b: inout Int) {
    let temp = a; a = b; b = temp
}

func swapStrings(_ a: inout String, _ b: inout String) {
    let temp = a; a = b; b = temp
}

// 제네릭으로 — 한 번만 작성하면 모든 타입에 사용 가능!
func swapValues<T>(_ a: inout T, _ b: inout T) {
    let temp = a
    a = b
    b = temp
}

var x = 10, y = 20
swapValues(&x, &y)
print(x, y)   // 20 10

var s1 = "안녕", s2 = "세계"
swapValues(&s1, &s2)
print(s1, s2)  // "세계" "안녕"
```

```output
20 10
세계 안녕
```

`<T>`에서 `T`는 **타입 매개변수(Type Parameter)** 입니다. "어떤 타입이든 들어올 수 있다"는 뜻이에요. 함수가 호출될 때 T가 구체적인 타입으로 결정됩니다.

### 개념 2: 제네릭 타입 만들기

함수뿐 아니라 struct, class, enum도 제네릭으로 만들 수 있습니다.

```swift
// 제네릭 스택 (LIFO: Last In, First Out)
struct Stack<Element> {
    private var items: [Element] = []

    var isEmpty: Bool { items.isEmpty }
    var count: Int { items.count }
    var top: Element? { items.last }

    mutating func push(_ item: Element) {
        items.append(item)
    }

    mutating func pop() -> Element? {
        items.isEmpty ? nil : items.removeLast()
    }
}

// Int 스택
var numberStack = Stack<Int>()
numberStack.push(1)
numberStack.push(2)
numberStack.push(3)
print(numberStack.pop()!)   // 3 — 마지막에 넣은 것이 먼저 나옴

// String 스택 — 같은 코드로 다른 타입 사용
var stringStack = Stack<String>()
stringStack.push("A")
stringStack.push("B")
print(stringStack.top!)     // "B"
```

```output
3
B
```

### 개념 3: 타입 제약 (Type Constraint)

"아무 타입이나 다 되는 건 아니고, 특정 능력이 있는 타입만 허용"할 때 타입 제약을 사용합니다.

```swift
// Comparable을 따르는 타입만 허용 — 비교할 수 있어야 최솟값을 찾으니까요
func findMinimum<T: Comparable>(_ array: [T]) -> T? {
    guard var minimum = array.first else { return nil }
    for item in array {
        if item < minimum {
            minimum = item
        }
    }
    return minimum
}

print(findMinimum([3, 1, 4, 1, 5])!)      // 1
print(findMinimum(["바나나", "사과", "딸기"])!)  // "사과" (사전순)
```

```output
1
사과
```

```swift
// 여러 제약 조합도 가능
func printSorted<T: Comparable & CustomStringConvertible>(_ items: [T]) {
    let sorted = items.sorted()
    for item in sorted {
        print(item.description)
    }
}
```

> 💡 **알고 계셨나요?**: Swift 표준 라이브러리의 `Array`도 제네릭입니다! `[String]`은 사실 `Array<String>`의 줄임말이에요. `Dictionary<Key, Value>`, `Optional<Wrapped>`, `Result<Success, Failure>` 전부 제네릭 타입이죠. 제네릭이 없었다면 Swift는 완전히 다른 언어가 되었을 겁니다.

### 개념 4: 에러 처리 — throw / try / catch

> 💡 **비유**: 에러 처리는 **보험**입니다. 사고가 안 나면 좋지만, 혹시 사고가 나면 보험이 피해를 줄여주죠. 코드에서도 "실패할 수 있는 작업"에 대해 미리 대비해두는 겁니다.

```swift
// 1단계: 에러 타입 정의 (enum + Error 프로토콜)
enum LoginError: Error {
    case emptyUsername
    case emptyPassword
    case invalidCredentials
    case networkError(message: String)
}

// 2단계: 에러를 던질 수 있는 함수 (throws)
func login(username: String, password: String) throws -> String {
    guard !username.isEmpty else {
        throw LoginError.emptyUsername
    }
    guard !password.isEmpty else {
        throw LoginError.emptyPassword
    }
    guard username == "admin" && password == "1234" else {
        throw LoginError.invalidCredentials
    }

    return "토큰-ABC123"    // 성공 시 토큰 반환
}

// 3단계: 에러 처리 (do-try-catch)
do {
    let token = try login(username: "admin", password: "1234")
    print("✅ 로그인 성공! 토큰: \(token)")
} catch LoginError.emptyUsername {
    print("❌ 아이디를 입력해주세요")
} catch LoginError.emptyPassword {
    print("❌ 비밀번호를 입력해주세요")
} catch LoginError.invalidCredentials {
    print("❌ 아이디 또는 비밀번호가 잘못되었습니다")
} catch {
    print("❌ 알 수 없는 에러: \(error)")
}
```

```output
✅ 로그인 성공! 토큰: 토큰-ABC123
```

에러를 처리하는 세 가지 방법:

```swift
// 방법 1: do-try-catch — 에러를 직접 처리
do {
    let token = try login(username: "", password: "1234")
    print(token)
} catch {
    print("에러 발생: \(error)")
}

// 방법 2: try? — 에러 시 nil 반환 (옵셔널)
let token = try? login(username: "admin", password: "1234")
print(token ?? "로그인 실패")   // "토큰-ABC123"
```

```output
에러 발생: emptyUsername
토큰-ABC123
```

```swift
// 방법 3: try! — 에러 시 크래시 (100% 확신할 때만!)
// let token = try! login(username: "", password: "")  // 💥 크래시!
```

> ⚠️ **흔한 오해**: "`try?`를 쓰면 에러가 무시되니까 나쁜 거 아닌가요?" — 상황에 따라 다릅니다. 에러 원인을 알 필요 없이 "성공/실패"만 구분하면 될 때는 `try?`가 가장 깔끔해요. 예를 들어 캐시 저장이 실패해도 앱이 계속 동작해야 하는 경우죠.

### 개념 5: Result 타입 — 성공과 실패를 값으로

`Result`는 [앞서 배운 enum](./04-enums-pattern.md)으로 만들어진 타입입니다. 에러를 **값으로 다룰 수 있게** 해줍니다.

```swift
enum DataError: Error {
    case notFound
    case invalidFormat
    case serverError(code: Int)
}

// Result를 반환하는 함수
func fetchUserName(id: Int) -> Result<String, DataError> {
    if id <= 0 {
        return .failure(.notFound)
    }
    if id > 1000 {
        return .failure(.serverError(code: 500))
    }
    return .success("사용자\(id)")
}

// Result 사용하기
let result = fetchUserName(id: 42)

switch result {
case .success(let name):
    print("✅ 이름: \(name)")
case .failure(let error):
    switch error {
    case .notFound:
        print("❌ 사용자를 찾을 수 없습니다")
    case .invalidFormat:
        print("❌ 잘못된 형식입니다")
    case .serverError(let code):
        print("❌ 서버 에러 (\(code))")
    }
}

// get()으로 throws 스타일로도 사용 가능
do {
    let name = try result.get()
    print(name)
} catch {
    print("에러: \(error)")
}
```

```output
✅ 이름: 사용자42
사용자42
```

## 실습: 직접 해보기

제네릭과 에러 처리를 결합한 간단한 데이터 저장소를 만들어 봅시다.

```swift
import Foundation

// 저장소 에러 타입
enum StorageError: Error {
    case keyNotFound(key: String)
    case typeMismatch(expected: String, actual: String)
    case storageFull(maxCapacity: Int)
}

// 제네릭 저장소
struct TypedStorage {
    private var storage: [String: Any] = [:]
    let maxCapacity: Int

    init(maxCapacity: Int = 100) {
        self.maxCapacity = maxCapacity
    }

    // 제네릭 메서드: 값 저장
    mutating func save<T>(_ value: T, forKey key: String) throws {
        guard storage.count < maxCapacity else {
            throw StorageError.storageFull(maxCapacity: maxCapacity)
        }
        storage[key] = value
        print("💾 저장: [\(key)] = \(value)")
    }

    // 제네릭 메서드: 값 로드 (타입 안전하게)
    func load<T>(key: String, as type: T.Type) throws -> T {
        guard let value = storage[key] else {
            throw StorageError.keyNotFound(key: key)
        }
        guard let typed = value as? T else {
            throw StorageError.typeMismatch(
                expected: String(describing: T.self),
                actual: String(describing: Swift.type(of: value))
            )
        }
        return typed
    }

    var count: Int { storage.count }
}

// 사용 예시
var store = TypedStorage(maxCapacity: 5)

do {
    // 다양한 타입 저장
    try store.save("민수", forKey: "name")
    try store.save(25, forKey: "age")
    try store.save(["Swift", "SwiftUI"], forKey: "skills")

    // 타입 안전하게 로드
    let name: String = try store.load(key: "name", as: String.self)
    let age: Int = try store.load(key: "age", as: Int.self)
    let skills: [String] = try store.load(key: "skills", as: [String].self)

    print("\n📋 사용자 정보:")
    print("  이름: \(name)")
    print("  나이: \(age)")
    print("  스킬: \(skills.joined(separator: ", "))")

    // 없는 키 접근 시 에러
    let _ = try store.load(key: "address", as: String.self)

} catch StorageError.keyNotFound(let key) {
    print("\n❌ '\(key)' 키를 찾을 수 없습니다")
} catch StorageError.typeMismatch(let expected, let actual) {
    print("\n❌ 타입 불일치: \(expected) 기대, \(actual) 실제")
} catch StorageError.storageFull(let max) {
    print("\n❌ 저장소가 가득 찼습니다 (최대 \(max)개)")
} catch {
    print("\n❌ 예상치 못한 에러: \(error)")
}
```

```output
💾 저장: [name] = 민수
💾 저장: [age] = 25
💾 저장: [skills] = ["Swift", "SwiftUI"]

📋 사용자 정보:
  이름: 민수
  나이: 25
  스킬: Swift, SwiftUI

❌ 'address' 키를 찾을 수 없습니다
```

## 더 깊이 알아보기

### Swift 6의 Typed Throws

Swift 6에서는 **Typed Throws** 기능이 도입되었습니다. 기존에는 `throws`만 표시하면 어떤 에러든 던질 수 있었는데, 이제 어떤 에러 타입을 던지는지 명시할 수 있어요.

```swift
// Swift 6: typed throws
func parseAge(_ text: String) throws(ValidationError) -> Int {
    guard let age = Int(text) else {
        throw .invalidFormat
    }
    guard age >= 0 else {
        throw .negativeValue
    }
    return age
}

enum ValidationError: Error {
    case invalidFormat
    case negativeValue
}
```

이 기능은 Swift Evolution 프로포절 SE-0413으로 제안되어 Swift 6에 포함되었습니다. 에러 처리를 더 정밀하게 할 수 있게 되었지만, 기존의 `throws`도 여전히 사용 가능하니 당장 모든 코드를 바꿀 필요는 없어요.

### 제네릭의 탄생 배경

제네릭은 1970년대 **ML 언어**에서 처음 등장한 **파라메트릭 다형성(parametric polymorphism)** 이 뿌리입니다. 이후 C++의 템플릿, Java의 제네릭을 거쳐 Swift에 이르렀죠. Swift의 제네릭은 C++처럼 컴파일 시점에 특수화(specialization)되어 성능이 좋고, Java처럼 사용하기 쉬우면서도, **프로토콜 제약**으로 타입 안전성까지 보장하는 "좋은 점만 모은" 설계입니다.

## 흔한 오해와 팁

> ⚠️ **흔한 오해**: "제네릭은 고급 기능이니까 나중에 배워도 된다" — 이미 쓰고 있습니다! `[String]`은 `Array<String>`, `String?`은 `Optional<String>`이에요. 직접 만드는 건 나중이더라도, 읽을 줄은 알아야 합니다.

> 🔥 **실무 팁**: 에러 처리에서 `catch` 순서가 중요합니다. **구체적인 에러를 먼저**, 일반적인 에러를 나중에 처리하세요. `catch { }`(모든 에러)를 맨 위에 놓으면 아래 catch가 실행되지 않습니다.

> 💡 **알고 계셨나요?**: Swift에서 `<T>`의 `T`는 관례일 뿐, 아무 이름이나 쓸 수 있습니다. 하지만 관례로 `T`(Type), `U`(또 다른 Type), `E`(Element), `K`(Key), `V`(Value)를 자주 사용해요. Apple의 공식 코드도 이 관례를 따릅니다.

## 핵심 정리

| 개념 | 설명 |
|------|------|
| **제네릭 `<T>`** | 타입을 매개변수로 받아 유연한 코드 작성 |
| **타입 제약** | `<T: Comparable>` — 특정 프로토콜을 따르는 타입만 허용 |
| **throws** | "이 함수는 에러를 던질 수 있다"는 선언 |
| **throw** | 실제로 에러를 던지는 키워드 |
| **do-try-catch** | 에러를 잡아서 처리하는 구문 |
| **try?** | 에러 시 nil 반환 (옵셔널) |
| **try!** | 에러 시 크래시 (확신할 때만 사용) |
| **Result** | `.success(값)` 또는 `.failure(에러)`를 담는 enum |
| **Error 프로토콜** | 에러 타입이 채택해야 하는 프로토콜 |

## 다음 섹션 미리보기

Ch2 "Swift 타입 시스템"을 모두 마쳤습니다! 구조체, 클래스, 프로토콜, 열거형, 제네릭, 에러 처리까지 — Swift의 타입 시스템을 완전히 익혔어요. 이제 드디어 화면을 만들 차례입니다! 다음 챕터 [Ch3. SwiftUI 첫걸음](../03-swiftui-start/01-hello-swiftui.md)에서 `View` 프로토콜, `body` 프로퍼티, `#Preview` 매크로를 만나봅시다. 지금까지 배운 모든 것이 SwiftUI에서 빛을 발하게 됩니다!

## 참고 자료

- [Generics — The Swift Programming Language](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/generics/) - Swift 공식 문서의 제네릭 설명
- [Error Handling — The Swift Programming Language](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/errorhandling/) - 에러 처리 공식 가이드
- [SE-0413: Typed Throws](https://github.com/swiftlang/swift-evolution/blob/main/proposals/0413-typed-throws.md) - Swift 6의 Typed Throws 프로포절
- [Embrace Swift Generics — WWDC 2022](https://developer.apple.com/videos/play/wwdc2022/110352/) - some, any, 제네릭의 관계를 설명하는 세션
