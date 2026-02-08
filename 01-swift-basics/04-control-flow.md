# 조건문과 반복문

> if, guard, switch, for-in, while 제어 흐름

## 개요

지금까지는 코드가 위에서 아래로 한 줄씩 실행되었죠. 하지만 실제 앱은 **상황에 따라 다르게 동작**해야 합니다. 로그인 성공이면 홈 화면을, 실패면 에러 메시지를 보여주는 것처럼요. 이런 **흐름 제어(Control Flow)** 가 프로그래밍의 핵심입니다.

**선수 지식**: [변수와 상수](./02-variables-constants.md), [컬렉션 타입](./03-collections.md)
**학습 목표**:
- `if`/`else`로 조건 분기를 구현한다
- `guard`로 조기 탈출 패턴을 사용한다
- `switch`로 여러 경우를 깔끔하게 처리한다
- `for-in`과 `while`로 반복 작업을 자동화한다

## 왜 알아야 할까?

앱의 모든 로직은 결국 "이럴 때는 이렇게, 저럴 때는 저렇게"의 연속입니다. 사용자가 버튼을 탭했을 때, 네트워크 응답이 왔을 때, 데이터가 비어있을 때... 모든 상황에 대응하려면 조건문이 필수죠. 그리고 배열의 모든 항목을 화면에 표시하거나, 특정 조건을 만족할 때까지 기다리려면 반복문이 필요합니다.

## 핵심 개념

### 개념 1: if/else — 기본 조건 분기

> 💡 **비유**: `if`문은 **갈림길의 이정표**입니다. "비가 오면 우산을 챙기고, 아니면 그냥 나간다" — 조건에 따라 다른 길을 가는 거죠.

```swift
let temperature = 32

// 기본 if/else
if temperature > 30 {
    print("🥵 너무 덥습니다! 에어컨을 켜세요.")
} else if temperature > 20 {
    print("😊 쾌적한 날씨입니다.")
} else if temperature > 10 {
    print("🧥 겉옷을 챙기세요.")
} else {
    print("🥶 많이 춥습니다!")
}
// 출력: "🥵 너무 덥습니다! 에어컨을 켜세요."
```

Swift의 `if`문에서 주의할 점 하나! 조건에 **괄호`()`가 필수가 아닙니다**. C나 Java에서 오신 분들은 `if (temperature > 30)` 처럼 쓰는 게 습관이겠지만, Swift에서는 괄호 없이 `if temperature > 30` 이 더 자연스러운 스타일이에요.

```swift
// 여러 조건을 AND(&&), OR(||)로 조합
let age = 20
let hasTicket = true

if age >= 19 && hasTicket {
    print("입장 가능합니다! 🎬")
}

// 쉼표(,)로도 AND 조건을 표현할 수 있습니다 (Swift 고유 문법)
if age >= 19, hasTicket {
    print("입장 가능합니다! 🎬")
}
```

### 개념 2: guard — "아니면 나가!" 패턴

> 💡 **비유**: `guard`는 **건물 경비원**입니다. "신분증 있으세요? 없으면 못 들어갑니다!" — 조건을 만족하지 않으면 바로 내보내고, 만족하면 통과시킵니다.

`guard`는 **조건이 거짓일 때 조기 탈출(early exit)** 하는 구문입니다. `if`와 반대로 생각하면 편해요.

```swift
func processOrder(itemCount: Int) {
    // guard: 조건이 거짓이면 else 블록 실행 후 함수 종료
    guard itemCount > 0 else {
        print("❌ 상품을 1개 이상 선택해주세요.")
        return
    }

    // 여기에 도달했다면 itemCount > 0이 보장됩니다
    print("✅ \(itemCount)개 상품을 주문합니다.")
}

processOrder(itemCount: 0)   // "❌ 상품을 1개 이상 선택해주세요."
processOrder(itemCount: 3)   // "✅ 3개 상품을 주문합니다."
```

> 🔥 **실무 팁**: `guard`는 함수 시작 부분에서 **유효성 검사**를 할 때 특히 유용합니다. "이 조건이 안 되면 더 진행할 필요 없어"라는 상황에 딱이죠. `if`로 중첩이 깊어지는 것보다 `guard`로 빨리 빠져나가는 게 코드가 훨씬 읽기 좋습니다.

