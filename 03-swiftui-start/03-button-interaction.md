# 버튼과 인터랙션

> Button, Label, 액션 처리, buttonStyle

## 개요

지금까지는 화면에 정보를 **보여주기만** 했죠. 이제 사용자가 **터치했을 때 반응하는** 앱을 만들 차례입니다! 이 섹션에서는 Button을 중심으로 사용자 인터랙션을 처리하는 방법을 배웁니다.

**선수 지식**: [02. 텍스트와 이미지](./02-text-image.md)에서 배운 Text, Image, 수정자 체이닝
**학습 목표**:
- Button을 생성하고 액션을 연결하기
- 다양한 버튼 스타일과 커스터마이징 방법 익히기
- @State로 간단한 상태 변경 체험하기
- Toggle, Slider, Stepper 같은 입력 컨트롤 사용하기

## 왜 알아야 할까?

버튼 없는 앱을 상상할 수 있나요? "좋아요" 누르기, "전송" 하기, "설정 변경"하기 — 사용자가 앱과 **소통**하는 거의 모든 순간에 버튼이 있습니다. SwiftUI에서 버튼을 만드는 건 놀라울 정도로 간단한데, 그 안에 숨어있는 개념들이 꽤 재미있거든요.

## 핵심 개념

### 개념 1: Button의 기본 구조

> 💡 **비유**: Button은 **엘리베이터 버튼**과 같습니다. 버튼의 모양(Label)이 있고, 누르면 실행되는 동작(Action)이 있죠. SwiftUI의 Button도 정확히 이 두 가지로 구성됩니다.

Button은 **액션(Action)** 과 **레이블(Label)** 두 부분으로 이루어집니다.

```swift
import SwiftUI

struct ButtonBasicsView: View {
    var body: some View {
        VStack(spacing: 20) {
            // 가장 간단한 버튼 — 텍스트 레이블
            Button("눌러보세요") {
                print("버튼이 눌렸습니다!")
            }

            // 액션과 레이블을 분리한 형태
            Button {
                print("커스텀 버튼 탭!")
            } label: {
                // 레이블에는 어떤 뷰든 넣을 수 있습니다
                HStack {
                    Image(systemName: "hand.tap.fill")
                    Text("탭하세요")
                }
                .font(.title3)
                .foregroundStyle(.white)
                .padding()
                .background(.blue)
                .clipShape(RoundedRectangle(cornerRadius: 12))
            }

            // Label을 활용한 버튼
            Button {
                print("다운로드 시작!")
            } label: {
                Label("다운로드", systemImage: "arrow.down.circle.fill")
            }
        }
        .padding()
    }
}

#Preview {
    ButtonBasicsView()
}
```

### 개념 2: @State — 버튼에 생명 불어넣기

> 💡 **비유**: @State는 **화이트보드**입니다. 내용이 바뀌면 보고 있는 모든 사람(화면)이 변경을 즉시 알아차리죠. 버튼을 누를 때마다 화이트보드의 숫자가 바뀌고, 화면이 자동으로 업데이트됩니다.

`print()`로만 확인하는 건 재미없죠? `@State`를 사용하면 버튼을 눌렀을 때 **화면이 실제로 바뀌는** 경험을 할 수 있습니다. @State는 뷰가 **기억하는 값**이에요. 이 값이 바뀌면 SwiftUI가 자동으로 화면을 다시 그립니다.

```swift
struct CounterView: View {
    // @State: 이 뷰가 소유하는 상태 변수
    @State private var count = 0

    var body: some View {
        VStack(spacing: 24) {
            // 현재 카운트를 표시
            Text("\(count)")
                .font(.system(size: 72, weight: .bold, design: .rounded))
                .foregroundStyle(count >= 0 ? .blue : .red)

            HStack(spacing: 20) {
                // 감소 버튼
                Button {
                    count -= 1   // @State 값 변경 → 화면 자동 업데이트
                } label: {
                    Image(systemName: "minus.circle.fill")
                        .font(.system(size: 44))
                        .foregroundStyle(.red)
                }

                // 리셋 버튼
                Button("리셋") {
                    count = 0
                }
                .buttonStyle(.bordered)

                // 증가 버튼
                Button {
                    count += 1
                } label: {
                    Image(systemName: "plus.circle.fill")
                        .font(.system(size: 44))
                        .foregroundStyle(.blue)
                }
            }
        }
        .padding()
    }
}

#Preview {
    CounterView()
}
```

