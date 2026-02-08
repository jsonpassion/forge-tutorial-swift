# 함수와 클로저

> func 정의, 클로저 표현식, map/filter/reduce

## 개요

같은 코드를 여러 번 복사해서 붙여넣고 있다면, 그건 **함수(Function)** 로 만들어야 한다는 신호입니다. 함수는 코드를 재사용 가능한 단위로 묶어주고, **클로저(Closure)** 는 함수를 더 유연하게 사용하게 해줍니다. Swift에서 클로저는 놀라울 정도로 자주 등장하니, 이 섹션을 잘 익혀두면 앞으로의 학습이 훨씬 수월해집니다.

**선수 지식**: [조건문과 반복문](./04-control-flow.md)까지의 기본 문법
**학습 목표**:
- 함수를 정의하고 호출하는 방법을 익힌다
- 매개변수 레이블, 기본값, 반환 타입을 활용한다
- 클로저의 개념과 축약 문법을 이해한다
- `map`, `filter`, `reduce`로 컬렉션을 함수형으로 다룬다

## 왜 알아야 할까?

SwiftUI에서 버튼을 만들 때 `Button("탭하세요") { /* 여기 */ }` 이런 코드를 보게 될 텐데, 중괄호 `{ }` 안의 코드가 바로 **클로저**입니다. SwiftUI는 클로저를 매우 많이 사용하기 때문에, 클로저를 이해하지 못하면 SwiftUI 코드가 전부 암호처럼 보일 수 있어요. 지금 확실히 잡아두면 나중에 감사할 겁니다!

## 핵심 개념

### 개념 1: 함수 기본

> 💡 **비유**: 함수는 **자판기**입니다. 돈(매개변수)을 넣고 버튼을 누르면, 정해진 과정을 거쳐 음료(반환값)가 나옵니다. 한번 만들어 놓으면 누구든 같은 방식으로 사용할 수 있죠.

```swift
// 기본 함수 정의
func greet(name: String) -> String {
    return "안녕하세요, \(name)님!"
}

// 함수 호출
let message = greet(name: "민수")
print(message)  // "안녕하세요, 민수님!"

// 반환값이 없는 함수
func printStars(count: Int) {
    let stars = String(repeating: "⭐", count: count)
    print(stars)
}
printStars(count: 5)  // "⭐⭐⭐⭐⭐"
```

### 개념 2: 매개변수 레이블

Swift 함수의 특별한 점은 **매개변수에 레이블(label)** 이 있다는 거예요. 이 덕분에 함수를 호출할 때 "무엇을 넘기는지" 바로 알 수 있습니다.

```swift
// 외부 레이블과 내부 이름을 따로 지정
func sendMessage(to recipient: String, content message: String) {
    print("\(recipient)에게: \(message)")
}
// 호출할 때는 외부 레이블을 사용 — 영어 문장처럼 자연스럽게 읽힙니다
sendMessage(to: "지영", content: "내일 회의 있어요")

// 레이블 생략 — 밑줄(_)을 사용
func add(_ a: Int, _ b: Int) -> Int {
    return a + b
}
let sum = add(3, 5)    // 레이블 없이 호출

// 기본값이 있는 매개변수
func orderCoffee(menu: String, size: String = "Regular") {
    print("\(size) \(menu) 주문 완료!")
}
orderCoffee(menu: "아메리카노")                // "Regular 아메리카노 주문 완료!"
orderCoffee(menu: "라떼", size: "Grande")      // "Grande 라떼 주문 완료!"
```

> 💡 **알고 계셨나요?**: Swift의 매개변수 레이블은 **Objective-C**의 전통에서 왔습니다. Objective-C는 메서드 이름이 매우 길어서 `[string insertString:aString atIndex:loc]` 같은 형태였는데, Swift는 이 "읽히는 코드" 철학을 이어받으면서도 더 깔끔하게 만들었어요.

### 개념 3: 클로저 — 이름 없는 함수

> 💡 **비유**: 함수가 **가게 간판**이 있는 정식 식당이라면, 클로저는 **푸드트럭**입니다. 이름 없이도 바로 그 자리에서 음식을 만들어 내죠. 필요한 곳에 즉석으로 정의해서 쓸 수 있습니다.

클로저(Closure)는 **이름 없이 정의하는 함수**입니다. 코드 블록을 변수에 저장하거나, 다른 함수에 전달할 수 있어요.

