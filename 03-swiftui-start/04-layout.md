# 레이아웃 시스템

> VStack, HStack, ZStack, Spacer, padding, frame

## 개요

지금까지 뷰들을 VStack 안에 세로로만 쌓았는데, 실제 앱 화면은 훨씬 복잡하죠. 이 섹션에서는 SwiftUI의 레이아웃 시스템을 배워서 뷰를 자유롭게 배치하는 법을 익힙니다. 세로, 가로, 겹치기, 간격 조절까지 — 앱다운 화면 구성의 핵심이에요.

**선수 지식**: [03. 버튼과 인터랙션](./03-button-interaction.md)에서 배운 Button, @State, 수정자
**학습 목표**:
- VStack, HStack, ZStack의 역할과 차이를 이해하기
- Spacer와 padding으로 간격과 여백을 조절하기
- frame, alignment로 뷰의 크기와 정렬을 제어하기
- 실전 레이아웃 패턴을 연습하기

## 왜 알아야 할까?

디자이너가 "이미지는 왼쪽에, 텍스트는 오른쪽에, 버튼은 아래에 배치해주세요"라고 하면 어떻게 할까요? SwiftUI의 레이아웃 시스템만 이해하면 **어떤 화면이든** 코드로 만들 수 있습니다. Stack 3형제(VStack, HStack, ZStack)가 레이아웃의 90%를 해결해준다고 해도 과언이 아니에요.

## 핵심 개념

### 개념 1: Stack 3형제 — VStack, HStack, ZStack

> 💡 **비유**: Stack은 **블록 쌓기**입니다. VStack은 블록을 **세로로** 쌓고, HStack은 **가로로** 나란히 놓고, ZStack은 블록을 **겹겹이** 쌓습니다. 이 세 가지 조합만으로 웬만한 레이아웃을 다 만들 수 있어요.

```swift
import SwiftUI

struct StackDemoView: View {
    var body: some View {
        VStack(spacing: 30) {
            // VStack: 세로 배치
            VStack(spacing: 8) {
                Text("VStack")
                    .font(.headline)
                Text("위에서 아래로")
                Text("세로로 쌓기")
            }
            .padding()
            .background(.blue.opacity(0.1))
            .clipShape(RoundedRectangle(cornerRadius: 8))

            // HStack: 가로 배치
            HStack(spacing: 8) {
                Text("H")
                Text("S")
                Text("t")
                Text("a")
                Text("c")
                Text("k")
            }
            .font(.title)
            .padding()
            .background(.green.opacity(0.1))
            .clipShape(RoundedRectangle(cornerRadius: 8))

            // ZStack: 겹치기
            ZStack {
                // 가장 먼저 쓴 것이 맨 뒤
                RoundedRectangle(cornerRadius: 12)
                    .fill(.purple.opacity(0.3))
                    .frame(width: 200, height: 100)

                // 그 다음이 위에 쌓임
                Text("ZStack: 겹치기")
                    .font(.headline)
                    .foregroundStyle(.white)
            }
        }
        .padding()
    }
}

#Preview {
    StackDemoView()
}
```

| Stack | 방향 | 비유 | 사용 예시 |
|-------|------|------|----------|
| VStack | 세로 (↕️) | 책 쌓기 | 폼, 목록, 프로필 |
| HStack | 가로 (↔️) | 벽돌 나열 | 탭 바, 통계 행 |
| ZStack | 겹치기 (⬆️) | 카드 겹치기 | 배지, 오버레이, 배경 |

### 개념 2: alignment — 정렬하기

> 💡 **비유**: alignment는 **줄 맞추기**입니다. 학교에서 줄을 설 때 왼쪽으로 맞출지, 가운데로 맞출지, 오른쪽으로 맞출지 정하는 것과 같아요.

Stack에 alignment 파라미터를 전달해서 자식 뷰들의 정렬을 제어할 수 있습니다.

