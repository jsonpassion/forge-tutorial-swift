# 컬렉션 타입

> Array, Dictionary, Set의 활용

## 개요

하나의 변수에 하나의 값만 담을 수 있다면, 학생 100명의 이름을 저장하려면 변수가 100개 필요하겠죠? **컬렉션(Collection)** 은 여러 값을 하나의 묶음으로 관리하는 방법입니다. Swift는 세 가지 컬렉션 타입을 제공하며, 각각 용도가 다릅니다.

**선수 지식**: [변수와 상수](./02-variables-constants.md)에서 var/let과 기본 타입을 이해한 상태
**학습 목표**:
- Array, Dictionary, Set의 차이를 이해한다
- 각 컬렉션에 값을 추가, 삭제, 조회하는 방법을 익힌다
- 상황에 맞는 컬렉션 타입을 선택할 수 있다

## 왜 알아야 할까?

앱에서 데이터는 거의 항상 **여러 개**입니다. 채팅 메시지 목록, 장바구니의 상품들, 설정 옵션들... 이런 데이터를 효율적으로 저장하고 찾고 관리하려면 올바른 컬렉션 타입을 써야 합니다. SwiftUI에서 `List`나 `ForEach`로 화면을 구성할 때도 거의 항상 컬렉션을 사용하게 되니, 지금 익혀두면 나중에 큰 도움이 됩니다.

## 핵심 개념

### 개념 1: Array — 순서가 있는 목록

> 💡 **비유**: Array는 **번호표가 붙은 사물함**입니다. 0번 칸, 1번 칸, 2번 칸... 순서대로 물건을 넣고, 번호로 꺼낼 수 있죠. 같은 물건을 여러 칸에 넣어도 됩니다.

Array(배열)는 **같은 타입의 값을 순서대로** 저장합니다. 가장 자주 쓰는 컬렉션이에요.

```swift
// 배열 생성
var fruits = ["사과", "바나나", "딸기"]
let numbers = [1, 2, 3, 4, 5]

// 값 접근 — 인덱스는 0부터 시작합니다!
print(fruits[0])    // "사과"
print(fruits[1])    // "바나나"

// 값 추가
fruits.append("포도")              // 맨 뒤에 추가
fruits.insert("망고", at: 1)       // 1번 위치에 삽입

// 값 삭제
fruits.remove(at: 0)               // 0번 요소 삭제
let last = fruits.removeLast()     // 마지막 요소 삭제하고 반환

// 유용한 프로퍼티
print(fruits.count)                // 배열 개수
print(fruits.isEmpty)              // 비어있는지 확인
print(fruits.contains("바나나"))    // 특정 값 포함 여부
```

> ⚠️ **흔한 오해**: "배열의 첫 번째 요소는 1번이다" — 프로그래밍에서 인덱스는 **0부터** 시작합니다. `fruits[1]`은 두 번째 요소예요. 존재하지 않는 인덱스에 접근하면 앱이 크래시하니 주의하세요!

### 개념 2: Dictionary — 키로 찾는 저장소

> 💡 **비유**: Dictionary는 **실제 사전**과 같습니다. 단어(키)로 검색하면 뜻(값)이 나오죠. "apple"을 찾으면 "사과"가 나오는 것처럼, 키를 넣으면 대응하는 값을 바로 꺼낼 수 있습니다.

Dictionary(딕셔너리)는 **키(Key)와 값(Value)의 쌍**으로 데이터를 저장합니다. 순서가 없는 대신, 키로 빠르게 값을 찾을 수 있어요.

```swift
// 딕셔너리 생성 — [키 타입: 값 타입]
var scores: [String: Int] = [
    "민수": 95,
    "지영": 88,
    "현우": 72
]

// 값 조회 — 키로 접근 (결과는 Optional!)
let minsuScore = scores["민수"]     // Optional(95)
let unknownScore = scores["철수"]   // nil (없는 키)

// 기본값 지정 — 키가 없을 때 사용할 값
let safeScore = scores["철수", default: 0]  // 0

// 값 추가/수정
scores["수진"] = 91                 // 새 항목 추가
scores["현우"] = 85                 // 기존 값 수정

// 값 삭제
scores["민수"] = nil                // 키-값 쌍 삭제

// 유용한 프로퍼티
print(scores.count)                 // 항목 수
print(scores.keys)                  // 모든 키
print(scores.values)                // 모든 값
```

> 💡 **알고 계셨나요?**: Dictionary에서 값을 조회하면 `Optional`이 반환됩니다. 해당 키가 없을 수도 있기 때문이죠. Optional에 대해서는 [옵셔널](./06-optionals.md) 섹션에서 자세히 다룹니다. 지금은 "키가 없으면 `nil`이 온다" 정도만 알아두세요.

### 개념 3: Set — 중복 없는 집합

> 💡 **비유**: Set은 **출석부**와 같습니다. 같은 이름이 두 번 적혀도 출석한 사람은 한 명이죠. 중복을 자동으로 제거하고, 순서는 중요하지 않습니다.

Set(집합)은 **같은 타입의 고유한 값**만 저장합니다. 순서가 없고, 중복이 자동으로 제거됩니다.

```swift
// Set 생성
var genres: Set<String> = ["액션", "코미디", "SF", "액션"]
print(genres)           // {"코미디", "SF", "액션"} — 중복 "액션" 제거!
print(genres.count)     // 3

// 값 추가/삭제
genres.insert("로맨스")
genres.remove("SF")

// 포함 여부 확인 — Set이 가장 빠릅니다!
print(genres.contains("액션"))   // true

// 집합 연산 — 수학 시간의 그 집합 연산!
let a: Set = [1, 2, 3, 4, 5]
let b: Set = [3, 4, 5, 6, 7]

print(a.union(b))           // 합집합: {1, 2, 3, 4, 5, 6, 7}
print(a.intersection(b))    // 교집합: {3, 4, 5}
print(a.subtracting(b))     // 차집합: {1, 2}
```

