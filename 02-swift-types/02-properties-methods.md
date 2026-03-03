# 프로퍼티와 메서드

> 저장/연산 프로퍼티, 인스턴스/타입 메서드

## 개요

앞서 구조체와 클래스를 배우면서 프로퍼티(데이터)와 메서드(기능)를 간단히 사용해 봤죠. 이번에는 Swift가 제공하는 다양한 프로퍼티와 메서드의 종류를 깊이 있게 다뤄봅니다. 저장 프로퍼티, 연산 프로퍼티, 프로퍼티 감시자, 타입 프로퍼티까지 — 이것들을 알면 타입 설계가 훨씬 우아해집니다.

**선수 지식**: [구조체와 클래스](./01-struct-class.md)에서 struct/class 기본 문법
**학습 목표**:
- 저장 프로퍼티와 연산 프로퍼티의 차이를 이해한다
- 프로퍼티 감시자(willSet/didSet)를 활용한다
- 인스턴스 메서드와 타입 메서드를 구분한다
- lazy 프로퍼티의 용도를 안다

## 왜 알아야 할까?

SwiftUI에서 `var body: some View`는 **연산 프로퍼티**이고, `@State`가 변경될 때 화면이 업데이트되는 원리 뒤에는 **프로퍼티 감시자**와 비슷한 메커니즘이 있습니다. 프로퍼티의 종류를 정확히 이해하면, SwiftUI의 상태 관리가 "마법"이 아닌 "논리"로 느껴지기 시작해요.

## 핵심 개념

### 개념 1: 저장 프로퍼티 (Stored Property)

> 💡 **비유**: 저장 프로퍼티는 **서랍**입니다. 물건(값)을 넣어두고, 필요할 때 꺼내 쓰는 공간이죠.

가장 기본적인 프로퍼티로, 값을 직접 저장합니다.

```run:swift
struct Song {
    let title: String          // 상수 저장 프로퍼티 — 한번 정하면 변경 불가
    let artist: String
    var playCount: Int = 0     // 변수 저장 프로퍼티 — 기본값 지정 가능
    var isFavorite: Bool = false
}

var song = Song(title: "Dynamite", artist: "BTS")
song.playCount += 1
song.isFavorite = true
print("\(song.title) — 재생 \(song.playCount)회")  // "Dynamite — 재생 1회"
```

```output
Dynamite — 재생 1회
```

### 개념 2: 연산 프로퍼티 (Computed Property)

> 💡 **비유**: 연산 프로퍼티는 **체온계**입니다. 체온을 저장하고 있는 게 아니라, 측정할 때마다 **계산해서** 알려주는 거죠.

값을 저장하지 않고, **접근할 때마다 계산**하는 프로퍼티입니다.

```run:swift
struct Rectangle {
    var width: Double      // 저장 프로퍼티
    var height: Double     // 저장 프로퍼티

    // 연산 프로퍼티 — 접근할 때마다 계산
    var area: Double {
        width * height
    }

    var perimeter: Double {
        2 * (width + height)
    }

    // 설명을 반환하는 연산 프로퍼티
    var description: String {
        "\(width) × \(height) (넓이: \(area))"
    }
}

var rect = Rectangle(width: 10, height: 5)
print(rect.area)          // 50.0
print(rect.description)   // "10.0 × 5.0 (넓이: 50.0)"

rect.width = 20
print(rect.area)          // 100.0 — 자동으로 새 값 계산!
```

```output
50.0
10.0 × 5.0 (넓이: 50.0)
100.0
```

연산 프로퍼티는 `get`과 `set`을 모두 가질 수 있습니다.

```run:swift
struct Temperature {
    var celsius: Double    // 섭씨 (저장)

    var fahrenheit: Double {
        get {
            celsius * 9 / 5 + 32     // 섭씨 → 화씨 변환
        }
        set {
            celsius = (newValue - 32) * 5 / 9  // 화씨 → 섭씨 역변환
        }
    }
}

var temp = Temperature(celsius: 100)
print(temp.fahrenheit)    // 212.0

temp.fahrenheit = 32      // set이 호출됨
print(temp.celsius)       // 0.0
```

```output
212.0
0.0
```

> 🔥 **실무 팁**: "이 값을 저장해야 할까, 계산해야 할까?" 고민된다면 — **다른 프로퍼티로부터 유도할 수 있다면 연산 프로퍼티**를 쓰세요. 넓이 = 가로 × 세로처럼요. 데이터 불일치를 방지할 수 있습니다.

### 개념 3: 프로퍼티 감시자 (Property Observer)

> 💡 **비유**: 프로퍼티 감시자는 **CCTV**입니다. 값이 바뀔 때마다 "바뀌기 직전!"(`willSet`)과 "방금 바뀌었다!"(`didSet`)를 알려줍니다.

