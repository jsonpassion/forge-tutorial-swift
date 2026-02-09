# Swift 6와 Concurrency 안전

> Strict Concurrency, Sendable 준수, 마이그레이션 전략

## 개요

Swift 6는 **데이터 경쟁(data race)을 컴파일 타임에 방지**하는 혁신적인 언어 모드입니다. 기존에는 런타임에서야 발견되던 동시성 버그를, 이제는 컴파일러가 미리 잡아줍니다. 이 섹션에서는 Swift 6의 Strict Concurrency가 무엇이고, 기존 프로젝트를 어떻게 마이그레이션하는지 배웁니다.

**선수 지식**: [Swift Concurrency 심화](../13-performance/01-concurrency-deep.md)
**학습 목표**:
- Swift 6 언어 모드의 Strict Concurrency가 무엇인지 이해할 수 있다
- Sendable 프로토콜과 actor 격리의 원리를 설명할 수 있다
- 기존 Swift 5 프로젝트를 Swift 6로 점진적으로 마이그레이션할 수 있다

## 왜 알아야 할까?

"앱이 가끔 크래시가 나는데 재현이 안 돼요" — 동시성 버그의 전형적인 증상이죠. 두 스레드가 같은 데이터를 동시에 읽고 쓰면 예측 불가능한 결과가 발생합니다. 이런 **데이터 경쟁**은 디버깅이 극도로 어렵고, 출시 후에야 간헐적으로 나타나기도 합니다. Swift 6는 이 문제를 **컴파일 타임에 원천 차단**합니다. Xcode 26의 새 프로젝트는 Swift 6.2의 "Approachable Concurrency"가 기본 활성화되어, 앞으로 모든 Swift 개발자가 알아야 할 핵심 지식입니다.

## 핵심 개념

### 개념 1: Sendable — "이 값은 안전하게 공유할 수 있어요"

> 💡 **비유**: Sendable은 **택배 가능 스티커**입니다. 택배 박스에 "배송 가능" 스티커가 붙어 있으면 안전하게 다른 곳으로 보낼 수 있죠. 스티커가 없으면? 배송 중 깨질 수 있으니 직접 들고 가야 합니다.

`Sendable`은 **마커 프로토콜**로, 값이 격리 도메인(isolation domain) 사이를 안전하게 이동할 수 있다고 선언합니다. 런타임 요구사항은 없고, 컴파일 타임에만 검증됩니다.

**자동으로 Sendable인 타입들:**

| 타입 | 이유 |
|------|------|
| `Int`, `String`, `Bool` 등 기본 타입 | 값 타입이므로 복사됨 |
| 모든 저장 프로퍼티가 Sendable인 `struct` | 값 타입 + 안전한 내용물 |
| 연관 값이 모두 Sendable인 `enum` | 값 타입 + 안전한 내용물 |
| `actor` 타입 | 자체 격리로 보호됨 |

```swift
// ✅ 자동으로 Sendable (모든 프로퍼티가 Sendable)
struct UserProfile {
    let name: String      // String은 Sendable
    let age: Int          // Int도 Sendable
}

// ❌ Sendable이 아님 (var 프로퍼티가 있는 class)
class UserSettings {
    var theme: String = "dark"  // 여러 스레드에서 동시 접근 가능!
}

// ✅ @unchecked Sendable: "내가 스레드 안전을 보장할게"
final class ThreadSafeCache: @unchecked Sendable {
    private var cache: [String: Any] = [:]
    private let lock = NSLock()

    func get(_ key: String) -> Any? {
        lock.lock()
        defer { lock.unlock() }
        return cache[key]
    }
}
```

Swift 6.2(iOS 18+)에서는 `Mutex`를 사용하면 `@unchecked` 없이도 안전한 공유가 가능합니다.

```swift
import Synchronization

// ✅ Mutex로 진정한 Sendable 달성 (iOS 18+)
final class SafeCounter: Sendable {
    let value = Mutex(0)  // Mutex 자체가 Sendable

    func increment() {
        value.withLock { $0 += 1 }
    }

    func current() -> Int {
        value.withLock { $0 }
    }
}
```

### 개념 2: Actor 격리 — 데이터를 보호하는 방

> 💡 **비유**: Actor는 **1인용 사무실**입니다. 사무실 안의 서류(데이터)는 그 방의 주인만 직접 만질 수 있어요. 다른 사람이 서류가 필요하면 문을 노크하고(`await`) 기다려야 합니다. 동시에 여러 명이 서류를 뒤적이는 일이 원천적으로 불가능하죠.

Swift 6에서 모든 가변 상태는 **격리 도메인**에 속합니다. 격리 도메인 사이에서 값을 주고받으려면 그 값이 `Sendable`이어야 합니다.

