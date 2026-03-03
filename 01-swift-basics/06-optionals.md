# 옵셔널

> Optional, 바인딩, 체이닝, nil 안전성의 철학

## 개요

프로그래밍에서 가장 흔한 크래시 원인은 뭘까요? 바로 **"값이 없는데 있다고 가정하고 사용한 것"** 입니다. Swift의 **옵셔널(Optional)** 은 이 문제를 언어 차원에서 근본적으로 해결합니다. 처음엔 조금 낯설 수 있지만, 한번 익히면 "이게 없이 어떻게 코딩했지?" 싶을 거예요.

**선수 지식**: [함수와 클로저](./05-functions-closures.md)까지의 기본 문법
**학습 목표**:
- Optional이 무엇인지, 왜 필요한지 이해한다
- `if let`, `guard let`으로 안전하게 값을 꺼내는 방법을 익힌다
- 옵셔널 체이닝과 nil 병합 연산자를 활용한다
- 강제 언래핑의 위험성을 이해한다

## 왜 알아야 할까?

Swift에서 옵셔널을 모르면 **한 발자국도 앞으로 갈 수 없습니다**. Dictionary에서 값을 꺼낼 때, 텍스트 필드의 입력값을 처리할 때, 네트워크 응답을 받을 때... 모두 옵셔널이 관련되어 있거든요. 실제로 Swift 앱 크래시의 30% 이상이 옵셔널을 잘못 처리해서 발생한다는 통계도 있습니다. 이 섹션은 Ch1에서 **가장 중요한 섹션**입니다.

## 핵심 개념

### 개념 1: Optional이란?

> 💡 **비유**: 옵셔널은 **선물 상자**입니다. 상자 안에 선물이 들어있을 수도 있고, 비어있을 수도 있죠. 열어보기(언래핑) 전까지는 안에 뭐가 있는지 알 수 없습니다. Swift는 "이 상자가 비어있을 수도 있어!"라고 미리 경고해주는 셈이에요.

```run:swift
// 일반 변수 — 반드시 값이 있어야 합니다
let name: String = "민수"
// let name2: String = nil  // ❌ 에러! 일반 타입에는 nil 불가

// 옵셔널 변수 — 값이 있을 수도, nil(없음)일 수도 있습니다
var nickname: String? = "스위프트 마스터"
print(nickname as Any)

nickname = nil    // ✅ nil 할당 가능
print(nickname as Any)
```

```output
Optional("스위프트 마스터")
nil
```

타입 뒤에 `?`를 붙이면 **옵셔널 타입**이 됩니다. `String?`은 "String이 있을 수도 있고, nil일 수도 있다"는 뜻이에요.

> ⚠️ **흔한 오해**: "`nil`은 0이나 빈 문자열과 같다" — 전혀 다릅니다! `0`은 숫자 0이라는 **값이 있는 것**이고, `""`은 빈 문자열이라는 **값이 있는 것**입니다. `nil`은 **값 자체가 없음**을 의미합니다. 빈 컵에 물이 없는 것(빈 문자열)과 컵 자체가 없는 것(nil)의 차이라고 생각하세요.

### 개념 2: 옵셔널 바인딩 — 안전하게 꺼내기

옵셔널에서 값을 꺼내려면 **언래핑(Unwrapping)** 이 필요합니다. 가장 안전한 방법은 `if let`과 `guard let`입니다.

```swift
let userInput: String? = "42"

// if let — 값이 있으면 블록 안에서 사용
if let value = userInput {
    print("입력값: \(value)")
    // value는 여기서 String 타입 (옵셔널 아님!)
} else {
    print("입력값이 없습니다")
}

// Swift 5.7+: 같은 이름으로 간단히 쓸 수도 있습니다
if let userInput {
    print("입력값: \(userInput)")  // 같은 이름으로 언래핑
}
```

```output
입력값: 42
입력값: 42
```

```swift
// guard let — 값이 없으면 빠져나가기
func processAge(input: String?) {
    guard let text = input else {
        print("❌ 입력값이 없습니다")
        return
    }

    guard let age = Int(text) else {
        print("❌ '\(text)'는 숫자가 아닙니다")
        return
    }

    // 여기에 도달하면 age가 확실히 존재합니다
    print("✅ 나이: \(age)세")
}

processAge(input: nil)      // "❌ 입력값이 없습니다"
processAge(input: "abc")    // "❌ 'abc'는 숫자가 아닙니다"
processAge(input: "25")     // "✅ 나이: 25세"
```

```output
❌ 입력값이 없습니다
❌ 'abc'는 숫자가 아닙니다
✅ 나이: 25세
```