```run:swift
struct GamePlayer {
    var name: String

    var score: Int = 0 {
        willSet {
            // 값이 바뀌기 직전에 호출
            print("⏳ \(name)의 점수가 \(score)에서 \(newValue)로 변경 예정")
        }
        didSet {
            // 값이 바뀐 직후에 호출
            print("✅ \(name)의 점수가 \(oldValue)에서 \(score)로 변경됨")

            // 점수가 100을 넘으면 축하!
            if score >= 100 && oldValue < 100 {
                print("🎉 \(name)님이 100점을 돌파했습니다!")
            }
        }
    }
}

var player = GamePlayer(name: "민수")
player.score = 50
player.score = 120
```

```output
⏳ 민수의 점수가 0에서 50으로 변경 예정
✅ 민수의 점수가 0에서 50로 변경됨
⏳ 민수의 점수가 50에서 120으로 변경 예정
✅ 민수의 점수가 50에서 120로 변경됨
🎉 민수님이 100점을 돌파했습니다!
```

> 💡 **알고 계셨나요?**: SwiftUI의 `@State`와 `@Observable`은 프로퍼티 감시자와 비슷한 원리를 사용합니다. 값이 변경되면 시스템에 알려서 화면을 다시 그리게 하는 거죠. 이 메커니즘을 이해하면 [상태 관리](../05-state-management/01-state-binding.md) 챕터가 훨씬 쉬워집니다.

### 개념 4: lazy 프로퍼티 — 필요할 때만 생성

> 💡 **비유**: lazy 프로퍼티는 **주문 제작 가구**입니다. 미리 만들어두지 않고, 실제로 필요해서 주문이 들어올 때 비로소 제작을 시작합니다.

`lazy` 키워드를 붙이면, 해당 프로퍼티에 **처음 접근할 때** 초기화됩니다. 비용이 큰 작업을 지연시킬 때 유용해요.

```run:swift
struct DataProcessor {
    var fileName: String

    // lazy: 처음 접근할 때만 초기화됨
    lazy var loadedData: [String] = {
        print("💾 \(fileName) 데이터를 로딩 중...")
        // 실제로는 파일을 읽는 등의 무거운 작업
        return ["데이터1", "데이터2", "데이터3"]
    }()
}

var processor = DataProcessor(fileName: "report.csv")
print("프로세서 생성 완료")       // 아직 loadedData는 초기화되지 않음

print(processor.loadedData)      // 이 시점에서야 로딩 시작!
```

```output
프로세서 생성 완료
💾 report.csv 데이터를 로딩 중...
["데이터1", "데이터2", "데이터3"]
```

> ⚠️ **흔한 오해**: "`lazy`는 `let`에도 쓸 수 있다" — `lazy`는 반드시 `var`에만 사용할 수 있습니다. 초기에는 값이 없다가 나중에 채워지니까 변경 가능해야 하기 때문이에요.

### 개념 5: 타입 프로퍼티와 타입 메서드

지금까지 배운 프로퍼티와 메서드는 **인스턴스**에 속한 것이었어요. `static` 키워드를 붙이면 **타입 자체**에 속하는 프로퍼티와 메서드를 만들 수 있습니다.

```run:swift
struct AppConfig {
    // 타입 프로퍼티 — 모든 인스턴스가 공유하는 값
    static let appName = "Swift 마스터"
    static let version = "1.0.0"
    static var launchCount = 0

    // 타입 메서드 — 인스턴스 없이 타입 이름으로 직접 호출
    static func incrementLaunch() {
        launchCount += 1
        print("\(appName) v\(version) — \(launchCount)번째 실행")
    }
}

// 인스턴스를 만들 필요 없이 타입 이름으로 접근
print(AppConfig.appName)      // "Swift 마스터"
AppConfig.incrementLaunch()   // "Swift 마스터 v1.0.0 — 1번째 실행"
AppConfig.incrementLaunch()   // "Swift 마스터 v1.0.0 — 2번째 실행"
```

```output
Swift 마스터
Swift 마스터 v1.0.0 — 1번째 실행
Swift 마스터 v1.0.0 — 2번째 실행
```

```swift
// 실용적인 예: 팩토리 메서드 패턴
struct Color {
    var red: Double
    var green: Double
    var blue: Double

    // 타입 메서드로 미리 정의된 색상 제공
    static func red() -> Color {
        Color(red: 1, green: 0, blue: 0)
    }

    static func blue() -> Color {
        Color(red: 0, green: 0, blue: 1)
    }

    // 타입 프로퍼티로도 가능
    static let white = Color(red: 1, green: 1, blue: 1)
    static let black = Color(red: 0, green: 0, blue: 0)
}

let myColor = Color.red()
let bg = Color.white
```

## 실습: 직접 해보기

다양한 프로퍼티를 활용해서 간단한 쇼핑몰 상품 모델을 만들어 봅시다.

