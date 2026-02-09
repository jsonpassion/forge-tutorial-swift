# SwiftUI 렌더링 최적화

> 불필요한 뷰 업데이트 방지, body 최적화, Equatable

## 개요

SwiftUI는 선언적 UI 프레임워크답게 "상태가 바뀌면 화면을 다시 그린다"는 단순한 원칙을 따릅니다. 하지만 이 원칙을 제대로 이해하지 못하면 버튼 하나를 탭했는데 화면 전체가 다시 계산되는 비효율이 생기죠. 이번 섹션에서는 SwiftUI가 뷰를 언제, 왜 다시 그리는지 이해하고, 불필요한 업데이트를 줄이는 방법을 배웁니다.

**선수 지식**: [@State와 @Binding](../05-state-management/01-state-binding.md), [@Observable 매크로](../05-state-management/02-observable.md)
**학습 목표**:
- SwiftUI의 뷰 identity와 diffing 메커니즘을 이해할 수 있다
- _printChanges()로 불필요한 뷰 업데이트를 진단할 수 있다
- 뷰 분해와 Equatable을 활용해 렌더링 성능을 최적화할 수 있다

## 왜 알아야 할까?

"앱이 버벅거려요"라는 피드백은 대부분 UI 렌더링 성능 문제입니다. SwiftUI는 똑똑하지만 만능은 아니에요. 리스트에 수백 개의 아이템이 있고, 하나가 바뀔 때마다 전체가 다시 계산된다면? 스크롤이 뚝뚝 끊기겠죠. 최적화를 알면 같은 기능도 **부드럽고 빠르게** 만들 수 있습니다.

## 핵심 개념

### 개념 1: SwiftUI가 뷰를 다시 그리는 기준

> 💡 **비유**: SwiftUI는 **감독관**입니다. 각 뷰의 출석부(identity)를 들고, 상태가 바뀐 뷰만 "다시 해!"라고 시킵니다. 문제는 감독관이 때때로 불필요하게 넓은 범위에 "다시 해!"라고 외칠 수 있다는 거죠.

SwiftUI는 뷰를 **두 가지 방법**으로 구분합니다.

| 구분 방식 | 설명 | 예시 |
|-----------|------|------|
| **구조적 identity** | 뷰 계층에서의 위치로 구분 | VStack의 첫 번째 Text vs 두 번째 Text |
| **명시적 identity** | `.id()` 수정자나 ForEach의 id로 구분 | `ForEach(items, id: \.id)` |

뷰의 `body`가 다시 호출되는 3가지 트리거:

1. **@State, @Observable 등 상태 변경** — 해당 프로퍼티를 읽는 뷰만 업데이트
2. **부모로부터 받은 파라미터 변경** — 부모가 다시 그려지면 자식도 재평가
3. **이벤트 소스** — `onReceive`, `onChange` 등이 발동될 때

### 개념 2: _printChanges()로 진단하기

뷰가 왜 다시 그려지는지 모르겠을 때, `_printChanges()`를 사용합니다.

```swift
struct ItemListView: View {
    @State private var items: [String] = []
    @State private var searchText = ""

    var body: some View {
        // 디버깅용: 콘솔에 뷰가 다시 그려지는 이유 출력
        let _ = Self._printChanges()

        List(filteredItems, id: \.self) { item in
            Text(item)
        }
        .searchable(text: $searchText)
    }

    var filteredItems: [String] {
        searchText.isEmpty ? items : items.filter { $0.contains(searchText) }
    }
}
```

콘솔 출력 예시:
- `ItemListView: @self changed.` → 뷰 자체가 재생성됨
- `ItemListView: _searchText changed.` → searchText 상태가 변경됨

> 🔥 **실무 팁**: `_printChanges()`는 디버깅 전용입니다. **언더스코어로 시작하는 API는 프라이빗 API**이므로, 출시 전에 반드시 제거하세요. 하지만 성능 문제를 진단할 때는 이만한 도구가 없습니다.

### 개념 3: 뷰 분해 — 가장 효과적인 최적화

> 💡 **비유**: 큰 뷰 하나를 작은 뷰 여러 개로 나누는 건, **아파트 전체 조명 스위치를 방별로 나누는 것**과 같습니다. 화장실 불만 켜면 되는데 집 전체 조명을 켤 필요 없잖아요?

SwiftUI에서 가장 효과적인 최적화는 **뷰를 작게 분해**하는 것입니다.