> 🔥 **실무 팁**: `if let` vs `guard let` 언제 쓸까요? **값이 있을 때 무언가를 하고 싶으면** `if let`, **값이 없으면 빠르게 빠져나가고 싶으면** `guard let`을 쓰세요. 함수 시작 부분에서 입력 검증할 때는 `guard let`이 압도적으로 깔끔합니다.

### 개념 3: 강제 언래핑 — 위험한 지름길

`!`를 붙이면 옵셔널을 강제로 벗길 수 있지만, 값이 `nil`이면 **앱이 즉시 크래시**합니다.

```swift
let possibleNumber: String? = "123"
let number = Int(possibleNumber!)!   // ⚠️ 위험! nil이면 크래시

// 이렇게 쓰지 마세요! 대신 이렇게:
if let text = possibleNumber, let number = Int(text) {
    print("안전한 숫자: \(number)")
}
```

```output
안전한 숫자: 123
```

> ⚠️ **흔한 오해**: "느낌표(!)를 쓰면 편하니까 자주 써도 된다" — **절대 아닙니다!** 강제 언래핑은 "여기서 nil이 올 수 없다고 100% 확신한다"는 의미입니다. 실무에서는 거의 쓸 일이 없어요. 대부분의 상황에서 `if let`이나 `guard let`이 정답입니다.

### 개념 4: 옵셔널 체이닝

> 💡 **비유**: 옵셔널 체이닝은 **도미노**와 같습니다. 중간에 하나라도 `nil`이면 거기서 멈추고 전체 결과가 `nil`이 됩니다. 크래시 없이요!

```swift
// 중첩된 옵셔널 접근
struct Address {
    var city: String
    var zipCode: String?
}

struct Person {
    var name: String
    var address: Address?
}

let person = Person(name: "지영", address: Address(city: "서울", zipCode: "06100"))

// 옵셔널 체이닝 — ?로 연결
let zipCode = person.address?.zipCode
print(zipCode as Any)

// address가 nil이면? 크래시 없이 nil 반환
let nobody = Person(name: "익명", address: nil)
let noZip = nobody.address?.zipCode
print(noZip as Any)
```

```output
Optional("06100")
nil
```

### 개념 5: nil 병합 연산자 (??)

옵셔널 값이 `nil`일 때 사용할 **기본값**을 지정하는 깔끔한 방법입니다.

```run:swift
let savedUsername: String? = nil

// nil 병합 연산자 — "값이 없으면 이걸 써"
let displayName = savedUsername ?? "게스트"
print(displayName)  // "게스트"

// if let 없이도 간결하게 기본값 처리
let fontSize: Int? = nil
let actualSize = fontSize ?? 16    // nil이면 16 사용
print("폰트 크기: \(actualSize)")  // "폰트 크기: 16"

// 연쇄 사용도 가능
let primary: String? = nil
let secondary: String? = nil
let fallback = "기본값"
let result = primary ?? secondary ?? fallback
print(result)  // "기본값"
```

```output
게스트
폰트 크기: 16
기본값
```

## 실습: 직접 해보기

사용자 프로필을 안전하게 처리하는 코드를 만들어 봅시다.

```swift
import Foundation

// 사용자 프로필 데이터 (일부 정보가 없을 수 있습니다)
let profiles: [[String: String?]] = [
    ["name": "민수", "email": "minsu@email.com", "phone": "010-1234-5678"],
    ["name": "지영", "email": nil, "phone": "010-9876-5432"],
    ["name": "현우", "email": "hyunwoo@email.com", "phone": nil],
    ["name": nil, "email": nil, "phone": nil]
]

print("📋 사용자 프로필 조회")
print("────────────────────────")

for (index, profile) in profiles.enumerated() {
    print("\n[\(index + 1)번 사용자]")

    // guard let으로 필수 정보 확인
    guard let name = profile["name"] ?? nil else {
        print("  ❌ 이름 없음 — 건너뜀")
        continue
    }

    print("  이름: \(name)")

    // nil 병합 연산자로 선택 정보 처리
    let email = (profile["email"] ?? nil) ?? "미등록"
    let phone = (profile["phone"] ?? nil) ?? "미등록"

    print("  이메일: \(email)")
    print("  전화: \(phone)")

    // 옵셔널 체이닝으로 이메일 도메인 추출
    if let emailValue = profile["email"] ?? nil {
        let domain = emailValue.split(separator: "@").last
            .map(String.init) ?? "알 수 없음"
        print("  도메인: \(domain)")
    }
}
```

## 더 깊이 알아보기