```swift
struct AlignmentDemoView: View {
    var body: some View {
        VStack(spacing: 20) {
            // VStack — 가로 정렬 (leading, center, trailing)
            VStack(alignment: .leading, spacing: 4) {
                Text("왼쪽 정렬")
                    .font(.headline)
                Text("모든 텍스트가")
                Text("왼쪽으로 붙습니다")
                    .font(.caption)
            }
            .frame(maxWidth: .infinity, alignment: .leading)
            .padding()
            .background(.blue.opacity(0.1))
            .clipShape(RoundedRectangle(cornerRadius: 8))

            // HStack — 세로 정렬 (top, center, bottom)
            HStack(alignment: .top, spacing: 12) {
                Text("큰\n텍스트")
                    .font(.title)
                    .padding()
                    .background(.green.opacity(0.2))

                Text("작은 텍스트")
                    .font(.caption)
                    .padding()
                    .background(.orange.opacity(0.2))
            }

            // ZStack — 동시 정렬
            ZStack(alignment: .bottomTrailing) {
                // 배경 카드
                RoundedRectangle(cornerRadius: 12)
                    .fill(.gray.opacity(0.2))
                    .frame(width: 200, height: 120)

                // 오른쪽 아래 배지
                Text("NEW")
                    .font(.caption2)
                    .fontWeight(.bold)
                    .foregroundStyle(.white)
                    .padding(.horizontal, 8)
                    .padding(.vertical, 4)
                    .background(.red)
                    .clipShape(Capsule())
                    .padding(8)
            }
        }
        .padding()
    }
}

#Preview {
    AlignmentDemoView()
}
```

### 개념 3: Spacer — 빈 공간의 마법

> 💡 **비유**: Spacer는 **스프링**입니다. 뷰 사이에 스프링을 넣으면 가능한 한 많이 늘어나면서 뷰들을 밀어내죠. 두 개를 넣으면 힘이 균등하게 분배되어 가운데 정렬 효과를 만들 수 있습니다.

Spacer는 **사용 가능한 공간을 최대한 차지**하는 투명한 뷰입니다.

```swift
struct SpacerDemoView: View {
    var body: some View {
        VStack(spacing: 20) {
            // Spacer로 양 끝에 배치
            HStack {
                Text("왼쪽")
                Spacer()        // 사이 공간을 최대한 차지
                Text("오른쪽")
            }
            .padding()
            .background(.gray.opacity(0.1))
            .clipShape(RoundedRectangle(cornerRadius: 8))

            // Spacer 2개로 가운데 배치
            HStack {
                Spacer()
                Text("가운데")
                Spacer()
            }
            .padding()
            .background(.gray.opacity(0.1))
            .clipShape(RoundedRectangle(cornerRadius: 8))

            // Spacer로 아래로 밀기
            VStack {
                Text("맨 위")
                    .font(.headline)
                Spacer()            // 위아래 사이를 최대한 벌림
                Text("맨 아래")
                    .font(.headline)
            }
            .frame(height: 150)
            .frame(maxWidth: .infinity)
            .padding()
            .background(.blue.opacity(0.1))
            .clipShape(RoundedRectangle(cornerRadius: 8))

            // 최소 간격 지정
            HStack {
                Text("A")
                Spacer(minLength: 50)  // 최소 50포인트 보장
                Text("B")
            }
            .padding()
            .background(.gray.opacity(0.1))
            .clipShape(RoundedRectangle(cornerRadius: 8))
        }
        .padding()
    }
}

#Preview {
    SpacerDemoView()
}
```

### 개념 4: padding과 frame

> 💡 **비유**: padding은 **택배 포장의 완충재**입니다. 상품(뷰) 주변에 뽁뽁이를 감싸면 여유 공간이 생기듯, padding은 뷰 주변에 여백을 만들어줍니다. frame은 **상자 크기**를 직접 지정하는 것이에요.

