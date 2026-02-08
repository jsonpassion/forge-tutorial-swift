# Hello, SwiftUI!

> View 프로토콜, body, #Preview, 프로젝트 구조

## 개요

드디어 화면에 무언가를 그릴 시간입니다! 지금까지 Swift 언어의 기초를 다졌으니, 이제 그 위에 아름다운 UI를 올려볼 차례인데요. 이 섹션에서는 SwiftUI의 기본 구조를 이해하고, 첫 번째 앱을 만들어봅니다.

**선수 지식**: [Ch2. Swift 타입 시스템](../02-swift-types/01-struct-class.md)에서 배운 구조체와 프로토콜
**학습 목표**:
- SwiftUI가 무엇이고 왜 혁명적인지 이해하기
- View 프로토콜과 body 프로퍼티의 역할 파악하기
- Xcode에서 SwiftUI 프로젝트를 생성하고 #Preview로 실시간 확인하기

## 왜 알아야 할까?

여러분이 Swift를 배우는 궁극적인 이유는 **앱을 만들기 위해서**죠. SwiftUI는 Apple이 제공하는 최신 UI 프레임워크로, iPhone, iPad, Mac, Apple Watch, Apple TV, Vision Pro까지 **모든 Apple 플랫폼**의 앱을 하나의 코드로 만들 수 있게 해줍니다.

과거의 UIKit은 "이 버튼을 여기에 추가하고, 크기를 이렇게 바꾸고, 색을 저렇게 칠해라"라고 **명령**하는 방식이었습니다. 반면 SwiftUI는 "이런 모습이면 좋겠어"라고 **선언**하면 시스템이 알아서 그려주는 방식이에요. 마치 요리사에게 "파스타 만들어주세요"라고 말하는 것과 "냄비에 물을 붓고, 불을 켜고, 면을 넣고..."라고 하나하나 지시하는 것의 차이라고 할 수 있습니다.

## 핵심 개념

### 개념 1: SwiftUI란?

> 💡 **비유**: SwiftUI는 **레고 설명서**와 같습니다. "이런 모양을 만들고 싶다"고 그림으로 보여주면, 레고 시스템이 알아서 블록을 조립해주는 거죠. 여러분은 최종 모습만 선언하면 됩니다.

SwiftUI는 2019년 WWDC에서 발표된 Apple의 **선언형(Declarative) UI 프레임워크**입니다. "선언형"이라는 말이 어렵게 들릴 수 있는데, 쉽게 말하면 **"무엇을(What)" 보여줄지만 말하면 "어떻게(How)" 그릴지는 프레임워크가 알아서 처리**한다는 뜻이에요.

| 비교 | 명령형 (UIKit) | 선언형 (SwiftUI) |
|------|---------------|-----------------|
| 방식 | 단계별로 지시 | 최종 모습을 선언 |
| 코드량 | 많음 | 적음 |
| 상태 관리 | 수동 동기화 | 자동 반영 |
| 실시간 미리보기 | 빌드 필요 | #Preview로 즉시 확인 |
| 멀티 플랫폼 | 플랫폼별 별도 코드 | 하나의 코드로 대응 |

### 개념 2: View 프로토콜과 body

> 💡 **비유**: View는 **액자 틀**이고, body는 **그 안에 넣을 그림**입니다. 모든 SwiftUI 화면 요소는 "나는 View야, 내 모습은 body에 있어"라고 약속하는 거죠.

SwiftUI에서 화면에 보이는 모든 것은 `View` 프로토콜을 따르는 구조체입니다. [앞서 프로토콜](../02-swift-types/03-protocols-extensions.md)에서 배운 것처럼, View 프로토콜은 "body라는 프로퍼티를 제공해야 해"라는 약속이에요.

```swift
import SwiftUI

// View 프로토콜을 따르는 구조체
struct ContentView: View {
    // body는 이 뷰가 어떻게 보일지 선언하는 곳
    var body: some View {
        Text("안녕하세요, SwiftUI!")
    }
}
```

여기서 `some View`가 눈에 띄죠? 이것은 "어떤 구체적인 View 타입을 반환하는데, 정확히 뭔지는 컴파일러가 알아서 파악해"라는 뜻입니다. Swift의 **불투명 반환 타입(Opaque Return Type)** 기능인데, 지금은 "body 앞에 항상 `some View`를 쓴다"고만 기억하면 충분합니다.

> ⚠️ **흔한 오해**: "SwiftUI View는 화면에 그려지는 실체다" — 사실 SwiftUI의 View 구조체는 **화면 설명서**에 가깝습니다. SwiftUI가 이 설명서를 읽고 실제 렌더링을 수행하죠. View 구조체는 매우 가볍기 때문에 자주 만들어지고 버려져도 성능 문제가 없습니다.

### 개념 3: @main과 App 프로토콜

> 💡 **비유**: `@main`은 건물의 **정문**입니다. 앱이 실행되면 가장 먼저 이 문을 통해 들어오죠.

SwiftUI 앱의 시작점은 `App` 프로토콜을 따르는 구조체입니다. `@main`이라는 표시가 붙어서 "여기가 앱의 진입점이야"라고 시스템에 알려줍니다.

