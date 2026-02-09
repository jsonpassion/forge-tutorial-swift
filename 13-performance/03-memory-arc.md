# 메모리 관리와 ARC

> ARC 동작 원리, 순환 참조, weak/unowned, 메모리 릭 탐지

## 개요

Swift는 **ARC(Automatic Reference Counting)**로 메모리를 관리합니다. 개발자가 직접 메모리를 할당하고 해제할 필요 없이, 컴파일러가 적절한 시점에 자동으로 처리해주죠. 하지만 "자동"이 "완벽"을 의미하진 않습니다. 순환 참조라는 함정을 이해하지 못하면 메모리 누수가 발생합니다.

**선수 지식**: [구조체와 클래스](../02-swift-types/01-struct-class.md), [Swift Concurrency 심화](./01-concurrency-deep.md)
**학습 목표**:
- ARC의 동작 원리(참조 카운트)를 이해할 수 있다
- 순환 참조가 발생하는 패턴을 식별할 수 있다
- weak/unowned를 적절히 사용해 순환 참조를 해결할 수 있다
- Xcode Memory Graph Debugger로 메모리 누수를 탐지할 수 있다

## 왜 알아야 할까?

메모리 누수는 앱이 **조금씩, 천천히 죽는** 문제입니다. 당장은 잘 동작하지만, 오래 사용하면 메모리가 계속 쌓여서 결국 시스템이 앱을 강제 종료합니다. 특히 화면 전환이 많은 앱에서, 뒤로 갔는데 이전 화면이 메모리에서 해제되지 않는 문제가 흔하죠. 사용자는 "앱이 자꾸 튕겨요"라고 불평하게 됩니다.

## 핵심 개념

### 개념 1: ARC — 자동 참조 카운팅

> 💡 **비유**: ARC는 **도서관 대출 시스템**입니다. 책(객체)을 빌리면 대출 카운트가 1 올라가고, 반납하면 1 내려가죠. 카운트가 0이 되면 책꽂이(메모리)에서 정리됩니다.

참조 타입(class)만 ARC의 관리 대상입니다. 값 타입(struct, enum)은 복사되므로 참조 카운트가 필요 없어요.

```swift
class Person {
    let name: String

    init(name: String) {
        self.name = name
        print("\(name) 생성됨")
    }

    deinit {
        print("\(name) 해제됨")
    }
}

// 참조 카운트 변화를 관찰해봅시다
var ref1: Person? = Person(name: "철수")  // 카운트: 1
var ref2 = ref1                           // 카운트: 2
var ref3 = ref1                           // 카운트: 3

ref1 = nil  // 카운트: 2 (아직 해제 안 됨)
ref2 = nil  // 카운트: 1 (아직 해제 안 됨)
ref3 = nil  // 카운트: 0 → "철수 해제됨" 출력!
```

### 개념 2: 순환 참조 — ARC의 맹점

> 💡 **비유**: 두 사람이 서로의 손을 잡고 있으면 둘 다 놓지 못합니다. A가 B를 잡고, B가 A를 잡으면 **교착 상태**죠. ARC에서도 두 객체가 서로를 강하게 참조하면 둘 다 해제되지 않습니다.

```swift
// ❌ 순환 참조 발생!
class Employee {
    let name: String
    var team: Team?  // 강한 참조

    init(name: String) { self.name = name }
    deinit { print("\(name) 해제됨") }
}

class Team {
    let name: String
    var leader: Employee?  // 강한 참조

    init(name: String) { self.name = name }
    deinit { print("팀 \(name) 해제됨") }
}

var employee: Employee? = Employee(name: "김개발")
var team: Team? = Team(name: "iOS팀")

employee?.team = team    // Employee → Team (강한)
team?.leader = employee  // Team → Employee (강한)

// nil로 설정해도 순환 참조 때문에 해제되지 않음!
employee = nil  // deinit 호출 안 됨 ⚠️
team = nil      // deinit 호출 안 됨 ⚠️
// 메모리 누수 발생!
```

### 개념 3: weak과 unowned — 순환 고리 끊기

순환 참조의 한쪽을 `weak` 또는 `unowned`로 바꾸면 됩니다.