### 개념 3: switch — 여러 경우를 깔끔하게

> 💡 **비유**: `switch`는 **자동 분류기**입니다. 편지를 넣으면 우편번호에 따라 자동으로 올바른 칸에 배분되죠. 여러 경우를 한눈에 정리할 수 있습니다.

```swift
let dayOfWeek = "수요일"

switch dayOfWeek {
case "월요일":
    print("😫 한 주의 시작...")
case "수요일":
    print("🐫 벌써 수요일! 반이나 왔어요.")
case "금요일":
    print("🎉 불금이다!")
case "토요일", "일요일":     // 여러 값을 한 case에
    print("😴 주말이다!")
default:                     // 나머지 모든 경우
    print("평범한 하루")
}
// 출력: "🐫 벌써 수요일! 반이나 왔어요."
```

Swift의 `switch`는 다른 언어와 크게 다른 점이 있어요. **`break`를 쓸 필요가 없습니다!** C나 Java에서는 `break`를 빼먹으면 다음 case로 떨어지는(fall-through) 버그가 생기는데, Swift는 매칭된 case만 실행하고 자동으로 빠져나옵니다.

```swift
// 범위(Range)도 사용 가능합니다
let score = 85

switch score {
case 90...100:
    print("A학점 🏆")
case 80..<90:
    print("B학점 👍")
case 70..<80:
    print("C학점")
case 60..<70:
    print("D학점")
default:
    print("F학점 😢")
}
// 출력: "B학점 👍"
```

> ⚠️ **흔한 오해**: "switch는 숫자나 문자열만 쓸 수 있다" — Swift의 `switch`는 **튜플, 범위, 열거형, 패턴 매칭** 등 거의 모든 타입과 함께 사용할 수 있어요. 이후 [열거형과 패턴 매칭](../02-swift-types/04-enums-pattern.md)에서 더 강력한 활용법을 배웁니다.

### 개념 4: for-in — 컬렉션 순회

> 💡 **비유**: `for-in`은 **컨베이어 벨트**입니다. 벨트 위의 물건이 하나씩 지나가면서 각각에 대해 작업을 수행하는 거죠.

```swift
// 배열 순회
let fruits = ["사과", "바나나", "딸기", "포도"]

for fruit in fruits {
    print("🍎 \(fruit)을(를) 씻고 있어요...")
}

// 숫자 범위 순회
for i in 1...5 {
    print("\(i)번째 반복")
}

// Dictionary 순회
let scores = ["민수": 95, "지영": 88, "현우": 72]

for (name, score) in scores {
    print("\(name): \(score)점")
}

// 인덱스가 필요할 때 — enumerated()
for (index, fruit) in fruits.enumerated() {
    print("\(index + 1)번: \(fruit)")
}
```

```swift
// where 절로 필터링하며 반복
let numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]

for number in numbers where number % 2 == 0 {
    print("\(number)은 짝수")    // 2, 4, 6, 8, 10
}
```

### 개념 5: while — 조건이 참인 동안 반복

```swift
// 기본 while — 조건을 먼저 확인하고 실행
var countdown = 5
while countdown > 0 {
    print("\(countdown)...")
    countdown -= 1
}
print("🚀 발사!")

// repeat-while — 먼저 실행하고 조건 확인 (다른 언어의 do-while)
var input = 0
repeat {
    input += 1
    print("시도 \(input)번째...")
} while input < 3
```

> 💡 **알고 계셨나요?**: Swift에서는 C 스타일의 `for (int i = 0; i < 10; i++)` 반복문이 **없습니다**. Swift 3에서 제거되었거든요. Swift Evolution 프로포절 SE-0007로 커뮤니티 투표를 거쳐 삭제된 건데, `for-in`이 더 안전하고 읽기 쉽다는 이유였습니다. 재미있는 건, `++`와 `--` 연산자도 같은 시기에 함께 제거되었다는 사실!

## 실습: 직접 해보기

간단한 성적 처리 프로그램을 만들어 봅시다.