```swift
import SwiftUI

// @main: 이것이 앱의 시작점임을 표시
@main
struct MyFirstApp: App {
    // Scene은 앱의 화면 구성 단위
    var body: some Scene {
        // WindowGroup은 앱의 메인 윈도우
        WindowGroup {
            // 처음 보여줄 뷰
            ContentView()
        }
    }
}
```

구조를 정리하면 이렇습니다:

- **App** → 앱 전체를 대표 (건물 전체)
- **Scene** → 화면 구성 단위 (층)
- **View** → 화면 요소 하나하나 (방 안의 가구)

### 개념 4: #Preview 매크로

> 💡 **비유**: #Preview는 **거울**입니다. 옷을 갈아입을 때마다(코드를 수정할 때마다) 거울에 바로 내 모습이 비치듯, 코드 변경 사항이 실시간으로 반영되죠.

Xcode의 Canvas에서 코드를 수정하면 실시간으로 미리보기가 업데이트됩니다. 이것을 가능하게 하는 것이 `#Preview` 매크로예요.

```swift
// 기본 Preview
#Preview {
    ContentView()
}

// 이름을 붙인 Preview (여러 상태를 비교할 때 유용)
#Preview("다크 모드") {
    ContentView()
        .preferredColorScheme(.dark)
}

#Preview("라이트 모드") {
    ContentView()
        .preferredColorScheme(.light)
}
```

> 💡 **알고 계셨나요?**: `#Preview`는 Swift 5.9에서 도입된 매크로 기능입니다. 이전에는 `PreviewProvider` 프로토콜을 사용했는데, 매크로 방식이 훨씬 간결하죠. 과거 방식의 코드를 보더라도 놀라지 마세요 — 지금은 `#Preview`가 표준입니다.

### 개념 5: 수정자(Modifier)

> 💡 **비유**: 수정자는 **스티커 꾸미기**와 같습니다. 기본 뷰에 스티커를 하나씩 붙여가듯, `.font()`, `.foregroundStyle()`, `.padding()` 같은 수정자를 체이닝해서 뷰를 꾸밀 수 있어요.

SwiftUI에서는 뷰 뒤에 점(`.`)으로 수정자를 연결해서 외관과 동작을 바꿉니다. 이것을 **수정자 체이닝(Modifier Chaining)** 이라고 합니다.

```swift
struct ContentView: View {
    var body: some View {
        Text("Hello, SwiftUI!")
            .font(.largeTitle)           // 큰 제목 폰트
            .fontWeight(.bold)           // 굵게
            .foregroundStyle(.blue)      // 파란색 텍스트
            .padding()                   // 주변에 여백 추가
            .background(.yellow)         // 노란색 배경
            .clipShape(RoundedRectangle(cornerRadius: 12))  // 둥근 모서리
    }
}

#Preview {
    ContentView()
}
```

> ⚠️ **흔한 오해**: "수정자 순서는 상관없다" — **순서가 매우 중요합니다!** `.padding()` 후에 `.background()`를 하면 패딩 영역까지 배경색이 칠해지지만, 순서를 바꾸면 결과가 완전히 달라집니다. 수정자는 **위에서 아래로** 순서대로 적용된다고 기억하세요.

## 실습: 첫 번째 SwiftUI 앱 만들기

Xcode에서 새 프로젝트를 만들어봅시다!

**프로젝트 생성 단계:**
1. Xcode를 열고 **Create New Project** 선택
2. **iOS** → **App** 선택
3. Product Name에 "MyFirstApp" 입력
4. **Interface**: SwiftUI, **Language**: Swift 확인
5. **Storage**: None 선택 (나중에 SwiftData를 배울 때 바꿉니다)
6. **Create** 클릭

생성된 프로젝트에서 `ContentView.swift`를 열면 기본 코드가 보입니다. 이것을 다음과 같이 수정해보세요:

```swift
import SwiftUI

struct ContentView: View {
    var body: some View {
        VStack(spacing: 16) {
            // 앱 아이콘 역할의 이미지
            Image(systemName: "swift")
                .font(.system(size: 60))
                .foregroundStyle(.orange)

            // 환영 메시지
            Text("Hello, SwiftUI!")
                .font(.largeTitle)
                .fontWeight(.bold)

            // 부가 설명
            Text("첫 번째 SwiftUI 앱을 만들었습니다!")
                .font(.subheadline)
                .foregroundStyle(.secondary)
        }
        .padding()
    }
}

#Preview {
    ContentView()
}
```

Canvas에서 **Resume** 버튼을 누르면 (또는 `Cmd + Option + P`) 미리보기가 나타납니다. 코드를 수정할 때마다 미리보기가 자동으로 업데이트되는 걸 확인해보세요!

**도전 과제**: `Text`의 내용이나 `.font()`, `.foregroundStyle()` 값을 바꿔가며 미리보기가 즉시 반영되는 것을 체험해보세요.

## 더 깊이 알아보기

### SwiftUI의 탄생 이야기