```swift
// 일반 함수
func square(number: Int) -> Int {
    return number * number
}

// 같은 기능의 클로저
let squareClosure = { (number: Int) -> Int in
    return number * number
}

print(square(number: 5))       // 25
print(squareClosure(5))        // 25
```

Swift에서는 클로저를 **점점 짧게 줄여 쓸 수 있습니다**. 이 축약 과정을 알아두면 다른 사람의 코드를 읽을 때 큰 도움이 됩니다.

```swift
let numbers = [5, 2, 8, 1, 9, 3]

// 1단계: 전체 클로저 문법
let sorted1 = numbers.sorted(by: { (a: Int, b: Int) -> Bool in
    return a < b
})

// 2단계: 타입 추론 — Swift가 타입을 알아서 추론
let sorted2 = numbers.sorted(by: { a, b in
    return a < b
})

// 3단계: 암시적 반환 — 한 줄이면 return 생략
let sorted3 = numbers.sorted(by: { a, b in a < b })

// 4단계: 단축 인자 — $0, $1로 매개변수 대체
let sorted4 = numbers.sorted(by: { $0 < $1 })

// 5단계: 후행 클로저 — 마지막 매개변수가 클로저면 괄호 밖으로
let sorted5 = numbers.sorted { $0 < $1 }
```

이 5단계가 모두 **동일한 결과**를 냅니다! 처음에는 1~2단계로 쓰다가, 익숙해지면 4~5단계로 줄여 쓰는 게 Swift의 관례예요.

> ⚠️ **흔한 오해**: "`$0`, `$1`이 대체 뭔가요? 너무 마법 같아요" — 전혀 마법이 아닙니다! `$0`은 첫 번째 매개변수, `$1`은 두 번째 매개변수를 가리키는 Swift의 **단축 인자 이름(Shorthand Argument Names)** 일 뿐이에요. 처음에는 명시적으로 `a, b in` 형태를 쓰고, 익숙해지면 `$0`, `$1`을 쓰세요.

### 개념 4: map, filter, reduce — 함수형 트리오

이 세 가지 함수는 컬렉션을 다루는 가장 우아한 방법입니다. for-in 반복문 없이도 강력한 데이터 처리가 가능해요.

```swift
let numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]

// map: 모든 요소를 변환 — "각각에 대해 이렇게 바꿔줘"
let doubled = numbers.map { $0 * 2 }
print(doubled)      // [2, 4, 6, 8, 10, 12, 14, 16, 18, 20]

let labels = numbers.map { "\($0)번" }
print(labels)       // ["1번", "2번", ... "10번"]

// filter: 조건에 맞는 요소만 선택 — "이 조건을 만족하는 것만 골라줘"
let evens = numbers.filter { $0 % 2 == 0 }
print(evens)        // [2, 4, 6, 8, 10]

// reduce: 모든 요소를 하나로 합침 — "전부 합쳐서 하나로 만들어줘"
let total = numbers.reduce(0) { $0 + $1 }
print(total)        // 55 (1+2+3+...+10)

// 체이닝: 여러 개를 연결해서 사용
let result = numbers
    .filter { $0 % 2 == 0 }   // 짝수만
    .map { $0 * $0 }           // 제곱
    .reduce(0, +)              // 전부 합산
print(result)       // 220 (4+16+36+64+100)
```

> 💡 **비유**: `map`은 **변환 공장** (원재료 → 제품), `filter`는 **품질 검사** (합격품만 통과), `reduce`는 **포장** (여러 개를 하나로 묶는 것)입니다.

## 실습: 직접 해보기

실생활에 가까운 예제를 만들어 봅시다.

```swift
import Foundation

// 쇼핑몰 상품 데이터
let products = [
    ("맥북 프로", 2990000),
    ("아이패드", 1290000),
    ("에어팟", 249000),
    ("아이폰 케이스", 49000),
    ("Apple Watch", 599000),
    ("매직 키보드", 449000)
]

// 1. map으로 가격에 부가세 10% 추가
let withTax = products.map { (name, price) in
    (name, Int(Double(price) * 1.1))
}
print("📦 부가세 포함 가격:")
for (name, price) in withTax {
    print("  \(name): \(price)원")
}

// 2. filter로 50만원 이하 상품만 골라내기
let affordable = products.filter { $0.1 <= 500000 }
print("\n💰 50만원 이하:")
for (name, price) in affordable {
    print("  \(name): \(price)원")
}

// 3. reduce로 전체 합계 계산
let totalPrice = products.reduce(0) { $0 + $1.1 }
print("\n🧮 전체 합계: \(totalPrice)원")

// 4. 함수를 조합한 할인 계산기
func applyDiscount(rate: Double) -> (Int) -> Int {
    return { price in
        Int(Double(price) * (1.0 - rate))
    }
}

let tenPercentOff = applyDiscount(rate: 0.1)
print("\n🏷️ 10% 할인 적용:")
for (name, price) in products {
    print("  \(name): \(price)원 → \(tenPercentOff(price))원")
}
```

