# 텍스트와 이미지

> Text, Image, 수정자(Modifier) 체이닝

## 개요

앱의 가장 기본적인 재료는 **글자**와 **그림**이죠. 이 섹션에서는 SwiftUI의 `Text`와 `Image` 뷰를 자유자재로 다루는 법을 배웁니다. 수정자 체이닝을 통해 폰트, 색상, 크기, 모양을 마음대로 조절하는 방법까지 익혀볼 거예요.

**선수 지식**: [01. Hello, SwiftUI!](./01-hello-swiftui.md)에서 배운 View, body, #Preview, 수정자 기초
**학습 목표**:
- Text 뷰로 다양한 스타일의 텍스트를 표시하기
- Image 뷰로 SF Symbols와 에셋 이미지를 사용하기
- 수정자 체이닝의 원리를 이해하고 자유롭게 활용하기

## 왜 알아야 할까?

어떤 앱을 열어보세요. 화면의 대부분은 **텍스트**와 **이미지**로 이루어져 있습니다. 메시지 앱의 대화 내용, 인스타그램의 사진, 설정 화면의 아이콘과 레이블... 모두 Text와 Image의 조합이에요. 이 두 가지를 제대로 다루는 것이 앱 UI의 시작점입니다.

## 핵심 개념

### 개념 1: Text 뷰 마스터하기

> 💡 **비유**: Text는 **만능 포스트잇**입니다. 글을 적고, 색을 바꾸고, 크기를 조절하고, 굵게 만들 수 있어요. 포스트잇에 데코레이션을 하듯, 수정자로 텍스트를 꾸밉니다.

Text는 SwiftUI에서 가장 기본적인 뷰입니다. 문자열을 화면에 표시하고, 다양한 수정자로 스타일을 입힐 수 있어요.

```swift
import SwiftUI

struct TextExamplesView: View {
    var body: some View {
        VStack(spacing: 20) {
            // 기본 텍스트
            Text("안녕하세요!")

            // 시스템 폰트 스타일 (Apple이 미리 정의한 크기와 굵기)
            Text("큰 제목")
                .font(.largeTitle)

            Text("본문 텍스트")
                .font(.body)

            Text("캡션")
                .font(.caption)

            // 직접 스타일 지정
            Text("커스텀 스타일")
                .font(.system(size: 24, weight: .bold, design: .rounded))
                .foregroundStyle(.purple)

            // 여러 줄 텍스트
            Text("SwiftUI는 선언형 UI 프레임워크입니다. 코드로 화면을 설명하면 시스템이 알아서 그려줍니다.")
                .font(.body)
                .multilineTextAlignment(.center)  // 여러 줄일 때 가운데 정렬
                .lineLimit(3)                     // 최대 3줄까지만 표시
                .lineSpacing(4)                   // 줄 간격

            // 텍스트 꾸미기
            Text("밑줄")
                .underline()
            Text("취소선")
                .strikethrough(color: .red)
            Text("기울임")
                .italic()
        }
        .padding()
    }
}

#Preview {
    TextExamplesView()
}
```

SwiftUI가 제공하는 **시스템 폰트 스타일**은 Apple의 디자인 가이드라인에 맞춰져 있어서, 직접 크기를 지정하는 것보다 시스템 스타일을 쓰는 것이 좋습니다. Dynamic Type(사용자가 설정에서 글자 크기를 바꾸는 기능)도 자동으로 지원되거든요.

| 스타일 | 용도 | 예시 |
|--------|------|------|
| `.largeTitle` | 화면의 큰 제목 | 앱 메인 타이틀 |
| `.title` / `.title2` / `.title3` | 섹션 제목 | 카테고리 헤더 |
| `.headline` | 강조 텍스트 | 셀 제목 |
| `.body` | 본문 | 설명 텍스트 |
| `.callout` | 보조 설명 | 부가 정보 |
| `.subheadline` | 작은 부제목 | 날짜, 시간 |
| `.footnote` | 각주 | 법적 고지 |
| `.caption` / `.caption2` | 최소 크기 텍스트 | 이미지 캡션 |

### 개념 2: 텍스트 합치기

> 💡 **비유**: 텍스트 합치기는 **컬러 펜 세트**로 한 문장을 쓰는 것과 같습니다. 빨간 펜으로 일부를 쓰고, 파란 펜으로 나머지를 쓰면 한 문장에 여러 색이 들어가죠.

SwiftUI에서는 `+` 연산자로 서로 다른 스타일의 텍스트를 하나로 합칠 수 있습니다.