SwiftUI는 WWDC 2019에서 처음 공개되었는데요, 발표 순간 **개발자들이 기립박수**를 쳤습니다. 그만큼 혁명적이었거든요. UIKit으로 복잡한 UI를 만들려면 수백 줄의 코드가 필요했는데, SwiftUI로는 몇십 줄이면 충분했으니까요.

사실 SwiftUI의 아이디어는 갑자기 나온 게 아닙니다. React(2013)와 Flutter(2017) 같은 선언형 UI 프레임워크가 웹과 모바일에서 큰 성공을 거뒀고, Apple도 이 흐름을 따라간 것이죠. 하지만 Apple은 한 발 더 나아가 Swift 언어와의 **깊은 통합**을 이뤘습니다. `@State`, `@Binding` 같은 프로퍼티 래퍼와 `some View` 같은 언어 기능이 SwiftUI를 위해 추가되었을 정도예요.

### SwiftUI의 진화

| 버전 | 연도 | 주요 추가 기능 |
|------|------|--------------|
| SwiftUI 1.0 | 2019 (iOS 13) | 기본 프레임워크 출시 |
| SwiftUI 2.0 | 2020 (iOS 14) | App 프로토콜, @main, LazyVStack |
| SwiftUI 3.0 | 2021 (iOS 15) | async/await 지원, Material |
| SwiftUI 4.0 | 2022 (iOS 16) | NavigationStack, Charts, Grid |
| SwiftUI 5.0 | 2023 (iOS 17) | @Observable, #Preview 매크로 |
| SwiftUI 6.0 | 2024 (iOS 18) | 커스텀 컨테이너, 메시 그래디언트 |
| SwiftUI 7.0 | 2025 (iOS 26) | Liquid Glass, 새로운 디자인 언어 |

매년 WWDC에서 SwiftUI가 크게 발전하고 있어서, iOS 26 기준으로 배우는 여러분은 가장 완성도 높은 SwiftUI를 경험하게 됩니다!

## 흔한 오해와 팁

> ⚠️ **흔한 오해**: "SwiftUI가 나왔으니 UIKit은 배울 필요 없다" — SwiftUI가 점점 강력해지고 있지만, 아직도 일부 고급 기능은 UIKit 브릿지가 필요합니다. 하지만 **새 프로젝트라면 SwiftUI로 시작**하는 것이 정답입니다. UIKit은 필요할 때 배워도 늦지 않아요.

> 🔥 **실무 팁**: Xcode의 Canvas가 멈추거나 미리보기가 안 나올 때는 `Cmd + Option + P`를 눌러 Canvas를 다시 시작하세요. 그래도 안 되면 **Product → Clean Build Folder** (`Cmd + Shift + K`)를 시도해보세요.

> 💡 **알고 계셨나요?**: SwiftUI의 View 구조체는 **값 타입(Value Type)** 입니다. [앞서 구조체](../02-swift-types/01-struct-class.md)에서 배운 것처럼 복사될 때 독립적인 사본이 만들어지죠. 이 덕분에 SwiftUI는 뷰 상태를 효율적으로 비교하고, 변경된 부분만 다시 그릴 수 있습니다.

## 핵심 정리

| 개념 | 설명 |
|------|------|
| SwiftUI | Apple의 선언형 UI 프레임워크. "무엇을" 보여줄지만 선언하면 됨 |
| View 프로토콜 | 모든 화면 요소가 따르는 약속. body 프로퍼티를 필수로 구현 |
| body | 뷰가 어떻게 보일지 선언하는 연산 프로퍼티. `some View`를 반환 |
| @main + App | 앱의 진입점. WindowGroup으로 첫 화면을 지정 |
| #Preview | Xcode에서 실시간 미리보기를 제공하는 매크로 |
| 수정자(Modifier) | `.font()`, `.padding()` 등으로 뷰를 꾸미는 메서드 체이닝 |
| 수정자 순서 | 수정자는 위에서 아래로 순서대로 적용되므로 순서가 결과에 영향 |

## 다음 섹션 미리보기

SwiftUI의 기본 구조를 이해했으니, 다음으로 [02. 텍스트와 이미지](./02-text-image.md)에서 `Text`와 `Image`를 자유자재로 다루는 법을 배웁니다. 폰트, 색상, SF Symbols, 이미지 리사이징까지 — 앱의 기본 재료들을 마스터해볼 거예요.

## 참고 자료

- [Apple SwiftUI 공식 문서](https://developer.apple.com/documentation/swiftui) - SwiftUI 전체 API 레퍼런스
- [SwiftUI Tutorials](https://developer.apple.com/tutorials/swiftui) - Apple 공식 SwiftUI 튜토리얼 (Landmarks 앱)
- [WWDC 2019 - Introducing SwiftUI](https://developer.apple.com/videos/play/wwdc2019/204/) - SwiftUI 최초 공개 세션
- [WWDC 2023 - Discover Observation in SwiftUI](https://developer.apple.com/videos/play/wwdc2023/10149/) - #Preview 매크로 포함 최신 기능
- [Swift.org - Macros](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/macros/) - #Preview의 기반이 되는 Swift 매크로 이해