### 어떤 컬렉션을 쓸까?

| 상황 | 추천 컬렉션 | 이유 |
|------|------------|------|
| 할 일 목록 | **Array** | 순서가 중요하고, 같은 내용이 있을 수 있음 |
| 사용자 정보 저장 | **Dictionary** | 키(이름, 이메일 등)로 빠르게 값을 찾아야 함 |
| 좋아요 누른 유저 ID | **Set** | 중복 방지 + 포함 여부 빠른 확인 |
| 채팅 메시지 | **Array** | 시간 순서가 중요 |
| 앱 설정 | **Dictionary** | 설정 이름(키)으로 값을 관리 |

## 실습: 직접 해보기

Playground에서 간단한 장바구니를 만들어 봅시다.

```swift
import Foundation

// 장바구니 (Array) — 같은 상품을 여러 개 담을 수 있습니다
var cart = ["아이폰 케이스", "충전 케이블", "보호 필름"]

// 상품 추가
cart.append("에어팟 케이스")
print("🛒 장바구니: \(cart)")
print("총 \(cart.count)개 상품")

// 가격표 (Dictionary) — 상품명으로 가격을 찾습니다
let priceList: [String: Int] = [
    "아이폰 케이스": 25000,
    "충전 케이블": 15000,
    "보호 필름": 12000,
    "에어팟 케이스": 18000
]

// 총 금액 계산
var total = 0
for item in cart {
    let price = priceList[item, default: 0]
    print("  \(item): \(price)원")
    total += price
}
print("💰 총 금액: \(total)원")

// 찜한 카테고리 (Set) — 중복 없이 관리
var wishCategories: Set<String> = ["전자기기", "액세서리"]
wishCategories.insert("전자기기")    // 이미 있으므로 추가 안 됨
wishCategories.insert("음식")
print("❤️ 관심 카테고리: \(wishCategories)")
```

## 더 깊이 알아보기

### 컬렉션과 값 타입

Swift의 Array, Dictionary, Set은 모두 **값 타입(Value Type)** 입니다. 이게 무슨 뜻이냐면, 컬렉션을 다른 변수에 대입하면 **복사본**이 만들어진다는 뜻이에요.

```swift
var original = [1, 2, 3]
var copy = original       // 복사본 생성
copy.append(4)

print(original)  // [1, 2, 3] — 원본은 변하지 않음!
print(copy)      // [1, 2, 3, 4]
```

이건 다른 언어(Java, Python 등)와 크게 다른 점입니다. 대부분의 언어에서 배열은 참조 타입이라 복사본을 수정하면 원본도 바뀌거든요. Swift의 이 설계 덕분에 "내가 넘긴 배열을 누군가 몰래 수정했네?" 같은 버그를 걱정할 필요가 없습니다.

> 💡 **알고 계셨나요?**: Swift 컬렉션은 실제로는 **COW(Copy-On-Write)** 라는 최적화를 사용합니다. 복사할 때 실제로 메모리를 복제하는 게 아니라, 수정이 일어날 때만 복사해요. 그래서 `let copy = original` 자체는 거의 비용이 없습니다. 성능 걱정 없이 마음 편히 쓰세요!

## 흔한 오해와 팁

> ⚠️ **흔한 오해**: "Dictionary는 넣은 순서대로 저장된다" — Dictionary와 Set은 **순서가 보장되지 않습니다**. 출력할 때마다 순서가 바뀔 수 있어요. 순서가 필요하면 Array를 사용하세요.

> 🔥 **실무 팁**: 빈 컬렉션을 만들 때는 타입을 명시해야 합니다. Swift가 추론할 값이 없기 때문이죠.

```swift
// 빈 컬렉션 생성
var emptyArray: [String] = []          // 빈 String 배열
var emptyDict: [String: Int] = [:]     // 빈 딕셔너리
var emptySet: Set<String> = []         // 빈 Set
```

## 핵심 정리

| 개념 | 설명 |
|------|------|
| **Array** | 순서가 있고, 중복을 허용하는 목록. `[값1, 값2]` |
| **Dictionary** | 키-값 쌍으로 저장. 키로 빠른 조회. `[키: 값]` |
| **Set** | 중복 없는 고유 값 집합. 순서 없음. `Set<타입>` |
| **인덱스** | 배열에서 위치를 나타내는 번호. 0부터 시작 |
| **.count** | 컬렉션의 요소 개수 |
| **.contains()** | 특정 값의 포함 여부 확인 |
| **값 타입** | 대입 시 복사본 생성. 원본에 영향 없음 |

## 다음 섹션 미리보기

데이터를 저장하는 법을 배웠으니, 이제 **조건에 따라 다르게 동작하고, 반복 작업을 자동화하는 방법**을 알아봅시다. [조건문과 반복문](./04-control-flow.md)에서 `if`, `switch`, `for-in` 등을 만나봅니다.

## 참고 자료

- [Collection Types — The Swift Programming Language](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/collectiontypes/) - Swift 공식 문서의 컬렉션 타입 설명
- [Arrays, Dictionaries, Sets — Hacking with Swift](https://www.hackingwithswift.com/100/swiftui/3) - 100 Days of SwiftUI Day 3
- [Value and Reference Types — Swift.org](https://www.swift.org/documentation/articles/value-and-reference-types.html) - 값 타입과 참조 타입의 차이