```swift
// ❌ 나쁜 예: 모든 상태가 한 뷰에 묶여 있음
struct ProfileView: View {
    @State private var name = "김철수"
    @State private var bio = "iOS 개발자"
    @State private var followerCount = 0
    @State private var isEditing = false

    var body: some View {
        VStack {
            // name이 바뀌면 followerCount 표시도 다시 계산됨!
            Text(name).font(.title)
            Text(bio).foregroundStyle(.secondary)
            Text("팔로워 \(followerCount)명")
            Button(isEditing ? "완료" : "편집") {
                isEditing.toggle()
            }
        }
    }
}
```

```swift
// ✅ 좋은 예: 독립적인 상태를 가진 뷰로 분해
struct ProfileView: View {
    var body: some View {
        VStack {
            ProfileHeaderView()
            FollowerCountView()
            EditButtonView()
        }
    }
}

// 이름과 바이오만 관리 — followerCount 변경에 영향 없음
struct ProfileHeaderView: View {
    @State private var name = "김철수"
    @State private var bio = "iOS 개발자"

    var body: some View {
        VStack {
            Text(name).font(.title)
            Text(bio).foregroundStyle(.secondary)
        }
    }
}

// 팔로워 수만 관리 — 이름 변경에 영향 없음
struct FollowerCountView: View {
    @State private var count = 0

    var body: some View {
        Text("팔로워 \(count)명")
    }
}
```

### 개념 4: @Observable의 세밀한 추적

`@Observable`은 `ObservableObject`보다 훨씬 효율적입니다. 프로퍼티 단위로 의존성을 추적하거든요.

```swift
// @Observable: body에서 읽은 프로퍼티만 추적
@Observable
class UserSettings {
    var theme = "light"      // ThemeView만 추적
    var fontSize = 16        // FontView만 추적
    var notifications = true // NotificationView만 추적
}

struct ThemeView: View {
    var settings: UserSettings

    var body: some View {
        // settings.theme만 읽으므로
        // fontSize나 notifications가 바뀌어도 이 뷰는 다시 그려지지 않음!
        Text("현재 테마: \(settings.theme)")
    }
}
```

모델이 커지면 관심사별로 분리하는 것도 좋은 방법입니다.

```swift
// 큰 모델을 분리
@Observable class AppearanceSettings {
    var theme = "light"
    var fontSize = 16
    var accentColor = "blue"
}

@Observable class NotificationSettings {
    var pushEnabled = true
    var soundEnabled = true
}

// 각 뷰가 필요한 모델만 참조
struct AppearanceView: View {
    var settings: AppearanceSettings  // 알림 설정 변경에 영향 없음

    var body: some View {
        Text("테마: \(settings.theme)")
    }
}
```

### 개념 5: Equatable로 비교 제어하기

뷰가 Equatable을 준수하면 SwiftUI가 이전 값과 비교해서 실제로 바뀌었을 때만 body를 호출합니다.

```swift
struct ExpensiveView: View, Equatable {
    let item: Item
    let onTap: () -> Void

    // 클로저는 비교할 수 없으니, item만 비교
    static func == (lhs: Self, rhs: Self) -> Bool {
        lhs.item.id == rhs.item.id &&
        lhs.item.title == rhs.item.title
    }

    var body: some View {
        // 복잡한 렌더링 로직...
        VStack {
            Text(item.title)
            // ... 많은 하위 뷰들
        }
    }
}
```

### 개념 6: Lazy 컨테이너 활용

`List`는 이미 lazy하지만, `ScrollView` + `VStack` 조합은 아닙니다.

```swift
// ❌ 1000개를 한꺼번에 생성
ScrollView {
    VStack {
        ForEach(items) { item in
            ItemRow(item: item)
        }
    }
}

// ✅ 화면에 보이는 것만 생성
ScrollView {
    LazyVStack {
        ForEach(items) { item in
            ItemRow(item: item)
        }
    }
}
```

> ⚠️ **흔한 오해**: "LazyVStack이 항상 VStack보다 좋다" — 아닙니다! 아이템이 20개 이하라면 VStack이 오히려 빠를 수 있습니다. LazyVStack은 뷰를 재생성하는 비용이 있어서, 소량에서는 한 번에 만드는 게 효율적이에요.

## 실습: 직접 해보기

성능 최적화 전후를 비교해봅시다.