```swift
struct PaddingFrameView: View {
    var body: some View {
        VStack(spacing: 20) {
            // padding — 여백 추가
            Text("기본 padding")
                .background(.yellow)
                .padding()               // 상하좌우 16pt
                .background(.orange)

            // 방향별 padding
            Text("방향별 padding")
                .background(.yellow)
                .padding(.horizontal, 30)  // 좌우 30pt
                .padding(.vertical, 8)     // 상하 8pt
                .background(.orange)

            // frame — 고정 크기 지정
            Text("고정 크기")
                .frame(width: 200, height: 50)
                .background(.green.opacity(0.3))

            // frame — 최대/최소 크기
            Text("전체 너비")
                .frame(maxWidth: .infinity)  // 가로 최대
                .padding()
                .background(.blue.opacity(0.2))

            // frame + alignment
            Text("오른쪽 정렬")
                .frame(maxWidth: .infinity, alignment: .trailing)
                .padding()
                .background(.purple.opacity(0.2))
        }
        .padding()
    }
}

#Preview {
    PaddingFrameView()
}
```

> ⚠️ **흔한 오해**: "padding과 frame은 같은 것이다" — 전혀 다릅니다! `padding`은 뷰 **주변에** 여백을 추가하는 것이고, `frame`은 뷰 **자체의 크기**를 지정하는 것입니다. padding은 내부 콘텐츠를 건드리지 않지만, frame은 뷰가 차지하는 영역 자체를 변경합니다.

### 개념 5: 실전 레이아웃 패턴

실제 앱에서 자주 쓰이는 레이아웃 패턴들을 살펴볼게요.

```swift
struct LayoutPatternsView: View {
    var body: some View {
        VStack(spacing: 20) {
            // 패턴 1: 리스트 셀 (아이콘 + 텍스트 + 화살표)
            HStack(spacing: 12) {
                Image(systemName: "person.circle.fill")
                    .font(.title2)
                    .foregroundStyle(.blue)

                VStack(alignment: .leading, spacing: 2) {
                    Text("김스위프트")
                        .font(.headline)
                    Text("iOS 개발자")
                        .font(.caption)
                        .foregroundStyle(.secondary)
                }

                Spacer()

                Image(systemName: "chevron.right")
                    .font(.caption)
                    .foregroundStyle(.gray)
            }
            .padding()
            .background(.gray.opacity(0.1))
            .clipShape(RoundedRectangle(cornerRadius: 12))

            // 패턴 2: 카드 (이미지 위에 텍스트 오버레이)
            ZStack(alignment: .bottomLeading) {
                // 배경
                RoundedRectangle(cornerRadius: 16)
                    .fill(
                        LinearGradient(
                            colors: [.blue, .purple],
                            startPoint: .topLeading,
                            endPoint: .bottomTrailing
                        )
                    )
                    .frame(height: 180)

                // 오버레이 텍스트
                VStack(alignment: .leading, spacing: 4) {
                    Text("SwiftUI 마스터")
                        .font(.title2)
                        .fontWeight(.bold)
                    Text("선언형 UI의 세계로!")
                        .font(.subheadline)
                }
                .foregroundStyle(.white)
                .padding()
            }

            // 패턴 3: 균등 분할 (HStack + frame)
            HStack(spacing: 0) {
                ForEach(["월", "화", "수", "목", "금"], id: \.self) { day in
                    Text(day)
                        .font(.headline)
                        .frame(maxWidth: .infinity)  // 균등 분할의 핵심!
                        .padding(.vertical, 12)
                        .background(day == "수" ? .blue : .clear)
                        .foregroundStyle(day == "수" ? .white : .primary)
                }
            }
            .clipShape(RoundedRectangle(cornerRadius: 12))
            .background(
                RoundedRectangle(cornerRadius: 12)
                    .fill(.gray.opacity(0.1))
            )
        }
        .padding()
    }
}

#Preview {
    LayoutPatternsView()
}
```

## 실습: 날씨 앱 화면 만들기