```swift
// ✅ weak으로 순환 참조 해결
class Employee {
    let name: String
    var team: Team?

    init(name: String) { self.name = name }
    deinit { print("\(name) 해제됨") }
}

class Team {
    let name: String
    weak var leader: Employee?  // 약한 참조 (카운트 증가 안 함)

    init(name: String) { self.name = name }
    deinit { print("팀 \(name) 해제됨") }
}

var employee: Employee? = Employee(name: "김개발")
var team: Team? = Team(name: "iOS팀")

employee?.team = team
team?.leader = employee

employee = nil  // "김개발 해제됨" ✅
team = nil      // "팀 iOS팀 해제됨" ✅
```

| 키워드 | 참조 카운트 | nil 가능 | 사용 시점 |
|--------|------------|---------|----------|
| `strong` | 증가 | - | 기본값, 소유 관계 |
| `weak` | 증가 안 함 | Optional | 상대방이 먼저 해제될 수 있을 때 |
| `unowned` | 증가 안 함 | Non-optional | 상대방이 항상 존재할 때 |

> ⚠️ **흔한 오해**: "`unowned`가 `weak`보다 빠르니까 항상 `unowned`를 쓰자" — `unowned`는 참조 대상이 먼저 해제되면 **크래시**합니다! 확신이 없으면 `weak`을 쓰세요. 성능 차이는 미미합니다.

### 개념 4: 클로저의 캡처 리스트

클로저에서 `self`를 캡처할 때 순환 참조가 가장 많이 발생합니다.

```swift
// ❌ 순환 참조: self → closure → self
class NetworkManager {
    var onComplete: (() -> Void)?
    var data: [String] = []

    func fetchData() {
        onComplete = {
            // self를 강하게 캡처 → 순환 참조!
            self.data.append("새 데이터")
        }
    }

    deinit { print("NetworkManager 해제됨") }
}
```

```swift
// ✅ [weak self]로 해결
class NetworkManager {
    var onComplete: (() -> Void)?
    var data: [String] = []

    func fetchData() {
        onComplete = { [weak self] in
            guard let self else { return }  // 옵셔널 바인딩
            self.data.append("새 데이터")
        }
    }

    deinit { print("NetworkManager 해제됨") }
}
```

SwiftUI에서 흔한 패턴:

```swift
@Observable
class TimerViewModel {
    var count = 0
    private var timer: Timer?

    func startTimer() {
        // Timer는 클로저를 강하게 캡처하므로 [weak self] 필수
        timer = Timer.scheduledTimer(withTimeInterval: 1, repeats: true) { [weak self] _ in
            self?.count += 1
        }
    }

    func stopTimer() {
        timer?.invalidate()
        timer = nil
    }

    deinit {
        stopTimer()
        print("TimerViewModel 해제됨")
    }
}
```

### 개념 5: Memory Graph Debugger — 누수 탐지

Xcode의 Memory Graph Debugger는 실행 중인 앱의 모든 객체와 참조 관계를 시각화합니다.

**사용 방법:**

1. 앱을 실행한 상태에서 Xcode 하단 **디버그 바**의 메모리 아이콘(세 개의 원) 클릭
2. 또는 **Debug → Debug Memory Graph** 메뉴 선택
3. 왼쪽 패널에서 해제되지 않은 객체를 확인
4. 보라색 느낌표(!) 아이콘이 있으면 **누수 의심** 객체

**누수를 찾는 실전 팁:**

- 화면을 push한 뒤 pop하고, Memory Graph를 확인합니다
- pop한 화면의 ViewModel이 여전히 살아있다면 누수!
- 객체를 클릭하면 오른쪽에 참조 그래프가 나옵니다
- 순환 참조라면 화살표가 원형으로 연결된 것이 보입니다

> 🔥 **실무 팁**: `deinit`에 print를 넣어두면 객체가 제대로 해제되는지 빠르게 확인할 수 있습니다. 특히 ViewModel과 Coordinator에 넣어두면 화면 전환 시 누수를 즉시 발견할 수 있어요.

## 실습: 직접 해보기

순환 참조를 직접 만들고 해결해봅시다.