```swift
// Actor: 자체 격리 도메인을 가진 참조 타입
actor ShoppingCart {
    private var items: [String] = []

    // actor 내부에서는 자유롭게 접근
    func add(_ item: String) {
        items.append(item)
    }

    func total() -> Int {
        items.count
    }
}

// 외부에서 접근하려면 반드시 await
let cart = ShoppingCart()
await cart.add("iPhone")          // await 필수!
let count = await cart.total()    // await 필수!
```

**`@MainActor`**는 메인 스레드에서 실행을 보장하는 글로벌 actor입니다. UI 코드는 여기에 격리됩니다.

```swift
@MainActor
class WeatherViewModel: Observable {
    var temperature: Double = 0.0  // 메인 스레드에서만 접근
    var isLoading = false

    func refresh() async {
        isLoading = true
        // 네트워크 요청은 백그라운드에서 실행
        let data = await WeatherAPI.fetch()
        temperature = data.temp  // 다시 메인 스레드
        isLoading = false
    }
}
```

**`nonisolated` 키워드**: actor 격리가 필요 없는 멤버에 사용합니다.

```swift
actor DataManager {
    var items: [String] = []

    // 불변 프로퍼티는 격리 불필요
    nonisolated let id = UUID()

    // 순수 계산도 격리 불필요
    nonisolated func description() -> String {
        "DataManager instance"
    }
}
```

### 개념 3: Swift 6 마이그레이션 — 한 걸음씩

> 💡 **비유**: Swift 6 마이그레이션은 **집 리모델링**입니다. 한 번에 전체를 부수는 게 아니라, 방 하나씩(모듈 하나씩) 리모델링하면서 나머지 방은 그대로 사용하는 거죠.

마이그레이션은 3단계로 진행합니다.

**1단계: Swift 5 모드에서 경고 켜기**

Swift 5 언어 모드를 유지하면서, Strict Concurrency 경고를 활성화합니다.

| 설정 방법 | 값 |
|-----------|------|
| Xcode Build Settings | "Strict Concurrency Checking" → "Complete" |
| Package.swift | `.enableUpcomingFeature("StrictConcurrency")` |
| 커맨드 라인 | `-strict-concurrency=complete` |

**2단계: 모듈별로 경고 수정**

```swift
// ⚠️ 경고: "Static property 'shared' is not concurrency-safe"
class NetworkManager {
    static let shared = NetworkManager()  // 문제!
}

// ✅ 해결 1: @MainActor 추가
@MainActor
class NetworkManager {
    static let shared = NetworkManager()
}

// ✅ 해결 2: actor로 변환
actor NetworkManager {
    static let shared = NetworkManager()
}
```

```swift
// ⚠️ 경고: 아직 업데이트 안 된 라이브러리 사용 시
@preconcurrency import SomeOldLibrary  // 경고 억제
```

**3단계: Swift 6 모드로 전환**

모든 경고를 해결한 후, Swift Language Version을 6으로 변경합니다. 경고가 에러로 바뀝니다.

### 개념 4: Swift 6.2 — Approachable Concurrency

Swift 6.0의 엄격한 규칙이 진입 장벽이 높다는 피드백을 받아, Swift 6.2(Xcode 26)에서는 **"다가가기 쉬운 동시성"**이 도입되었습니다.

**핵심 변화: 모듈 전체를 `@MainActor`로 기본 격리**

```swift
// Package.swift에서 설정
.target(name: "MyApp", swiftSettings: [
    .defaultIsolation(MainActor.self)  // 모든 코드가 기본 @MainActor
])
```

이렇게 설정하면 모든 코드가 기본적으로 메인 스레드에서 실행됩니다. 백그라운드 실행이 필요한 곳에만 명시적으로 표시하면 됩니다.

```swift
// 기본: 메인 스레드에서 실행 (별도 표시 불필요)
struct ContentView: View {
    var body: some View {
        Text("Hello")
    }
}

// 명시적으로 백그라운드 실행이 필요할 때만 @concurrent 사용
@concurrent
func processLargeFile(_ url: URL) async throws -> Data {
    // 이 함수는 백그라운드 스레드에서 실행됩니다
    try Data(contentsOf: url)
}
```

**`nonisolated(nonsending)` 기본 동작**: `nonisolated async` 함수가 이제 호출자의 actor에서 실행됩니다. 불필요한 스레드 홉(thread hop)이 사라져 성능이 향상됩니다.

## 실습: 직접 해보기

Swift 6 마이그레이션 체크리스트입니다.

**마이그레이션 체크리스트:**

- [ ] Xcode Build Settings에서 "Strict Concurrency Checking"을 "Complete"로 설정
- [ ] 모델/유틸리티 등 말단 모듈부터 경고 수정 시작
- [ ] `class`에 `Sendable` 필요하면 `final` + 모든 프로퍼티 `let` 또는 `actor`로 변환
- [ ] 뷰모델에 `@MainActor` 추가
- [ ] 업데이트 안 된 라이브러리는 `@preconcurrency import` 사용
- [ ] `@unchecked Sendable` 대신 `Mutex` 또는 `actor` 사용 검토
- [ ] 모든 경고 해결 후 Swift Language Version을 6으로 변경
- [ ] Swift 6.2라면 `.defaultIsolation(MainActor.self)` 활성화 검토