Stack 조합으로 날씨 앱의 메인 화면을 만들어봅시다.

```swift
import SwiftUI

struct WeatherView: View {
    var body: some View {
        ZStack {
            // 배경 그래디언트
            LinearGradient(
                colors: [.blue, .cyan, .blue.opacity(0.6)],
                startPoint: .top,
                endPoint: .bottom
            )
            .ignoresSafeArea()  // 화면 전체를 채움

            VStack(spacing: 20) {
                // 도시 이름
                Text("서울")
                    .font(.largeTitle)
                    .fontWeight(.medium)
                    .foregroundStyle(.white)

                // 현재 날씨
                VStack(spacing: 4) {
                    Image(systemName: "sun.max.fill")
                        .font(.system(size: 64))
                        .symbolRenderingMode(.multicolor)

                    Text("23°")
                        .font(.system(size: 72, weight: .thin))
                        .foregroundStyle(.white)

                    Text("맑음")
                        .font(.title3)
                        .foregroundStyle(.white.opacity(0.8))
                }

                // 최고/최저 온도
                HStack(spacing: 20) {
                    Label("최고: 26°", systemImage: "thermometer.high")
                    Label("최저: 18°", systemImage: "thermometer.low")
                }
                .font(.subheadline)
                .foregroundStyle(.white.opacity(0.8))

                Spacer()

                // 시간별 예보
                VStack(alignment: .leading, spacing: 12) {
                    Text("시간별 예보")
                        .font(.headline)
                        .foregroundStyle(.white)

                    HStack(spacing: 0) {
                        ForEach(hourlyForecast, id: \.time) { forecast in
                            VStack(spacing: 8) {
                                Text(forecast.time)
                                    .font(.caption)
                                Image(systemName: forecast.icon)
                                    .font(.title3)
                                    .symbolRenderingMode(.multicolor)
                                Text("\(forecast.temp)°")
                                    .font(.callout)
                                    .fontWeight(.semibold)
                            }
                            .frame(maxWidth: .infinity)
                        }
                    }
                    .foregroundStyle(.white)
                }
                .padding()
                .background(.ultraThinMaterial)
                .clipShape(RoundedRectangle(cornerRadius: 16))

                Spacer()
            }
            .padding()
        }
    }

    // 시간별 예보 데이터
    var hourlyForecast: [(time: String, icon: String, temp: Int)] {
        [
            ("지금", "sun.max.fill", 23),
            ("1시", "cloud.sun.fill", 22),
            ("2시", "cloud.fill", 21),
            ("3시", "cloud.sun.fill", 22),
            ("4시", "sun.max.fill", 24)
        ]
    }
}

#Preview {
    WeatherView()
}
```

이 코드에서 배울 수 있는 핵심 패턴들:
- **ZStack + LinearGradient**: 배경 그래디언트 위에 콘텐츠를 겹치기
- **.ignoresSafeArea()**: 상단 노치/하단 홈 영역까지 배경을 확장
- **Spacer()**: VStack 안에서 콘텐츠를 위아래로 밀어 간격 조절
- **.ultraThinMaterial**: iOS의 반투명 블러 효과 (Liquid Glass의 기반)
- **ForEach + frame(maxWidth: .infinity)**: 균등 분할

## 더 깊이 알아보기

### SwiftUI 레이아웃의 3단계 규칙

SwiftUI의 레이아웃은 놀랍도록 논리적인 **3단계 협상** 과정을 거칩니다:

1. **부모가 자식에게 제안**: "이만큼의 공간이 있어" (proposed size)
2. **자식이 스스로 결정**: "나는 이만큼 필요해" (자식 뷰가 자기 크기를 결정)
3. **부모가 배치**: 자식이 결정한 크기를 기반으로 위치를 정함