```run:swift
import Foundation

struct Product {
    let id: Int                    // 상수 저장 프로퍼티
    var name: String               // 변수 저장 프로퍼티
    var originalPrice: Int         // 원가
    var discountRate: Double = 0   // 할인율 (0~1)

    var salesCount: Int = 0 {
        didSet {
            if salesCount >= 100 && oldValue < 100 {
                print("🔥 \(name)이(가) 베스트셀러에 등극했습니다!")
            }
        }
    }

    // 연산 프로퍼티: 할인가 계산
    var finalPrice: Int {
        Int(Double(originalPrice) * (1.0 - discountRate))
    }

    // 연산 프로퍼티: 가격 표시 문자열
    var priceLabel: String {
        if discountRate > 0 {
            return "\(originalPrice)원 → \(finalPrice)원 (\(Int(discountRate * 100))% 할인)"
        }
        return "\(originalPrice)원"
    }

    // 타입 프로퍼티: 카테고리 상수
    static let categories = ["전자기기", "의류", "식품", "도서"]
    static var totalProducts = 0

    // 타입 메서드
    static func createWithAutoId(name: String, price: Int) -> Product {
        totalProducts += 1
        return Product(id: totalProducts, name: name, originalPrice: price)
    }
}

// 타입 메서드로 상품 생성
var macbook = Product.createWithAutoId(name: "맥북 프로", price: 2990000)
macbook.discountRate = 0.1

var airpods = Product.createWithAutoId(name: "에어팟", price: 249000)

print(macbook.priceLabel)    // "2990000원 → 2691000원 (10% 할인)"
print(airpods.priceLabel)    // "249000원"

// 프로퍼티 감시자 테스트
for _ in 1...100 {
    macbook.salesCount += 1
}
// 🔥 맥북 프로이(가) 베스트셀러에 등극했습니다!

print("📊 총 등록 상품: \(Product.totalProducts)개")  // 2개
print("📂 카테고리: \(Product.categories)")
```

```output
2990000원 → 2691000원 (10% 할인)
249000원
🔥 맥북 프로이(가) 베스트셀러에 등극했습니다!
📊 총 등록 상품: 2개
📂 카테고리: ["전자기기", "의류", "식품", "도서"]
```

## 더 깊이 알아보기

### 연산 프로퍼티 vs 메서드, 어떤 걸 써야 할까?

`var area: Double { width * height }` 대신 `func area() -> Double { width * height }`로 써도 동작은 같죠. 그렇다면 어떤 걸 써야 할까요?

Swift 커뮤니티에서 통용되는 기준은 이렇습니다:
- **계산 비용이 낮고** (O(1) 수준), **부수효과가 없다**면 → 연산 프로퍼티
- **계산 비용이 높거나**, 네트워크 호출 같은 **부수효과**가 있다면 → 메서드

Apple의 Swift API Design Guidelines에서도 "프로퍼티는 복잡도가 O(1)이어야 한다"고 권장합니다. 프로퍼티에 접근하는 코드를 읽는 사람은 "그냥 값을 읽는구나"라고 기대하지, "무거운 작업이 돌아가겠구나"라고는 생각하지 않거든요.

## 흔한 오해와 팁

> ⚠️ **흔한 오해**: "연산 프로퍼티는 값을 캐싱한다" — 연산 프로퍼티는 **접근할 때마다 매번 다시 계산**합니다. 결과를 저장하지 않아요. 캐싱이 필요하면 저장 프로퍼티 + `didSet` 조합이나 `lazy`를 고려하세요.

> 🔥 **실무 팁**: `didSet`에서 값을 다시 변경할 수 있습니다. 예를 들어 점수가 음수가 되지 않게 보정할 때 유용해요.

```swift
var score: Int = 0 {
    didSet {
        if score < 0 { score = 0 }   // 음수면 0으로 보정
    }
}
```

> 💡 **알고 계셨나요?**: `static`과 `class` 키워드의 차이는 — `static`은 재정의(override)가 불가능하지만, `class`는 하위 클래스에서 재정의할 수 있습니다. struct에서는 `static`만 사용할 수 있어요.

## 핵심 정리

| 개념 | 설명 |
|------|------|
| **저장 프로퍼티** | 값을 직접 저장. `var name: String` |
| **연산 프로퍼티** | 접근할 때마다 계산. `var area: Double { ... }` |
| **willSet / didSet** | 값 변경 전후에 호출되는 감시자 |
| **lazy** | 첫 접근 시 초기화. `lazy var data = ...` |
| **static** | 타입 자체에 속하는 프로퍼티/메서드 |
| **get / set** | 연산 프로퍼티의 읽기/쓰기 구현 |
| **newValue / oldValue** | willSet에서 새 값, didSet에서 이전 값 |

## 다음 섹션 미리보기

프로퍼티와 메서드로 타입의 내부를 채우는 법을 배웠습니다. 다음은 타입에 **능력을 부여하는 약속**, 바로 [프로토콜과 익스텐션](./03-protocols-extensions.md)입니다. Swift가 "프로토콜 지향 프로그래밍" 언어라 불리는 이유를 알게 될 거예요.

## 참고 자료

- [Properties — The Swift Programming Language](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/properties/) - Swift 공식 문서의 프로퍼티 종류별 설명
- [Methods — The Swift Programming Language](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/methods/) - 인스턴스/타입 메서드 공식 문서
- [Swift API Design Guidelines](https://www.swift.org/documentation/api-design-guidelines/) - 프로퍼티 vs 메서드 선택 기준
- [Properties and Methods — Hacking with Swift](https://www.hackingwithswift.com/100/swiftui/10) - 100 Days of SwiftUI Day 10