```swift
struct CombinedTextView: View {
    var body: some View {
        VStack(spacing: 20) {
            // + 연산자로 텍스트 합치기
            Text("Hello, ")
                .font(.title)
                .foregroundStyle(.blue)
            + Text("SwiftUI!")
                .font(.title)
                .fontWeight(.bold)
                .foregroundStyle(.orange)

            // AttributedString으로 더 세밀한 스타일링
            Text(makeAttributedGreeting())
        }
        .padding()
    }

    // AttributedString을 만드는 함수
    func makeAttributedGreeting() -> AttributedString {
        var greeting = AttributedString("Swift를 ")
        greeting.font = .body

        var highlight = AttributedString("마스터")
        highlight.font = .title2
        highlight.foregroundColor = .red

        var rest = AttributedString("하는 중!")
        rest.font = .body

        return greeting + highlight + rest
    }
}

#Preview {
    CombinedTextView()
}
```

### 개념 3: Image 뷰 — SF Symbols 활용하기

> 💡 **비유**: SF Symbols는 Apple이 제공하는 **무료 아이콘 백화점**입니다. 6,000개 이상의 아이콘이 준비되어 있어서, 검색만 하면 원하는 아이콘을 바로 가져다 쓸 수 있어요.

SwiftUI에서 이미지를 표시하는 방법은 크게 세 가지입니다:
1. **SF Symbols** — Apple이 제공하는 시스템 아이콘
2. **에셋 이미지** — 프로젝트에 직접 추가한 이미지
3. **AsyncImage** — 인터넷에서 다운로드한 이미지

먼저 SF Symbols부터 알아볼게요.

```swift
struct SFSymbolsView: View {
    var body: some View {
        VStack(spacing: 20) {
            // 기본 SF Symbol
            Image(systemName: "star.fill")

            // 크기 조절 — font 수정자 사용
            Image(systemName: "heart.fill")
                .font(.system(size: 40))
                .foregroundStyle(.red)

            // 여러 색상 (멀티컬러 심볼)
            Image(systemName: "cloud.sun.rain.fill")
                .font(.system(size: 50))
                .symbolRenderingMode(.multicolor)

            // 그래디언트 색상
            Image(systemName: "flame.fill")
                .font(.system(size: 50))
                .foregroundStyle(
                    .linearGradient(
                        colors: [.red, .orange],
                        startPoint: .bottom,
                        endPoint: .top
                    )
                )

            // SF Symbol + 텍스트 조합 (Label)
            Label("즐겨찾기", systemImage: "star.fill")
                .font(.title2)
                .foregroundStyle(.yellow)
        }
        .padding()
    }
}

#Preview {
    SFSymbolsView()
}
```

> 🔥 **실무 팁**: SF Symbols 앱을 Mac에 설치하면 6,000개 이상의 아이콘을 검색하고 미리볼 수 있습니다. App Store에서 무료로 다운로드하세요. 아이콘 이름을 복사해서 `systemName:`에 바로 붙여넣기 할 수 있어요.

### 개념 4: 에셋 이미지와 리사이징

> 💡 **비유**: 에셋 이미지를 넣는 건 **사진을 액자에 끼우는 것**과 같습니다. 사진이 액자보다 크면 잘라야 하고, 작으면 늘려야 하죠. `.resizable()`과 `.aspectRatio()`가 이 역할을 합니다.

프로젝트의 Asset Catalog에 추가한 이미지를 사용하는 방법입니다.

```swift
struct AssetImageView: View {
    var body: some View {
        VStack(spacing: 20) {
            // 에셋 카탈로그의 이미지 (이미지 이름으로 참조)
            // Image("my-photo")

            // resizable()로 크기 조절 가능하게 만들기
            // (resizable 없이는 원본 크기로만 표시됩니다!)
            Image(systemName: "photo.artframe")
                .resizable()                    // 크기 조절 가능
                .aspectRatio(contentMode: .fit)  // 비율 유지하며 맞추기
                .frame(width: 200, height: 200)  // 프레임 크기 지정

            // 원형으로 자르기
            Image(systemName: "person.fill")
                .resizable()
                .aspectRatio(contentMode: .fill)  // 비율 유지하며 채우기
                .frame(width: 100, height: 100)
                .clipShape(Circle())              // 원형으로 자르기
                .overlay(                         // 테두리 추가
                    Circle()
                        .stroke(.blue, lineWidth: 3)
                )

            // 둥근 모서리 사각형
            Image(systemName: "photo")
                .resizable()
                .aspectRatio(contentMode: .fill)
                .frame(width: 150, height: 100)
                .clipShape(RoundedRectangle(cornerRadius: 16))
                .shadow(radius: 5)                // 그림자 효과
        }
        .padding()
    }
}

#Preview {
    AssetImageView()
}
```