이 과정이 왜 중요한지 예를 들어볼게요. `Text("Hello")`에게 200x200 공간을 제안하면, Text는 "나는 텍스트 크기만큼만 필요해"라고 답합니다. 반면 `Color.blue`에게 같은 제안을 하면 "200x200 전부 쓸게!"라고 답하죠. 뷰마다 **공간을 사용하는 방식이 다릅니다**.

### LazyVStack과 LazyHStack

일반 VStack/HStack은 모든 자식 뷰를 **한 번에** 생성합니다. 아이템이 10개면 괜찮지만, 1,000개라면? 그래서 **Lazy** 버전이 있어요. LazyVStack/LazyHStack은 화면에 **보이는 뷰만** 생성하고, 스크롤해서 사라지면 메모리에서 해제합니다. 성능 최적화의 핵심이죠. 이것은 [다음 섹션](./05-lists-scroll.md)에서 더 자세히 다룹니다.

## 흔한 오해와 팁

> ⚠️ **흔한 오해**: "GeometryReader를 써야 반응형 레이아웃을 만들 수 있다" — 대부분의 경우 `frame(maxWidth: .infinity)`, Spacer, 그리고 Stack 조합만으로 충분합니다. GeometryReader는 정말 필요한 경우에만 사용하세요. 남용하면 레이아웃이 오히려 복잡해집니다.

> 🔥 **실무 팁**: 복잡한 레이아웃이 잘 안 될 때는 **각 뷰에 `.border(.red)` 수정자**를 임시로 붙여보세요. 뷰의 실제 영역이 빨간 테두리로 보여서 디버깅이 훨씬 쉬워집니다.

> 💡 **알고 계셨나요?**: iOS 16에서 추가된 `Grid`는 행과 열을 정밀하게 제어할 수 있는 레이아웃 컨테이너입니다. HTML의 `<table>`과 비슷한데, Stack 조합으로 복잡한 테이블을 만들 때보다 훨씬 깔끔해요.

## 핵심 정리

| 개념 | 설명 |
|------|------|
| VStack | 자식 뷰를 세로로 배치. alignment로 가로 정렬 제어 |
| HStack | 자식 뷰를 가로로 배치. alignment로 세로 정렬 제어 |
| ZStack | 자식 뷰를 겹쳐서 배치. 먼저 선언한 뷰가 뒤에 깔림 |
| Spacer | 사용 가능한 공간을 최대한 차지하는 투명 뷰 |
| padding | 뷰 주변에 여백을 추가. 방향과 크기 지정 가능 |
| frame | 뷰의 크기를 고정 또는 최대/최소로 지정 |
| alignment | Stack과 frame에서 자식 뷰의 정렬 방향 지정 |
| .ignoresSafeArea() | Safe Area를 무시하고 화면 전체로 확장 |
| .infinity | `frame(maxWidth: .infinity)`로 가능한 최대 크기 사용 |

## 다음 섹션 미리보기

레이아웃의 기초를 배웠으니, 다음으로 [05. 리스트와 스크롤](./05-lists-scroll.md)에서 **많은 데이터를 스크롤 가능한 목록**으로 표시하는 법을 배웁니다. List, ForEach, ScrollView, LazyVStack — 앱에서 가장 자주 쓰이는 패턴들이에요!

## 참고 자료

- [Apple Layout 공식 문서](https://developer.apple.com/documentation/swiftui/layout-fundamentals) - SwiftUI 레이아웃 기초
- [WWDC 2022 - Compose custom layouts with SwiftUI](https://developer.apple.com/videos/play/wwdc2022/10056/) - Grid와 커스텀 레이아웃
- [WWDC 2019 - Building Custom Views with SwiftUI](https://developer.apple.com/videos/play/wwdc2019/237/) - 레이아웃 3단계 규칙 상세 설명
- [Apple HStack 공식 문서](https://developer.apple.com/documentation/swiftui/hstack) - HStack API 레퍼런스
- [Apple VStack 공식 문서](https://developer.apple.com/documentation/swiftui/vstack) - VStack API 레퍼런스