```swift
import Foundation

// 학생 성적 데이터
let studentScores = [
    "민수": 95, "지영": 88, "현우": 72,
    "수진": 91, "태호": 65, "예린": 83
]

// 전체 학생의 성적 처리
var passCount = 0
var failCount = 0
var totalScore = 0

print("📋 성적표")
print("─────────────────")

for (name, score) in studentScores {
    totalScore += score

    // guard로 과락 체크
    guard score >= 70 else {
        print("❌ \(name): \(score)점 — 과락")
        failCount += 1
        continue    // 다음 학생으로
    }

    // switch로 등급 판정
    let grade: String
    switch score {
    case 90...100:
        grade = "A"
    case 80..<90:
        grade = "B"
    case 70..<80:
        grade = "C"
    default:
        grade = "F"
    }

    print("✅ \(name): \(score)점 — \(grade)등급")
    passCount += 1
}

print("─────────────────")
let average = totalScore / studentScores.count
print("📊 평균: \(average)점")
print("합격: \(passCount)명 / 불합격: \(failCount)명")
```

## 더 깊이 알아보기

### Swift의 switch가 특별한 이유

Swift의 `switch`는 단순한 값 비교를 넘어서 **패턴 매칭(Pattern Matching)** 을 지원합니다. 이 기능은 Haskell, Rust 같은 함수형 언어에서 영감을 받은 것인데요, Swift가 이걸 매우 실용적으로 구현해서 많은 개발자들이 좋아하는 기능 중 하나입니다.

또한 Swift의 `switch`는 **반드시 모든 경우를 다뤄야 합니다(exhaustive)**. 빠뜨린 case가 있으면 컴파일 에러가 발생해요. "이 경우를 깜빡했네!" 하는 버그를 원천 차단하는 셈이죠. 그래서 `default`가 없으면 모든 가능한 값을 case로 나열해야 합니다.

## 흔한 오해와 팁

> ⚠️ **흔한 오해**: "for-in에서 인덱스를 못 쓴다" — `enumerated()` 메서드를 사용하면 인덱스와 값을 동시에 받을 수 있습니다. `for (index, value) in array.enumerated()` 형태로 사용하세요.

> 🔥 **실무 팁**: `if`와 `guard` 중 뭘 쓸지 고민될 때, 이렇게 판단하세요. 조건이 **실패했을 때 처리할 게 많다**면 `if`, 조건을 확인하고 **빠르게 빠져나가고 싶다**면 `guard`. 특히 함수 시작 부분의 유효성 검사에는 `guard`가 압도적으로 깔끔합니다.

> ⚠️ **흔한 오해**: "while 반복문이 for-in보다 좋다" — 컬렉션이나 범위를 순회할 때는 항상 `for-in`을 쓰세요. `while`은 **반복 횟수를 미리 알 수 없을 때**(예: 사용자 입력을 기다리는 경우)에만 사용하는 것이 Swift의 관례입니다.

## 핵심 정리

| 개념 | 설명 |
|------|------|
| **if/else** | 조건에 따라 다른 코드를 실행하는 기본 분기문 |
| **guard** | 조건이 거짓이면 조기 탈출. 유효성 검사에 적합 |
| **switch** | 여러 경우를 깔끔하게 매칭. 모든 경우를 다뤄야 함 |
| **for-in** | 컬렉션이나 범위를 순회하는 반복문 |
| **while** | 조건이 참인 동안 반복. 반복 횟수를 모를 때 사용 |
| **where** | for-in이나 switch에서 추가 조건을 거는 절 |
| **범위 연산자** | `1...5` (닫힌 범위), `1..<5` (반열린 범위) |

## 다음 섹션 미리보기

지금까지 데이터를 저장하고, 조건을 판단하고, 반복하는 법을 배웠습니다. 이제 **코드를 재사용 가능한 단위로 묶는 방법**을 알아볼 차례예요. [함수와 클로저](./05-functions-closures.md)에서 `func`과 클로저의 세계를 만나봅시다.

## 참고 자료

- [Control Flow — The Swift Programming Language](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/controlflow/) - Swift 공식 문서의 제어 흐름
- [Conditions and Loops — Hacking with Swift](https://www.hackingwithswift.com/100/swiftui/5) - 100 Days of SwiftUI Day 5
- [SE-0007: Remove C-style for-loops](https://github.com/swiftlang/swift-evolution/blob/main/proposals/0007-remove-c-style-for-loops.md) - C 스타일 for 루프 제거 프로포절
- [Pattern Matching in Swift — Swift by Sundell](https://www.swiftbysundell.com/articles/pattern-matching-in-swift/) - 패턴 매칭 심화