`.fit`과 `.fill`의 차이가 중요합니다:

| contentMode | 동작 | 결과 |
|-------------|------|------|
| `.fit` | 이미지 전체가 보이도록 축소 | 여백이 생길 수 있음 |
| `.fill` | 프레임을 꽉 채우도록 확대 | 이미지가 잘릴 수 있음 |

> ⚠️ **흔한 오해**: "Image를 만들면 바로 크기를 바꿀 수 있다" — `.resizable()` 수정자를 먼저 적용하지 않으면 `.frame()`을 써도 이미지 크기가 바뀌지 않습니다. **반드시 `.resizable()`을 먼저** 호출하세요!

### 개념 5: AsyncImage — 인터넷 이미지 로드하기

> 💡 **비유**: AsyncImage는 **택배 추적**과 같습니다. 주문(URL)하면 배송 중에는 로딩 표시가 나오고, 도착하면 상품(이미지)이 보이고, 배송 실패하면 안내 메시지가 나오죠.

인터넷에서 이미지를 다운로드해서 표시하는 것도 SwiftUI에서는 간단합니다.

```swift
struct AsyncImageView: View {
    var body: some View {
        VStack(spacing: 20) {
            // 기본 AsyncImage
            AsyncImage(url: URL(string: "https://picsum.photos/200")) { image in
                // 이미지 로드 성공
                image
                    .resizable()
                    .aspectRatio(contentMode: .fill)
            } placeholder: {
                // 로딩 중
                ProgressView()
            }
            .frame(width: 200, height: 200)
            .clipShape(RoundedRectangle(cornerRadius: 12))

            // phase를 활용한 상세 처리
            AsyncImage(url: URL(string: "https://picsum.photos/300")) { phase in
                switch phase {
                case .empty:
                    // 아직 로딩 중
                    ProgressView()
                        .frame(width: 150, height: 150)
                case .success(let image):
                    // 로드 성공
                    image
                        .resizable()
                        .aspectRatio(contentMode: .fit)
                        .frame(width: 150, height: 150)
                        .clipShape(RoundedRectangle(cornerRadius: 8))
                case .failure:
                    // 로드 실패
                    Image(systemName: "photo.badge.exclamationmark")
                        .font(.system(size: 40))
                        .foregroundStyle(.gray)
                        .frame(width: 150, height: 150)
                @unknown default:
                    EmptyView()
                }
            }
        }
        .padding()
    }
}

#Preview {
    AsyncImageView()
}
```

## 실습: 프로필 카드 만들기

지금까지 배운 Text와 Image를 조합해서 간단한 프로필 카드를 만들어봅시다.

```swift
import SwiftUI

struct ProfileCardView: View {
    // 프로필 정보
    let name = "김스위프트"
    let role = "iOS 개발자"
    let bio = "SwiftUI를 사랑하는 개발자입니다. 매일 새로운 것을 배우고 있어요!"

    var body: some View {
        VStack(spacing: 16) {
            // 프로필 이미지 (SF Symbol로 대체)
            Image(systemName: "person.crop.circle.fill")
                .resizable()
                .aspectRatio(contentMode: .fit)
                .frame(width: 100, height: 100)
                .foregroundStyle(.blue)

            // 이름
            Text(name)
                .font(.title2)
                .fontWeight(.bold)

            // 역할 배지
            Text(role)
                .font(.subheadline)
                .foregroundStyle(.white)
                .padding(.horizontal, 12)
                .padding(.vertical, 4)
                .background(.blue)
                .clipShape(Capsule())

            // 자기소개
            Text(bio)
                .font(.body)
                .foregroundStyle(.secondary)
                .multilineTextAlignment(.center)
                .padding(.horizontal)

            // 통계 정보
            HStack(spacing: 30) {
                StatItem(value: "128", label: "게시물")
                StatItem(value: "1.2K", label: "팔로워")
                StatItem(value: "456", label: "팔로잉")
            }
        }
        .padding(24)
        .background(.ultraThinMaterial)   // 반투명 머티리얼 배경
        .clipShape(RoundedRectangle(cornerRadius: 20))
        .shadow(radius: 10)
        .padding()
    }
}

// 통계 항목을 위한 작은 뷰
struct StatItem: View {
    let value: String
    let label: String

    var body: some View {
        VStack(spacing: 4) {
            Text(value)
                .font(.title3)
                .fontWeight(.bold)
            Text(label)
                .font(.caption)
                .foregroundStyle(.secondary)
        }
    }
}

#Preview {
    ProfileCardView()
}
```