이 코드에서 `count`가 바뀔 때마다 SwiftUI는 `body`를 다시 계산해서 화면을 업데이트합니다. 여러분이 직접 "화면을 다시 그려!"라고 명령할 필요가 없어요 — 이것이 선언형 UI의 마법이죠.

> ⚠️ **흔한 오해**: "@State 변수는 body가 호출될 때마다 초기화된다" — 아닙니다! @State는 SwiftUI가 뷰 외부에서 별도로 관리하기 때문에 body가 다시 계산되어도 값이 유지됩니다. `@State private var count = 0`에서 `= 0`은 **최초 한 번만** 적용되는 초기값이에요.

### 개념 3: 버튼 스타일

> 💡 **비유**: 버튼 스타일은 **옷**과 같습니다. 같은 사람(같은 기능)이어도 정장을 입으면 격식있어 보이고, 캐주얼을 입으면 편안해 보이죠. 버튼도 스타일에 따라 분위기가 달라집니다.

SwiftUI는 여러 가지 기본 버튼 스타일을 제공합니다.

```swift
struct ButtonStylesView: View {
    var body: some View {
        VStack(spacing: 16) {
            // 기본 스타일 (텍스트만)
            Button("기본 스타일") { }

            // 테두리 스타일
            Button("Bordered") { }
                .buttonStyle(.bordered)

            // 눈에 띄는 스타일 (배경색 채움)
            Button("Bordered Prominent") { }
                .buttonStyle(.borderedProminent)

            // 테두리 없는 스타일
            Button("Borderless") { }
                .buttonStyle(.borderless)

            // 일반 스타일 (탭 효과 없음)
            Button("Plain") { }
                .buttonStyle(.plain)

            // tint로 색상 변경
            Button("커스텀 색상") { }
                .buttonStyle(.borderedProminent)
                .tint(.green)

            // controlSize로 크기 변경
            HStack {
                Button("Mini") { }
                    .controlSize(.mini)
                Button("Small") { }
                    .controlSize(.small)
                Button("Regular") { }
                    .controlSize(.regular)
                Button("Large") { }
                    .controlSize(.large)
            }
            .buttonStyle(.borderedProminent)

            // 비활성화
            Button("비활성화 버튼") { }
                .buttonStyle(.borderedProminent)
                .disabled(true)  // 탭 불가, 회색 처리
        }
        .padding()
    }
}

#Preview {
    ButtonStylesView()
}
```

| 스타일 | 모양 | 사용 상황 |
|--------|------|----------|
| 기본 (없음) | 텍스트만 | 네비게이션 링크 느낌 |
| `.bordered` | 은은한 배경 | 보조 액션 |
| `.borderedProminent` | 채워진 배경 | 주요 액션 (CTA) |
| `.borderless` | 테두리 없음 | 목록 안의 버튼 |
| `.plain` | 꾸밈 없음 | 완전히 커스텀할 때 |

### 개념 4: 다양한 입력 컨트롤

> 💡 **비유**: 입력 컨트롤은 **자동차 계기판**과 같습니다. 스위치(Toggle)로 라이트를 켜고 끄고, 슬라이더(Slider)로 음량을 조절하고, 스테퍼(Stepper)로 온도를 정밀하게 올리고 내리죠.

버튼 외에도 사용자 입력을 받을 수 있는 다양한 컨트롤이 있습니다.

```swift
struct InputControlsView: View {
    @State private var isNotificationOn = true
    @State private var volume: Double = 50
    @State private var quantity = 1
    @State private var selectedColor = "빨강"
    let colors = ["빨강", "파랑", "초록", "노랑"]

    var body: some View {
        VStack(spacing: 24) {
            // Toggle — 켜기/끄기 스위치
            Toggle("알림 받기", isOn: $isNotificationOn)

            // Slider — 연속 값 조절
            VStack(alignment: .leading) {
                Text("볼륨: \(Int(volume))%")
                Slider(value: $volume, in: 0...100, step: 1) {
                    Text("볼륨")
                } minimumValueLabel: {
                    Image(systemName: "speaker.fill")
                } maximumValueLabel: {
                    Image(systemName: "speaker.wave.3.fill")
                }
            }

            // Stepper — 정수 단위 증감
            Stepper("수량: \(quantity)개", value: $quantity, in: 1...99)

            // Picker — 여러 옵션 중 선택
            Picker("색상", selection: $selectedColor) {
                ForEach(colors, id: \.self) { color in
                    Text(color).tag(color)
                }
            }
            .pickerStyle(.segmented)  // 세그먼트 스타일

            // 현재 상태 요약
            VStack(alignment: .leading, spacing: 8) {
                Text("현재 설정:")
                    .font(.headline)
                Text("알림: \(isNotificationOn ? "켜짐" : "꺼짐")")
                Text("볼륨: \(Int(volume))%")
                Text("수량: \(quantity)개")
                Text("색상: \(selectedColor)")
            }
            .font(.body)
            .padding()
            .background(.gray.opacity(0.1))
            .clipShape(RoundedRectangle(cornerRadius: 8))
        }
        .padding()
    }
}

#Preview {
    InputControlsView()
}
```