## 더 깊이 알아보기

Swift의 동시성 안전은 긴 여정의 결과물입니다. 2021년 WWDC에서 async/await과 actor가 처음 소개되었을 때, 많은 개발자가 환호했지만 동시에 혼란스러워했어요. "어떤 코드가 어떤 스레드에서 돌아가는 거지?" 라는 질문이 끊이지 않았죠.

2024년 9월 Swift 6.0이 출시되면서 데이터 경쟁이 **컴파일 에러**로 바뀌었지만, 커뮤니티의 반응은 갈렸습니다. "안전하지만 너무 어렵다"는 피드백이 쏟아졌고, 2025년 2월 Swift 팀은 **"Improving the approachability of data-race safety"** 비전 문서를 발표하며 문제를 인정했습니다.

그 결과가 Swift 6.2의 **Approachable Concurrency**입니다. "대부분의 앱은 그렇게 많은 동시성이 필요하지 않다"는 현실적 인식 아래, 기본을 `@MainActor`로 두고 필요할 때만 백그라운드로 보내는 모델로 전환했습니다. 이것은 UIKit 시대의 "기본은 메인 스레드, 필요하면 DispatchQueue.global()"과 같은 직관적 모델로의 회귀이기도 합니다.

## 흔한 오해와 팁

> ⚠️ **흔한 오해**: "Swift 6 컴파일러를 쓰면 무조건 Swift 6 모드다" — Swift 6 **컴파일러**와 Swift 6 **언어 모드**는 다릅니다. Swift 6 컴파일러로도 Swift 5 언어 모드를 계속 사용할 수 있어요. 마이그레이션은 준비되었을 때 하면 됩니다.

> 🔥 **실무 팁**: 마이그레이션은 **말단 모듈부터** 시작하세요. 모델, 유틸리티 등 의존성이 적은 모듈을 먼저 Swift 6로 전환하고, 점차 앱 타겟으로 올라가는 것이 효율적입니다.

> 💡 **알고 계셨나요?**: Tony Hoare는 1965년에 발명한 null 참조를 "10억 달러짜리 실수"라고 불렀습니다. Swift의 옵셔널이 그 문제를 해결했듯, Sendable과 actor 모델은 데이터 경쟁이라는 또 다른 "수십억 달러짜리 실수"를 해결합니다.

## 핵심 정리

| 개념 | 설명 |
|------|------|
| Sendable | 격리 도메인 간 안전한 전달을 보장하는 마커 프로토콜 |
| @Sendable | 클로저가 안전하게 격리 도메인을 넘을 수 있다는 표시 |
| actor | 자체 격리 도메인을 가진 참조 타입 (외부 접근 시 await 필요) |
| @MainActor | 메인 스레드 실행을 보장하는 글로벌 actor |
| nonisolated | actor 격리에서 제외하는 키워드 |
| @preconcurrency | 아직 업데이트 안 된 모듈의 동시성 경고 억제 |
| Mutex | iOS 18+에서 @unchecked 없이 스레드 안전을 보장하는 동기화 도구 |
| @concurrent | Swift 6.2에서 명시적으로 백그라운드 실행을 요청하는 속성 |
| defaultIsolation | 모듈 전체의 기본 격리 도메인을 설정 (Swift 6.2) |

## 다음 섹션 미리보기

Swift 6가 언어의 안전성을 혁신했다면, **visionOS**는 Swift가 활약하는 무대를 3차원으로 확장합니다. [visionOS와 공간 컴퓨팅](./02-visionos.md)에서 Apple Vision Pro를 위한 공간 앱 개발의 기초를 배워봅시다.

## 참고 자료

- [Adopting strict concurrency in Swift 6 apps - Apple Developer](https://developer.apple.com/documentation/swift/adoptingswift6) - Swift 6 마이그레이션 공식 가이드
- [Migrate your app to Swift 6 - WWDC24](https://developer.apple.com/videos/play/wwdc2024/10169/) - 실습 기반 마이그레이션 세션
- [What's new in Swift - WWDC25](https://developer.apple.com/videos/play/wwdc2025/245/) - Approachable Concurrency 소개
- [Swift 6 Concurrency Migration Guide - Swift.org](https://www.swift.org/migration/documentation/swift-6-concurrency-migration-guide/migrationstrategy/) - 공식 마이그레이션 전략
- [Announcing Swift 6 - Swift.org](https://www.swift.org/blog/announcing-swift-6/) - Swift 6 출시 블로그