이 코드에서 주목할 점이 있어요. `StatItem`이라는 **별도의 뷰**를 만들었다는 것! SwiftUI에서는 이렇게 작은 뷰를 분리해서 재사용하는 것이 좋은 습관입니다. 마치 레고 블록을 만들어 놓고 조립하는 것처럼요.

## 더 깊이 알아보기

### SF Symbols의 탄생

SF Symbols는 WWDC 2019에서 SwiftUI와 함께 공개되었습니다. Apple의 시스템 폰트인 San Francisco와 완벽하게 어울리도록 설계된 아이콘 세트인데요, 처음 1,500개로 시작해서 지금은 **6,000개 이상**으로 늘어났습니다.

SF Symbols의 특별한 점은 **텍스트처럼 동작**한다는 것입니다. `.font()` 수정자로 크기를 조절할 수 있고, Dynamic Type에 따라 자동으로 크기가 변하며, 텍스트와 정렬이 자연스럽게 맞춰집니다. 이것이 Apple이 아이콘을 단순 이미지가 아닌 "심볼"이라고 부르는 이유예요.

### foregroundColor에서 foregroundStyle로

iOS 17부터 `.foregroundColor()`가 `.foregroundStyle()`로 대체되었습니다. 단순히 이름만 바뀐 게 아니에요. `foregroundStyle`은 단색뿐 아니라 **그래디언트, 머티리얼, 계층적 스타일**까지 지원하거든요. 기존 `foregroundColor`도 아직 동작하지만, 새 코드에서는 `foregroundStyle`을 사용하는 것이 좋습니다.

## 흔한 오해와 팁

> ⚠️ **흔한 오해**: "수정자를 많이 붙이면 성능이 나빠진다" — SwiftUI의 수정자는 내부적으로 뷰 트리를 효율적으로 구성하므로, 수정자를 적절히 사용하는 것은 성능에 거의 영향을 미치지 않습니다. 가독성 좋게 마음껏 체이닝하세요!

> 🔥 **실무 팁**: 색상에 `.primary`와 `.secondary`를 쓰면 다크 모드에서 자동으로 적절한 색상으로 바뀝니다. 하드코딩된 `.black`이나 `.white` 대신 시맨틱 색상을 사용하세요.

> 💡 **알고 계셨나요?**: `Label("텍스트", systemImage: "icon.name")`은 아이콘과 텍스트를 함께 표시하는 편의 뷰입니다. 컨텍스트에 따라 자동으로 아이콘만, 텍스트만, 또는 둘 다 표시할 수 있어서 접근성에도 좋습니다.

## 핵심 정리

| 개념 | 설명 |
|------|------|
| Text | 문자열을 화면에 표시하는 기본 뷰 |
| .font() | 시스템 폰트 스타일(`.title`, `.body` 등) 또는 커스텀 크기 지정 |
| .foregroundStyle() | 텍스트/이미지 색상 지정 (단색, 그래디언트, 머티리얼 지원) |
| Image(systemName:) | SF Symbols 아이콘을 표시 (6,000개 이상) |
| .resizable() | 이미지를 크기 조절 가능하게 만듦 (필수 선행 수정자) |
| .aspectRatio() | `.fit` (전체 표시) 또는 `.fill` (꽉 채우기) |
| .clipShape() | 뷰를 원형, 캡슐, 둥근 사각형 등으로 자르기 |
| AsyncImage | URL에서 이미지를 비동기 로드 (로딩/성공/실패 처리) |
| Label | 아이콘 + 텍스트를 함께 표시하는 편의 뷰 |

## 다음 섹션 미리보기

텍스트와 이미지로 정보를 **보여주는** 법을 배웠으니, 다음으로 [03. 버튼과 인터랙션](./03-button-interaction.md)에서 사용자가 **터치**했을 때 반응하는 버튼을 만들어봅니다. 앱이 진짜 "살아있는" 느낌이 드는 순간이에요!

## 참고 자료

- [Apple Text 공식 문서](https://developer.apple.com/documentation/swiftui/text) - Text 뷰의 모든 수정자 레퍼런스
- [Apple Image 공식 문서](https://developer.apple.com/documentation/swiftui/image) - Image 뷰 사용법
- [SF Symbols](https://developer.apple.com/sf-symbols/) - SF Symbols 앱 다운로드 및 가이드
- [WWDC 2023 - What's new in SF Symbols 5](https://developer.apple.com/videos/play/wwdc2023/10197/) - SF Symbols 최신 기능
- [Apple Human Interface Guidelines - Typography](https://developer.apple.com/design/human-interface-guidelines/typography) - Apple 타이포그래피 디자인 가이드