여기서 `$isNotificationOn`처럼 변수 앞에 `$`가 붙은 것이 보이죠? 이것은 **바인딩(Binding)** 이라고 합니다. "이 값을 읽기만 하는 게 아니라, 수정도 할 수 있게 연결해줘"라는 의미예요. Toggle이 스위치를 바꿀 때, Slider가 값을 조절할 때 `$`를 통해 원본 @State 값이 직접 업데이트됩니다. 바인딩은 [Ch5. 상태 관리](../05-state-management/01-state-binding.md)에서 더 자세히 배웁니다.

> 🔥 **실무 팁**: `$` (달러 사인)은 @State 변수의 **projected value**로, Binding 타입을 반환합니다. @State 변수 자체는 읽기용, `$`를 붙이면 읽기+쓰기 양방향 연결이 됩니다. 지금은 "입력 컨트롤에는 `$`를 붙인다"고 기억하세요!

## 실습: 주문 앱 만들기

지금까지 배운 버튼과 컨트롤들을 조합해서 간단한 커피 주문 화면을 만들어봅시다.

```swift
import SwiftUI

struct CoffeeOrderView: View {
    @State private var coffeeType = "아메리카노"
    @State private var isIced = false
    @State private var shotCount = 2
    @State private var sugarLevel: Double = 50
    @State private var orderPlaced = false

    let coffeeTypes = ["아메리카노", "라떼", "카푸치노", "모카"]

    var body: some View {
        VStack(spacing: 20) {
            // 타이틀
            Text("☕ 커피 주문")
                .font(.largeTitle)
                .fontWeight(.bold)

            // 커피 종류 선택
            Picker("커피 종류", selection: $coffeeType) {
                ForEach(coffeeTypes, id: \.self) { type in
                    Text(type).tag(type)
                }
            }
            .pickerStyle(.segmented)

            // 아이스 여부
            Toggle("아이스로 주문", isOn: $isIced)

            // 샷 수
            Stepper("에스프레소 샷: \(shotCount)샷", value: $shotCount, in: 1...5)

            // 당도 조절
            VStack(alignment: .leading) {
                Text("당도: \(Int(sugarLevel))%")
                Slider(value: $sugarLevel, in: 0...100, step: 25)
            }

            // 주문 요약
            VStack(alignment: .leading, spacing: 4) {
                Text("주문 요약")
                    .font(.headline)
                Text("\(isIced ? "아이스" : "핫") \(coffeeType)")
                Text("에스프레소 \(shotCount)샷 · 당도 \(Int(sugarLevel))%")
                    .foregroundStyle(.secondary)
            }
            .padding()
            .frame(maxWidth: .infinity, alignment: .leading)
            .background(.gray.opacity(0.1))
            .clipShape(RoundedRectangle(cornerRadius: 12))

            // 주문 버튼
            Button {
                orderPlaced = true
            } label: {
                Text("주문하기")
                    .font(.title3)
                    .fontWeight(.semibold)
                    .frame(maxWidth: .infinity)
            }
            .buttonStyle(.borderedProminent)
            .controlSize(.large)
            .tint(.brown)
            .disabled(orderPlaced)  // 주문 완료 후 비활성화

            // 주문 완료 메시지
            if orderPlaced {
                Label("주문이 접수되었습니다!", systemImage: "checkmark.circle.fill")
                    .foregroundStyle(.green)
                    .font(.headline)

                Button("새 주문") {
                    orderPlaced = false
                }
                .buttonStyle(.bordered)
            }
        }
        .padding()
    }
}

#Preview {
    CoffeeOrderView()
}
```

## 더 깊이 알아보기

### Button의 역할(Role)

iOS 15부터 Button에 **역할(role)** 을 지정할 수 있습니다. 역할에 따라 버튼의 기본 스타일이 자동으로 달라져요.