## 더 깊이 알아보기

### 함수는 일급 객체 (First-Class Citizen)

Swift에서 함수는 **일급 객체**입니다. 이게 무슨 뜻이냐면, 함수를 변수에 저장하고, 다른 함수에 넘기고, 함수의 결과로 반환할 수 있다는 뜻이에요. 사실 Swift에서 **클로저와 함수는 같은 것**입니다. 함수는 이름이 있는 클로저일 뿐이죠.

이 개념은 Haskell, Lisp 같은 함수형 프로그래밍 언어에서 온 것인데, Swift가 이를 실용적으로 잘 녹여내서 모던 프로그래밍의 장점을 살리고 있습니다. 특히 SwiftUI에서 `Button`, `ForEach`, `sheet` 등이 모두 클로저를 매개변수로 받는 구조라서, 이 개념이 정말 중요해요.

## 흔한 오해와 팁

> ⚠️ **흔한 오해**: "클로저는 고급 기능이니까 나중에 배워도 된다" — SwiftUI를 쓰는 순간부터 클로저를 매일 만나게 됩니다. `Button { }`, `List { }`, `.onAppear { }` 모두 클로저예요. 기본은 지금 꼭 잡아두세요.

> 🔥 **실무 팁**: `map`/`filter`/`reduce`가 for-in보다 항상 좋은 건 아닙니다. 로직이 복잡하면 for-in이 읽기 쉬울 수 있어요. **"한 줄로 쓸 수 있는가?"** 를 기준으로, 간단한 변환은 `map`/`filter`, 복잡한 로직은 for-in을 쓰세요.

> 💡 **알고 계셨나요?**: `reduce(0, +)` 에서 `+`는 사실 함수입니다! Swift에서 연산자도 함수이기 때문에, `+`만 넘기면 "두 값을 더해주는 함수"로 동작합니다. `sorted(by: <)` 도 같은 원리예요.

## 핵심 정리

| 개념 | 설명 |
|------|------|
| **func** | 함수를 정의하는 키워드. `func 이름(매개변수) -> 반환타입` |
| **매개변수 레이블** | 호출 시 가독성을 높이는 외부 이름. `_`로 생략 가능 |
| **기본값** | 매개변수에 기본값을 지정. 호출 시 생략 가능 |
| **클로저** | 이름 없는 함수. `{ (매개변수) -> 반환타입 in 본문 }` |
| **후행 클로저** | 마지막 매개변수가 클로저일 때 `()` 밖에 작성 |
| **$0, $1** | 클로저의 단축 인자 이름 |
| **map** | 모든 요소를 변환하여 새 배열 반환 |
| **filter** | 조건에 맞는 요소만 걸러서 새 배열 반환 |
| **reduce** | 모든 요소를 하나의 값으로 합침 |

## 다음 섹션 미리보기

함수와 클로저를 배웠으니, 이제 Swift에서 가장 독특하고 중요한 개념인 **옵셔널(Optional)** 을 만나볼 차례입니다. "값이 있을 수도 있고, 없을 수도 있는" 상황을 Swift가 어떻게 안전하게 처리하는지 [옵셔널](./06-optionals.md)에서 알아봅시다.

## 참고 자료

- [Functions — The Swift Programming Language](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/functions/) - Swift 공식 문서의 함수 설명
- [Closures — The Swift Programming Language](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/closures/) - Swift 공식 문서의 클로저 설명
- [Functions and Closures — Hacking with Swift](https://www.hackingwithswift.com/100/swiftui/6) - 100 Days of SwiftUI Day 6-7
- [Understanding Swift Closures — Bugfender](https://bugfender.com/blog/swift-closures/) - 클로저 심화 가이드
