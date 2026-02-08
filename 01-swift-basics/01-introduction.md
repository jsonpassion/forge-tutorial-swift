# 개발 환경 설정

> Xcode 설치, Playground, 첫 번째 프로그램 실행

## 개요

Swift로 iOS 앱을 만들려면 먼저 **작업 공간**이 필요합니다. 목수에게 작업대와 도구가 필요하듯, 우리에게는 **Xcode**라는 강력한 도구가 있죠. 이 섹션에서는 Xcode를 설치하고, Playground에서 Swift 코드를 직접 실행해 봅니다.

**선수 지식**: 없음 (완전 처음부터 시작합니다!)
**학습 목표**:
- Xcode를 설치하고 기본 인터페이스를 이해한다
- Playground에서 Swift 코드를 작성하고 실행한다
- 첫 번째 SwiftUI 프로젝트를 생성하고 시뮬레이터에서 실행한다

## 왜 알아야 할까?

모든 iOS 앱은 Xcode에서 태어납니다. App Store에 올라간 수백만 개의 앱이 전부 이 도구 하나에서 만들어졌다는 게 놀랍지 않나요? Xcode는 코드 편집, UI 디자인, 디버깅, 테스트, 앱스토어 배포까지 **모든 과정을 하나로** 해결하는 통합 개발 환경(IDE)입니다.

특히 **Playground**는 코드를 작성하면 결과를 즉시 확인할 수 있어서, Swift 문법을 배울 때 가장 빠르게 피드백을 받을 수 있는 최고의 연습장이에요.

## 핵심 개념

### 개념 1: Xcode란?

> 💡 **비유**: Xcode는 **만능 요리 스튜디오**입니다. 재료 손질(코드 작성), 요리(빌드), 맛보기(시뮬레이터), 포장(앱스토어 배포)까지 한 공간에서 전부 가능하죠.

Xcode는 Apple이 무료로 제공하는 공식 개발 도구입니다. Swift와 SwiftUI로 iPhone, iPad, Mac, Apple Watch, Vision Pro 앱을 모두 만들 수 있어요.

**Xcode에 포함된 것들:**

| 구성 요소 | 역할 |
|----------|------|
| **코드 에디터** | Swift 코드 작성, 자동완성, 문법 강조 |
| **Interface Builder** | 드래그 & 드롭으로 UI 디자인 (SwiftUI에서는 Preview 사용) |
| **시뮬레이터** | 실제 기기 없이 앱을 테스트 |
| **Instruments** | 성능 분석과 메모리 디버깅 |
| **Playground** | Swift 코드를 즉시 실행해 볼 수 있는 실험 공간 |
| **Coding Intelligence** | AI 기반 코드 제안과 자동완성 (Xcode 26 신기능) |

### 개념 2: Xcode 설치하기

Xcode를 설치하는 방법은 두 가지입니다.

**방법 1: Mac App Store (추천)**

1. Mac에서 **App Store**를 엽니다
2. 검색창에 **Xcode**를 입력합니다
3. **받기** 버튼을 클릭합니다
4. 설치가 완료되면 **열기**를 클릭합니다

**방법 2: Apple Developer 웹사이트**

