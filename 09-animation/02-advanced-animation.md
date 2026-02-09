# 고급 애니메이션

> spring, keyframe, PhaseAnimator, 타이밍 커브

## 개요

기본 애니메이션만으로도 멋진 UI를 만들 수 있지만, 앱에 "와, 이 앱 뭔가 다르다"라는 느낌을 주려면 더 정교한 도구가 필요합니다. 이 섹션에서는 스프링의 물리적 특성을 세밀하게 제어하고, 여러 단계를 거치는 복잡한 애니메이션을 선언적으로 구현하는 방법을 배웁니다.

**선수 지식**: [01. 기본 애니메이션](./01-basic-animation.md)
**학습 목표**:
- 스프링 애니메이션의 물리적 파라미터(duration, bounce) 이해하기
- `KeyframeAnimator`로 다단계 키프레임 애니메이션 구현하기
- `PhaseAnimator`로 자동 순환 애니메이션 만들기
- `CustomAnimation` 프로토콜로 나만의 애니메이션 타입 만들기

## 왜 알아야 할까?

Apple의 앱을 자세히 관찰하면 단순한 페이드나 슬라이드가 아닌, 여러 속성이 서로 다른 타이밍으로 변하는 복잡한 애니메이션이 곳곳에 숨어 있습니다. 사진 앱에서 이미지를 확대할 때의 탄성 있는 움직임, 메시지 앱에서 말풍선이 나타날 때의 바운스 — 이런 디테일이 사용자에게 "고급스럽다"는 느낌을 줍니다. 이 섹션의 도구들로 그 수준의 애니메이션을 만들 수 있습니다.

## 핵심 개념

### 개념 1: 스프링 애니메이션 깊이 파기

> 💡 **비유**: 스프링 애니메이션은 실제 **용수철에 매달린 추**의 움직임을 시뮬레이션합니다. 용수철이 딱딱하면(stiffness↑) 빠르게 튕기고, 물에 담가져 있으면(damping↑) 금방 멈추죠. iOS 17부터는 이런 물리 파라미터 대신 **duration**(얼마나 오래)과 **bounce**(얼마나 튕기나)로 직관적으로 제어합니다.

```swift
import SwiftUI

struct SpringComparisonView: View {
    @State private var animate = false

    var body: some View {
        VStack(spacing: 24) {
            Text("스프링 프리셋 비교")
                .font(.headline)

            // 세 가지 프리셋 비교
            springRow("smooth", .smooth, .blue)
            springRow("snappy", .snappy, .green)
            springRow("bouncy", .bouncy, .orange)

            // 커스텀 스프링
            springRow("커스텀", .spring(duration: 0.8, bounce: 0.6), .purple)

            Button("애니메이트") {
                animate.toggle()
            }
            .buttonStyle(.borderedProminent)
        }
        .padding()
    }

    // 스프링 비교를 위한 헬퍼 뷰
    func springRow(_ name: String, _ animation: Animation, _ color: Color) -> some View {
        HStack {
            Text(name)
                .font(.caption)
                .frame(width: 60, alignment: .leading)
            RoundedRectangle(cornerRadius: 8)
                .fill(color.gradient)
                .frame(width: 40, height: 40)
                .offset(x: animate ? 150 : 0)
                .animation(animation, value: animate)
        }
    }
}

#Preview {
    SpringComparisonView()
}
```

스프링 파라미터 요약:

| 프리셋 | duration | bounce | 특징 |
|--------|----------|--------|------|
| `.smooth` | 0.5 | 0 | 바운스 없이 부드럽게 안착 |
| `.snappy` | 0.5 | ~0.15 | 약간의 반동, 빠른 느낌 |
| `.bouncy` | 0.5 | ~0.25 | 눈에 보이는 바운스 |
| 커스텀 | 자유 | -1.0~1.0 | 완전 제어 (음수 = 감쇠↑) |

### 개념 2: KeyframeAnimator — 영화같은 다단계 애니메이션

> 💡 **비유**: `KeyframeAnimator`는 **스톱모션 애니메이션 감독**입니다. "1초에는 여기, 2초에는 저기, 3초에는 이 크기로" 각 장면(키프레임)을 지정하면, SwiftUI가 장면 사이를 자연스럽게 연결합니다.

iOS 17에서 도입된 `KeyframeAnimator`는 여러 프로퍼티가 **각각 다른 타이밍**으로 변하는 복잡한 애니메이션을 만들 수 있습니다.