```swift
import SwiftUI

@Observable
class CounterModel {
    var count = 0
    var label = "카운터"
}

// 최적화된 카운터 뷰
struct OptimizedCounterView: View {
    var model: CounterModel

    var body: some View {
        VStack(spacing: 20) {
            // label이 바뀌어도 CountDisplay는 영향 없음
            CountDisplay(count: model.count)

            LabelDisplay(label: model.label)

            HStack {
                Button("증가") { model.count += 1 }
                    .buttonStyle(.borderedProminent)
                Button("라벨 변경") {
                    model.label = "카운터 #\(Int.random(in: 1...99))"
                }
                .buttonStyle(.bordered)
            }
        }
        .padding()
    }
}

struct CountDisplay: View {
    let count: Int

    var body: some View {
        let _ = Self._printChanges()  // 디버깅용
        Text("\(count)")
            .font(.system(size: 72, weight: .bold))
    }
}

struct LabelDisplay: View {
    let label: String

    var body: some View {
        let _ = Self._printChanges()  // 디버깅용
        Text(label)
            .foregroundStyle(.secondary)
    }
}

#Preview {
    OptimizedCounterView(model: CounterModel())
}
```

## 더 깊이 알아보기

WWDC 2021의 **"Demystify SwiftUI"** 세션은 SwiftUI 성능 이해의 바이블입니다. Apple 엔지니어가 SwiftUI가 뷰를 어떻게 구분하고(identity), 언제 변화를 감지하며(lifetime), 어떻게 의존성을 추적하는지(dependency)를 처음으로 공개한 세션이죠. 이후 WWDC 2023에서는 **"Demystify SwiftUI Performance"** 세션이 이어졌는데, Instruments의 SwiftUI 프로파일링 도구와 body 호출 횟수 추적 기능이 소개되었습니다.

`@Observable`이 `ObservableObject`보다 효율적인 이유는 **의존성 추적 방식**이 다르기 때문입니다. `ObservableObject`는 `objectWillChange` 퍼블리셔가 발동하면 그 객체를 관찰하는 **모든 뷰**가 다시 그려집니다. 반면 `@Observable`은 Swift 매크로가 각 프로퍼티의 getter에 추적 코드를 삽입해서, **body에서 실제로 읽은 프로퍼티**만 추적합니다.

## 흔한 오해와 팁

> 💡 **알고 계셨나요?**: SwiftUI의 `body`는 매 프레임마다 호출되는 게 아닙니다. 상태가 변경될 때만 호출됩니다. 그리고 `body`가 호출된다고 화면이 반드시 다시 그려지는 것도 아닙니다. SwiftUI는 이전 결과와 비교(diff)해서 실제로 바뀐 부분만 렌더링합니다.

> 🔥 **실무 팁**: 복잡한 `drawingGroup()`이나 Metal 최적화보다, **뷰 분해**가 가장 효과적인 최적화입니다. 대부분의 성능 문제는 하나의 거대한 뷰에 모든 상태를 넣어서 생깁니다.

## 핵심 정리

| 개념 | 설명 |
|------|------|
| 구조적 identity | 뷰 계층에서의 위치로 뷰를 구분 |
| _printChanges() | 뷰가 다시 그려지는 이유를 콘솔에 출력 (디버깅용) |
| 뷰 분해 | 독립적 상태를 가진 작은 뷰로 나눠 업데이트 범위 축소 |
| @Observable | 프로퍼티 단위 의존성 추적으로 정밀한 업데이트 |
| Equatable | 커스텀 비교로 불필요한 body 호출 방지 |
| LazyVStack | 화면에 보이는 뷰만 생성하는 지연 로딩 컨테이너 |

## 다음 섹션 미리보기

UI 렌더링 최적화를 알았으니, 이제 눈에 보이지 않는 곳의 성능을 다룰 차례입니다. [메모리 관리와 ARC](./03-memory-arc.md)에서 Swift의 메모리 관리 원리와 메모리 누수를 잡는 방법을 배워봅시다.

## 참고 자료

- [Demystify SwiftUI - WWDC21](https://developer.apple.com/videos/play/wwdc2021/10022/) - SwiftUI identity, lifetime, dependency 핵심 세션
- [Demystify SwiftUI Performance - WWDC23](https://developer.apple.com/videos/play/wwdc2023/10160/) - SwiftUI 성능 프로파일링
- [Avoiding Repeated SwiftUI View Updates - Fatbobman](https://fatbobman.com/en/posts/avoid_repeated_calculations_of_swiftui_views/) - 뷰 업데이트 최적화 심층 가이드
- [SwiftUI Performance Tips - Apple Developer](https://developer.apple.com/documentation/swiftui/performance) - 공식 성능 가이드