```swift
import SwiftUI

// 순환 참조가 해결된 안전한 구현
@Observable
class ChatRoom {
    let name: String
    var messages: [String] = []
    private var onNewMessage: ((String) -> Void)?

    init(name: String) {
        self.name = name
        print("ChatRoom '\(name)' 생성됨")
    }

    // [weak self] 패턴으로 안전하게 구현
    func startListening() {
        onNewMessage = { [weak self] message in
            self?.messages.append(message)
        }
    }

    func simulateMessage(_ text: String) {
        onNewMessage?(text)
    }

    deinit {
        print("ChatRoom '\(name)' 해제됨 ✅")
    }
}

struct ChatView: View {
    @State private var room: ChatRoom? = ChatRoom(name: "Swift 스터디")
    @State private var newMessage = ""

    var body: some View {
        VStack {
            if let room {
                List(room.messages, id: \.self) { msg in
                    Text(msg)
                }

                HStack {
                    TextField("메시지", text: $newMessage)
                    Button("전송") {
                        room.simulateMessage(newMessage)
                        newMessage = ""
                    }
                }
                .padding()
            }

            Button("채팅방 나가기") {
                room = nil  // deinit이 호출되어야 정상!
            }
            .foregroundStyle(.red)
        }
        .onAppear {
            room?.startListening()
        }
    }
}

#Preview {
    ChatView()
}
```

## 더 깊이 알아보기

ARC는 Apple이 2011년 Objective-C에 도입한 기술입니다. 그 전에는 개발자가 직접 `retain`/`release`를 호출해야 했는데, 수동 메모리 관리는 버그의 온상이었죠. Java나 C# 같은 언어는 **가비지 컬렉션(GC)**을 사용하는데, Apple이 GC 대신 ARC를 선택한 이유가 있습니다. GC는 주기적으로 전체 메모리를 스캔하면서 "멈춤 현상(stop-the-world)"을 일으킵니다. 60fps를 유지해야 하는 모바일 앱에서는 치명적이죠. ARC는 참조 카운트가 0이 되는 **즉시** 해제하므로, 예측 가능하고 일정한 성능을 보장합니다.

한편, null 참조의 발명자 Tony Hoare는 2009년에 이를 **"10억 달러짜리 실수"**라고 공개적으로 사과했습니다. Swift의 옵셔널 시스템은 이 문제를 해결하기 위한 것이고, `weak` 참조가 Optional인 것도 "참조 대상이 사라질 수 있음"을 타입 시스템으로 표현한 거죠.

## 흔한 오해와 팁

> ⚠️ **흔한 오해**: "SwiftUI에서는 메모리 누수가 안 생긴다" — `@Observable` 클래스에서 클로저 캡처, Timer, NotificationCenter 등을 사용하면 SwiftUI에서도 누수가 발생합니다. `deinit`이 호출되는지 항상 확인하세요.

> 💡 **알고 계셨나요?**: `struct`는 참조 카운트가 없어서 ARC 오버헤드가 전혀 없습니다. 그래서 Swift에서 struct를 기본으로 권장하는 거예요. class가 필요한 경우(참조 공유, 상속)에만 class를 쓰세요.

## 핵심 정리

| 개념 | 설명 |
|------|------|
| ARC | 참조 카운트가 0이 되면 자동으로 메모리 해제 |
| 순환 참조 | 두 객체가 서로를 강하게 참조해 해제되지 않는 상태 |
| weak | 참조 카운트를 증가시키지 않는 약한 참조 (Optional) |
| unowned | 참조 카운트를 증가시키지 않는 약한 참조 (Non-optional, 위험) |
| [weak self] | 클로저에서 self를 약하게 캡처하는 캡처 리스트 |
| Memory Graph | Xcode에서 객체 참조 관계를 시각화하는 디버깅 도구 |
| deinit | 객체가 해제될 때 호출되는 소멸자 |

## 다음 섹션 미리보기

메모리 누수를 코드 레벨에서 이해했으니, 이제 도구를 사용해 앱 전체의 성능을 분석해봅시다. [Instruments 프로파일링](./04-instruments.md)에서 Time Profiler, Allocations, Leaks 등 전문 프로파일링 도구 사용법을 배웁니다.

## 참고 자료

- [Automatic Reference Counting - Swift.org](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/automaticreferencecounting/) - ARC 공식 문서
- [Gathering Information About Memory Use - Apple Developer](https://developer.apple.com/documentation/xcode/gathering-information-about-memory-use) - Xcode 메모리 디버깅 가이드
- [Managing self and cancellable references in Combine - Swift by Sundell](https://www.swiftbysundell.com/articles/combine-self-cancellable-memory-management/) - Combine 메모리 관리
- [Diagnose Memory Issues - WWDC21](https://developer.apple.com/videos/play/wwdc2021/10180/) - 메모리 진단 세션