```swift
import SwiftUI

struct KeyframeAnimationView: View {
    @State private var trigger = false

    var body: some View {
        VStack(spacing: 40) {
            // 로켓 발사 애니메이션!
            KeyframeAnimator(
                initialValue: RocketState(),
                trigger: trigger
            ) { state in
                // 현재 보간된 state를 기반으로 뷰 렌더링
                Image(systemName: "paperplane.fill")
                    .font(.system(size: 50))
                    .foregroundStyle(.blue)
                    .scaleEffect(state.scale)
                    .rotationEffect(.degrees(state.rotation))
                    .offset(y: state.yOffset)
                    .opacity(state.opacity)
            } keyframes: { _ in
                // y 위치 트랙: 위로 올라갔다가 돌아옴
                KeyframeTrack(\.yOffset) {
                    SpringKeyframe(0, duration: 0.2)
                    SpringKeyframe(-100, duration: 0.5, spring: .bouncy)
                    CubicKeyframe(-50, duration: 0.3)
                    SpringKeyframe(0, duration: 0.4, spring: .smooth)
                }

                // 크기 트랙: 커졌다 작아졌다
                KeyframeTrack(\.scale) {
                    LinearKeyframe(1.0, duration: 0.1)
                    SpringKeyframe(1.5, duration: 0.3, spring: .bouncy)
                    SpringKeyframe(1.0, duration: 0.5)
                }

                // 회전 트랙
                KeyframeTrack(\.rotation) {
                    LinearKeyframe(0, duration: 0.2)
                    CubicKeyframe(-30, duration: 0.3)
                    CubicKeyframe(30, duration: 0.3)
                    SpringKeyframe(0, duration: 0.3)
                }
            }

            Button("발사!") {
                trigger.toggle()
            }
            .buttonStyle(.borderedProminent)
        }
    }
}

// 키프레임 상태를 담는 구조체
struct RocketState {
    var yOffset: CGFloat = 0
    var scale: CGFloat = 1.0
    var rotation: Double = 0
    var opacity: Double = 1.0
}

#Preview {
    KeyframeAnimationView()
}
```

키프레임 타입 비교:

| 키프레임 | 보간 방식 | 용도 |
|---------|----------|------|
| `LinearKeyframe` | 직선 보간 | 일정한 속도 변화 |
| `SpringKeyframe` | 스프링 물리 | 자연스러운 움직임 |
| `CubicKeyframe` | 큐빅 커브 (부드러운 곡선) | 가속/감속 효과 |
| `MoveKeyframe` | 보간 없이 즉시 이동 | 순간 변화 |

### 개념 3: PhaseAnimator — 자동 순환 애니메이션

> 💡 **비유**: `PhaseAnimator`는 **회전목마**와 같습니다. 정해진 단계(Phase)를 순서대로 돌며 자동으로 반복하죠. 트리거 없이도 계속 돌아가거나, 특정 이벤트에 한 사이클만 실행할 수 있습니다.

```swift
import SwiftUI

// 애니메이션 페이즈 정의
enum PulsePhase: CaseIterable {
    case initial    // 시작 상태
    case expand     // 확장
    case contract   // 수축

    var scale: CGFloat {
        switch self {
        case .initial: 1.0
        case .expand: 1.3
        case .contract: 0.9
        }
    }

    var opacity: Double {
        switch self {
        case .initial: 1.0
        case .expand: 0.7
        case .contract: 1.0
        }
    }

    var color: Color {
        switch self {
        case .initial: .blue
        case .expand: .purple
        case .contract: .blue
        }
    }
}

struct PhaseAnimatorView: View {
    var body: some View {
        VStack(spacing: 40) {
            // 자동 반복 (trigger 없이)
            PhaseAnimator(PulsePhase.allCases) { phase in
                Circle()
                    .fill(phase.color.gradient)
                    .frame(width: 100, height: 100)
                    .scaleEffect(phase.scale)
                    .opacity(phase.opacity)
            } animation: { phase in
                // 각 페이즈 전환마다 다른 애니메이션 적용 가능
                switch phase {
                case .initial: .smooth(duration: 0.4)
                case .expand: .spring(duration: 0.5, bounce: 0.3)
                case .contract: .easeOut(duration: 0.3)
                }
            }

            Text("자동 맥박 애니메이션")
                .foregroundStyle(.secondary)
        }
    }
}

#Preview {
    PhaseAnimatorView()
}
```

> 🔥 **실무 팁**: `PhaseAnimator`는 로딩 인디케이터, 주의 끌기 효과, 온보딩 강조 애니메이션에 안성맞춤입니다. `trigger` 파라미터를 추가하면 특정 이벤트 시에만 한 사이클을 실행하는 **트리거 모드**로 사용할 수 있어요.

### 개념 4: contentTransition — 텍스트와 심볼 전환

숫자나 텍스트가 변할 때 단순히 "뚝" 바뀌는 대신, 부드러운 전환 효과를 줄 수 있습니다.