베타 버전이나 특정 버전이 필요하다면 [Apple Developer](https://developer.apple.com/xcode/) 사이트에서 직접 다운로드할 수 있습니다.

> ⚠️ **흔한 오해**: "iOS 앱 개발에는 최신 고사양 Mac이 필수다" — 사실 Apple Silicon(M1 이상) Mac이라면 메모리 8GB로도 충분히 시작할 수 있습니다. 다만 Xcode 자체가 약 12GB 이상이므로 **저장 공간은 넉넉히 확보**해 두세요.

**시스템 요구 사항 (Xcode 26 기준):**

| 항목 | 최소 사양 |
|------|----------|
| **운영 체제** | macOS Sequoia 15.6 이상 |
| **저장 공간** | 약 35GB 이상 (Xcode + 시뮬레이터) |
| **프로세서** | Apple Silicon 또는 Intel (AI 기능은 Apple Silicon 필요) |
| **RAM** | 8GB 이상 (16GB 권장) |

> 🔥 **실무 팁**: Xcode를 처음 설치한 후 실행하면 "추가 구성 요소를 설치합니다"라는 메시지가 나타날 수 있습니다. 이건 시뮬레이터 런타임 등 필수 도구를 설치하는 과정이니 꼭 완료해 주세요.

### 개념 3: Playground — Swift의 놀이터

> 💡 **비유**: Playground는 **스케치북**입니다. 완성된 그림을 그리기 전에 자유롭게 연습하고 실험하는 공간이죠. 코드를 한 줄 작성하면 바로 옆에 결과가 표시됩니다.

Playground를 만들어 봅시다:

1. Xcode를 실행합니다
2. 메뉴에서 **File → New → Playground** 를 선택합니다
3. **Blank**를 선택하고 **Next**를 클릭합니다
4. 이름을 입력하고 (예: "SwiftPractice") **Create**를 클릭합니다

첫 번째 코드를 작성해 볼까요?

```swift
import Foundation

// 🎉 첫 번째 Swift 코드!
print("안녕하세요, Swift!")

// 간단한 계산
let year = 2026
let swiftAge = year - 2014  // Swift가 공개된 2014년부터 계산
print("Swift는 올해로 \(swiftAge)살입니다!")

// 변수에 이름을 저장하고 인사해 봅시다
var myName = "개발자"
print("\(myName)님, Swift 세계에 오신 걸 환영합니다! 🚀")
```

코드를 입력한 후 좌측 하단의 **▶ 실행 버튼**을 누르거나, 단축키 **⇧⌘↩** (Shift + Command + Return)을 누르면 결과가 즉시 우측에 표시됩니다.

> 💡 **알고 계셨나요?**: `print()` 함수의 결과뿐 아니라, 각 줄 오른쪽에 나타나는 작은 회색 텍스트도 확인해 보세요. Playground는 모든 표현식의 값을 자동으로 보여줍니다. 변수에 값을 대입하면 그 값이, 계산을 하면 결과가 바로 표시되죠. 이걸 **인라인 결과(Inline Results)** 라고 합니다.

### 개념 4: 첫 번째 SwiftUI 프로젝트 만들기

Playground에서 Swift 문법을 익히는 것도 좋지만, 실제 앱을 만들어 보는 것만큼 설레는 건 없죠! SwiftUI 프로젝트를 하나 만들어 봅시다.

1. Xcode를 실행합니다
2. **Create New Project** (또는 File → New → Project)를 선택합니다
3. **iOS** 탭에서 **App**을 선택하고 **Next**를 클릭합니다
4. 프로젝트 설정을 입력합니다:

| 항목 | 입력 값 |
|------|---------|
| **Product Name** | MyFirstApp |
| **Organization Identifier** | com.yourname |
| **Interface** | SwiftUI |
| **Language** | Swift |
| **Storage** | None (나중에 SwiftData를 배울 때 변경) |

5. **Next**를 클릭하고 저장 위치를 선택한 후 **Create**를 누릅니다

프로젝트가 생성되면 `ContentView.swift` 파일이 자동으로 열립니다. 이미 기본 코드가 작성되어 있어요!

```swift
import SwiftUI

struct ContentView: View {
    var body: some View {
        // VStack: 세로로 요소를 쌓는 컨테이너
        VStack {
            // 지구본 아이콘 표시
            Image(systemName: "globe")
                .imageScale(.large)
                .foregroundStyle(.tint)

            // 텍스트 표시
            Text("Hello, world!")
        }
        .padding()  // 주변에 여백 추가
    }
}

// Xcode 우측에서 실시간 미리보기를 확인할 수 있습니다
#Preview {
    ContentView()
}
```

지금은 코드가 다 이해되지 않아도 전혀 걱정하지 마세요. 앞으로 하나씩 배울 거예요!

**시뮬레이터에서 실행해 봅시다:**

1. Xcode 상단 중앙에서 시뮬레이터 기기를 선택합니다 (예: iPhone 16 Pro)
2. **▶ Run 버튼**을 클릭하거나 **⌘R** (Command + R)을 누릅니다
3. 시뮬레이터가 실행되면서 "Hello, world!" 앱이 나타납니다

축하합니다! 방금 첫 번째 iOS 앱을 실행했습니다! 🎉

### 개념 5: Xcode 인터페이스 둘러보기

Xcode를 처음 열면 화면이 좀 복잡해 보일 수 있는데요, 크게 4개 영역만 기억하면 됩니다.

| 영역 | 위치 | 역할 | 단축키 (표시/숨기기) |
|------|------|------|---------------------|
| **Navigator** | 왼쪽 | 파일 목록, 검색, 에러 확인 | ⌘0 |
| **Editor** | 중앙 | 코드 작성 공간 | (항상 표시) |
| **Canvas / Preview** | 중앙 오른쪽 | SwiftUI 실시간 미리보기 | ⌥⌘↩ |
| **Inspector** | 오른쪽 | 선택한 요소의 속성 편집 | ⌥⌘0 |

> 🔥 **실무 팁**: 화면이 좁게 느껴지면 **⌘0**으로 Navigator를, **⌥⌘0**으로 Inspector를 숨길 수 있습니다. 코딩에 집중할 때는 Editor와 Preview만 남겨두면 시원하게 쓸 수 있어요.

## 실습: 직접 해보기

"Hello, world!"를 나만의 인사말로 바꿔 봅시다. `ContentView.swift`를 다음과 같이 수정해 보세요.

```swift
import SwiftUI

struct ContentView: View {
    var body: some View {
        VStack(spacing: 16) {
            // 로켓 아이콘으로 변경
            Image(systemName: "rocket.fill")
                .font(.system(size: 60))
                .foregroundStyle(.orange)

            // 나만의 인사말
            Text("Swift 마스터 가이드")
                .font(.title)
                .fontWeight(.bold)

            Text("앱스토어 런칭까지 함께 가봅시다!")
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

Canvas(미리보기)에서 변경 사항이 실시간으로 반영되는 걸 확인해 보세요. 텍스트를 마음대로 바꿔보고, `systemName`에 다른 아이콘 이름(예: `"star.fill"`, `"heart.fill"`)을 넣어 보는 것도 재미있습니다.

## 더 깊이 알아보기

### Swift의 탄생 이야기

Swift는 2010년 7월, Apple의 엔지니어 **Chris Lattner**가 자신의 노트북에서 혼자 시작한 프로젝트였습니다. 당시 코드명은 **"Shiny"** — 말 그대로 "반짝이는 새 것"이라는 뜻이었는데, TV 드라마 Firefly의 레퍼런스이기도 했다고 해요.

놀라운 건, Apple 경영진의 첫 반응이 **"왜 새 언어가 필요하죠? Objective-C가 iPhone을 성공시켰잖아요"** 였다는 겁니다. 하지만 Lattner는 포기하지 않았고, 2011년부터 동료들이 합류하기 시작했습니다. 2013년에는 Apple Developer Tools 팀의 주요 프로젝트가 되었고, 마침내 **2014년 WWDC에서 세상에 공개**되었죠.

Swift는 Objective-C, Rust, Haskell, Ruby, Python, C# 등 수많은 언어에서 좋은 아이디어를 가져와 만들어졌습니다. 그래서 다른 언어를 배워본 경험이 있다면 Swift의 문법이 어딘가 친숙하게 느껴질 수도 있어요.

## 흔한 오해와 팁

> ⚠️ **흔한 오해**: "iOS 앱 개발을 하려면 iPhone이 꼭 있어야 한다" — **시뮬레이터만으로도 대부분의 개발과 테스트가 가능**합니다. 실제 기기가 필요한 건 카메라, 블루투스 등 하드웨어 기능을 테스트할 때뿐이에요. 이 튜토리얼의 대부분은 시뮬레이터로 충분합니다.

> 💡 **알고 계셨나요?**: Xcode 26부터는 **AI 코딩 어시스턴트**가 내장되어 있습니다. 코드를 작성하다가 자연어로 질문하거나, 코드 제안을 받을 수 있어요. 처음에는 직접 타이핑하면서 익히는 걸 추천하지만, 익숙해지면 생산성을 크게 높여줄 도구입니다.

> 🔥 **실무 팁**: Xcode 단축키를 일찍 익혀두면 개발 속도가 확 빨라집니다. 가장 자주 쓰는 3개만 먼저 기억하세요: **⌘R** (실행), **⌘B** (빌드), **⌘.** (실행 중지).

## 핵심 정리

| 개념 | 설명 |
|------|------|
| **Xcode** | Apple 공식 IDE. 코딩, 디버깅, 빌드, 배포를 모두 처리 |
| **Playground** | Swift 코드를 즉시 실행하고 결과를 확인하는 실험 공간 |
| **시뮬레이터** | 실제 기기 없이 iPhone/iPad 앱을 테스트하는 가상 환경 |
| **SwiftUI 프로젝트** | Interface: SwiftUI, Language: Swift로 새 프로젝트 생성 |
| **#Preview** | Canvas에서 SwiftUI 뷰의 실시간 미리보기를 활성화하는 매크로 |
| **⌘R** | 앱 빌드 & 시뮬레이터 실행 단축키 |

## 다음 섹션 미리보기

개발 환경이 준비되었으니, 이제 Swift 언어의 가장 기본적인 빌딩 블록인 **변수와 상수**를 배워봅시다. `var`와 `let`의 차이, Swift가 데이터 타입을 다루는 독특한 방식을 알아볼 거예요.

## 참고 자료

- [Xcode 26 Release Notes](https://developer.apple.com/documentation/xcode-release-notes/xcode-26-release-notes) - Xcode 26 공식 변경 사항과 신기능
- [Apple SwiftUI 튜토리얼](https://developer.apple.com/tutorials/swiftui) - Apple 공식 SwiftUI 입문 튜토리얼
- [Chris Lattner on the origins of Swift](https://oleb.net/2019/chris-lattner-swift-origins/) - Swift 탄생 비화 인터뷰
- [100 Days of SwiftUI - Day 1](https://www.hackingwithswift.com/100/swiftui/1) - Paul Hudson의 SwiftUI 입문 첫째 날