### "10억 달러짜리 실수"

옵셔널이 왜 탄생했는지 알려면, **Tony Hoare**라는 컴퓨터 과학자의 고백을 들어봐야 합니다. 1965년, 그는 ALGOL 언어에 **null 참조(null reference)** 를 도입했는데요, 2009년 강연에서 이를 **"10억 달러짜리 실수(Billion Dollar Mistake)"** 라고 부르며 깊이 후회했습니다.

null 참조 때문에 지난 수십 년간 수없이 많은 프로그램이 크래시하고, 보안 취약점이 생기고, 시스템이 멈췄거든요. Java의 `NullPointerException`, Objective-C의 예기치 않은 nil 메시지... 모두 같은 문제입니다.

Swift의 옵셔널은 이 문제를 **타입 시스템 차원에서** 해결합니다. "이 변수는 nil이 될 수 있다"를 타입으로 명시하게 함으로써, 프로그래머가 nil 가능성을 **반드시 인지하고 처리하도록** 강제하는 거예요. 의외로 Kotlin의 `?`, Rust의 `Option<T>`, TypeScript의 `strict null checks`도 모두 같은 철학을 따릅니다. Swift가 이 트렌드를 주도한 셈이죠.

## 흔한 오해와 팁

> ⚠️ **흔한 오해**: "옵셔널은 Swift만의 특별한 기능이다" — 옵셔널의 **개념** 자체는 Haskell의 `Maybe`, Rust의 `Option` 등에도 있습니다. 하지만 Swift처럼 **메인스트림 언어에서 이를 기본으로 채택**하고, `?` 하나로 직관적으로 쓸 수 있게 만든 건 Swift의 큰 공로입니다.

> 🔥 **실무 팁**: 옵셔널을 다룰 때 이 우선순위를 기억하세요:
> 1. `if let` / `guard let` — 가장 안전하고 일반적
> 2. `??` (nil 병합) — 기본값이 있을 때 가장 깔끔
> 3. 옵셔널 체이닝 `?.` — 중첩 접근에 편리
> 4. `!` (강제 언래핑) — **최후의 수단.** 거의 쓰지 마세요

> 💡 **알고 계셨나요?**: `Optional<String>`과 `String?`은 완전히 **같은 타입**입니다. `String?`은 `Optional<String>`의 문법적 설탕(syntactic sugar)일 뿐이에요. 내부적으로 옵셔널은 `enum`으로 구현되어 있습니다: `.some(값)` 또는 `.none`(nil). 이건 [열거형과 패턴 매칭](../02-swift-types/04-enums-pattern.md)에서 더 자세히 다룹니다.

## 핵심 정리

| 개념 | 설명 |
|------|------|
| **Optional (?)** | 값이 있거나 nil일 수 있는 타입. `String?`, `Int?` |
| **nil** | 값이 없음을 나타내는 특별한 리터럴 |
| **if let** | 옵셔널에 값이 있으면 언래핑하여 사용 |
| **guard let** | 값이 없으면 조기 탈출. 유효성 검사에 적합 |
| **강제 언래핑 (!)** | 강제로 값을 꺼냄. nil이면 크래시. 사용 자제 |
| **옵셔널 체이닝 (?.)** | 연쇄적으로 옵셔널에 접근. nil이면 전체가 nil |
| **nil 병합 (??)** | nil일 때 사용할 기본값 지정 |

## 다음 섹션 미리보기

Ch1 "Swift 시작하기"를 모두 마쳤습니다! 🎉 변수, 컬렉션, 조건문, 함수, 옵셔널까지 Swift의 기초 문법을 익혔어요. 다음 챕터 [Ch2. Swift 타입 시스템](../02-swift-types/01-struct-class.md)에서는 **구조체와 클래스**, 프로토콜, 열거형 등 Swift의 타입 중심 설계를 깊이 파고들어 봅시다. 여기서부터 Swift가 정말 빛나기 시작합니다!

## 참고 자료

- [Optionals — The Swift Programming Language](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/thebasics/#Optional-Binding) - Swift 공식 문서의 옵셔널 설명
- [Optionals — Hacking with Swift](https://www.hackingwithswift.com/100/swiftui/12) - 100 Days of SwiftUI에서 옵셔널 다루기
- [Tony Hoare: Null References, The Billion Dollar Mistake](https://www.infoq.com/presentations/Null-References-The-Billion-Dollar-Mistake-Tony-Hoare/) - "10억 달러짜리 실수" 원본 강연
- [Optional Chaining — Swift.org](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/optionalchaining/) - 옵셔널 체이닝 공식 가이드