```swift
// 삭제 버튼 — 자동으로 빨간색
Button("삭제", role: .destructive) {
    // 삭제 로직
}

// 취소 버튼 — 자동으로 회색/굵게
Button("취소", role: .cancel) {
    // 취소 로직
}
```

이 역할 시스템은 Apple이 **일관된 사용자 경험**을 위해 도입한 것입니다. 앱마다 삭제 버튼의 색이 다르면 사용자가 혼란스럽겠죠? `role: .destructive`만 지정하면 시스템이 알아서 적절한 위험 신호(빨간색)를 표시해줍니다.

### SwiftUI의 "뷰는 가볍다" 철학

SwiftUI에서 Button을 누를 때마다 `body`가 다시 계산된다고 했죠. "매번 뷰를 새로 만들면 느리지 않나?"라고 걱정될 수 있는데, SwiftUI의 View 구조체는 **매우 가볍습니다**. 실제 화면을 그리는 것이 아니라 "어떻게 보일지"의 설명서일 뿐이거든요. SwiftUI는 이전 설명서와 새 설명서를 비교(diffing)해서 **바뀐 부분만** 효율적으로 업데이트합니다.

## 흔한 오해와 팁

> ⚠️ **흔한 오해**: "Button 안에서 무거운 작업을 해도 된다" — Button의 액션 클로저는 **메인 스레드**에서 실행됩니다. 네트워크 호출이나 파일 처리 같은 무거운 작업을 여기서 직접 하면 UI가 멈출 수 있어요. 무거운 작업은 [Ch7. 네트워킹과 동시성](../07-networking/01-async-await.md)에서 배울 async/await를 사용해야 합니다.

> 🔥 **실무 팁**: `.borderedProminent` 스타일은 화면에 **하나만** 사용하는 것이 좋습니다. 주요 행동(CTA: Call To Action)을 강조하기 위한 스타일이거든요. 보조 버튼에는 `.bordered`를 사용하세요.

> 💡 **알고 계셨나요?**: SwiftUI의 Button은 접근성(Accessibility)을 자동으로 지원합니다. VoiceOver가 켜져 있으면 버튼의 레이블 텍스트를 읽어주고, 탭 동작도 자동으로 인식됩니다. 별도의 접근성 코드를 추가할 필요가 없어요!

## 핵심 정리

| 개념 | 설명 |
|------|------|
| Button | 액션(Action) + 레이블(Label)로 구성된 탭 가능한 뷰 |
| @State | 뷰가 소유하는 상태 변수. 값이 바뀌면 화면 자동 업데이트 |
| $ (바인딩) | @State 변수에 `$`를 붙여 읽기+쓰기 양방향 연결 |
| .buttonStyle() | `.bordered`, `.borderedProminent` 등 기본 스타일 적용 |
| role | `.destructive`, `.cancel`로 버튼의 의미를 시스템에 전달 |
| Toggle | 켜기/끄기 스위치 — `$`로 Bool 값에 바인딩 |
| Slider | 연속 값 조절 — `$`로 Double 값에 바인딩 |
| Stepper | 정수 단위 증감 — `$`로 Int 값에 바인딩 |
| Picker | 여러 옵션 중 하나를 선택 |

## 다음 섹션 미리보기

버튼과 컨트롤을 배웠는데, 지금까지는 모든 걸 VStack 안에 세로로만 쌓았죠? 다음으로 [04. 레이아웃 시스템](./04-layout.md)에서 HStack, ZStack, Spacer, padding을 활용해 뷰를 **자유롭게 배치**하는 법을 배웁니다. 진짜 앱다운 화면 구성이 시작됩니다!

## 참고 자료

- [Apple Button 공식 문서](https://developer.apple.com/documentation/swiftui/button) - Button 뷰의 모든 API
- [Apple State 공식 문서](https://developer.apple.com/documentation/swiftui/state) - @State 프로퍼티 래퍼
- [WWDC 2019 - SwiftUI Essentials](https://developer.apple.com/videos/play/wwdc2019/216/) - SwiftUI 핵심 개념 (버튼, 상태 포함)
- [Apple Human Interface Guidelines - Buttons](https://developer.apple.com/design/human-interface-guidelines/buttons) - 버튼 디자인 가이드라인
- [WWDC 2023 - Demystify SwiftUI performance](https://developer.apple.com/videos/play/wwdc2023/10160/) - SwiftUI 뷰 업데이트 원리