```swift
import SwiftUI

struct ContentTransitionView: View {
    @State private var count = 0

    var body: some View {
        VStack(spacing: 30) {
            // 숫자가 슬롯머신처럼 굴러가며 바뀜
            Text("\(count)")
                .font(.system(size: 72, weight: .bold, design: .rounded))
                .contentTransition(.numericText(countsDown: false))

            // SF Symbol 전환 효과
            Image(systemName: count % 2 == 0 ? "sun.max.fill" : "moon.fill")
                .font(.system(size: 50))
                .foregroundStyle(count % 2 == 0 ? .yellow : .indigo)
                .contentTransition(.symbolEffect(.replace))

            Button("+1") {
                withAnimation(.snappy) {
                    count += 1
                }
            }
            .buttonStyle(.borderedProminent)
        }
    }
}

#Preview {
    ContentTransitionView()
}
```

## 더 깊이 알아보기

### 스프링 애니메이션의 물리학

스프링 애니메이션의 수학적 기반은 **감쇠 조화 진동(Damped Harmonic Oscillation)**입니다. 실제 스프링이 늘어났다 줄어드는 것과 동일한 물리 방정식을 사용하죠. iOS 17 이전에는 `mass`(질량), `stiffness`(강성), `damping`(감쇠)이라는 물리 파라미터를 직접 지정해야 했는데, 사실 이 값들의 의미를 직관적으로 이해하기 어려웠습니다.

WWDC 2023에서 Apple의 엔지니어들은 `duration`과 `bounce`라는 **인지적(perceptual)** 파라미터로 전환했습니다. "0.5초 동안, 30% 바운스"라고 하면 누구나 결과를 예측할 수 있으니까요. 이 접근은 Disney의 "12가지 애니메이션 원칙" 중 하나인 **Squash & Stretch(찌그러짐과 늘어남)**와도 맥이 닿아 있습니다.

> 💡 **알고 계셨나요?**: `bounce` 값이 1.0이면 영원히 멈추지 않는 완전 탄성 충돌이 되고, 0이면 바운스 없이 부드럽게 안착합니다. 음수 값(예: -0.2)은 "과감쇠(over-damped)"로, 스프링보다도 더 느리게 목표에 도달합니다.

## 흔한 오해와 팁

> ⚠️ **흔한 오해**: "KeyframeAnimator와 PhaseAnimator는 같은 것이다" — `KeyframeAnimator`는 **시간 기반**으로 프로퍼티별 타이밍을 정밀 제어하고, `PhaseAnimator`는 **상태(Phase) 기반**으로 단계를 순회합니다. 정밀한 모션이 필요하면 Keyframe, 간단한 반복/순환이면 Phase를 선택하세요.

> 🔥 **실무 팁**: `KeyframeAnimator`에서 여러 `KeyframeTrack`을 사용할 때, 각 트랙의 **총 duration이 같을 필요는 없습니다**. SwiftUI가 가장 긴 트랙에 맞춰 전체 애니메이션 길이를 결정합니다. 하지만 일관된 사용자 경험을 위해 비슷한 길이로 맞추는 것이 좋아요.

## 핵심 정리

| 개념 | 설명 |
|------|------|
| `.spring(duration:bounce:)` | iOS 17+ 직관적 스프링 파라미터 |
| `.smooth / .snappy / .bouncy` | 세 가지 스프링 프리셋 |
| `KeyframeAnimator` | 시간 기반, 프로퍼티별 독립 타이밍 제어 |
| `KeyframeTrack` | 키프레임 애니메이터의 개별 프로퍼티 트랙 |
| `PhaseAnimator` | 상태 기반 순환 애니메이션 |
| `.contentTransition` | 텍스트/심볼 전환 효과 (`.numericText`, `.symbolEffect`) |
| `CustomAnimation` | 나만의 애니메이션 타입 프로토콜 (iOS 17+) |

## 다음 섹션 미리보기

멋진 애니메이션도 사용자 입력에 반응하지 않으면 의미가 없겠죠? 다음 [03. 제스처](./03-gestures.md)에서는 탭, 드래그, 핀치, 회전 등 다양한 제스처를 인식하고 애니메이션과 결합하는 방법을 배웁니다.

## 참고 자료

- [Explore SwiftUI animation - WWDC 2023](https://developer.apple.com/videos/play/wwdc2023/10156/) — KeyframeAnimator, PhaseAnimator, 스프링 프리셋 발표
- [KeyframeAnimator - Apple 공식 문서](https://developer.apple.com/documentation/swiftui/keyframeanimator) — 키프레임 API 레퍼런스
- [PhaseAnimator - Apple 공식 문서](https://developer.apple.com/documentation/swiftui/phaseanimator) — 페이즈 애니메이터 API
- [Spring Animations - SwiftUI 가이드](https://github.com/GetStream/swiftui-spring-animations) — 스프링 파라미터 시각화 레퍼런스
