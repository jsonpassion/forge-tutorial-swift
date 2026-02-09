# iOS 26 Liquid Glass 디자인

> glassEffect, GlassEffectContainer, 머티리얼, 새로운 디자인 언어

## 개요

iOS 26에서 Apple은 역대 가장 큰 디자인 변화를 가져왔습니다. 바로 **Liquid Glass** — 빛을 굴절시키고 뒤쪽 콘텐츠를 비추는 반투명 유리 소재예요. 네비게이션 바, 탭 바, 툴바가 모두 유리처럼 변했고, 우리도 커스텀 뷰에 이 효과를 적용할 수 있습니다.

**선수 지식**: [04. SF Symbols과 에셋 관리](./04-sf-symbols.md)에서 배운 아이콘 활용법
**학습 목표**:
- Liquid Glass 디자인 철학 이해하기
- .glassEffect() 수정자로 커스텀 뷰에 유리 효과 적용하기
- GlassEffectContainer로 유리 요소 그룹화하기
- .glass / .glassProminent 버튼 스타일 활용하기
- 접근성 고려사항 이해하기

## 왜 알아야 할까?

iOS 26으로 앱을 빌드하면 기본 UI 컴포넌트가 **자동으로** Liquid Glass 스타일을 적용받아요. 하지만 커스텀 뷰나 플로팅 버튼 같은 요소에 유리 효과를 직접 적용하려면 새로운 API를 알아야 합니다. 이 디자인 언어를 이해하지 못하면, 앱이 시스템과 어울리지 않는 **동떨어진** 느낌을 줄 수 있어요.

## 핵심 개념

### 개념 1: Liquid Glass란?

> 💡 **비유**: Liquid Glass는 **반투명 유리창**입니다. 뒤에 있는 콘텐츠가 은은하게 비치면서 계층감을 주는 효과죠. 오래된 유리창처럼 빛이 살짝 굴절되어, 뒤쪽의 색과 형태가 왜곡되며 독특한 깊이감을 만들어냅니다.

Liquid Glass는 단순한 블러(흐림) 효과가 아닙니다. 핵심적인 3가지 특징이 있어요:

- **렌싱(Lensing)**: 빛을 굴절시켜 뒤쪽 콘텐츠가 살짝 왜곡되어 보임
- **스펙큘러 하이라이트**: 실시간으로 빛 반사를 시뮬레이션
- **적응형 색상**: 뒤쪽 콘텐츠에 따라 유리의 밝기와 색감이 자동 조절

iOS 26에서는 다음 컴포넌트가 **자동으로** Liquid Glass를 적용받습니다:
- NavigationBar (네비게이션 바)
- TabBar (탭 바)
- Toolbar (툴바)
- Sheet (부분 높이일 때)
- Alert, ConfirmationDialog

코드를 한 줄도 바꾸지 않아도 이 효과가 자동으로 적용돼요!

### 개념 2: .glassEffect() — 커스텀 뷰에 유리 효과 적용

> 💡 **비유**: `.glassEffect()`는 뷰 위에 **유리판을 올려놓는 것**이에요. 유리판의 종류(투명도)와 모양(원형, 캡슐, 사각형)을 선택할 수 있습니다.

```swift
import SwiftUI

struct GlassEffectDemoView: View {
    var body: some View {
        ZStack {
            // 배경 이미지 (유리 효과가 비출 콘텐츠)
            LinearGradient(
                colors: [.blue, .purple, .pink],
                startPoint: .topLeading,
                endPoint: .bottomTrailing
            )
            .ignoresSafeArea()

            VStack(spacing: 30) {
                // 기본 유리 효과 (.regular)
                Text("Regular Glass")
                    .font(.headline)
                    .padding()
                    .glassEffect()

                // 투명 유리 효과 (.clear)
                Text("Clear Glass")
                    .font(.headline)
                    .padding()
                    .glassEffect(.clear)

                // 원형 유리
                Image(systemName: "heart.fill")
                    .font(.title)
                    .padding(20)
                    .glassEffect(.regular, in: .circle)

                // 둥근 사각형 유리
                Label("설정", systemImage: "gear")
                    .font(.headline)
                    .padding()
                    .glassEffect(
                        .regular,
                        in: RoundedRectangle(cornerRadius: 16)
                    )
            }
        }
    }
}

#Preview {
    GlassEffectDemoView()
}
```

Glass 타입에는 3가지 변형이 있어요:

| 타입 | 용도 |
|------|------|
| `.regular` | 대부분의 UI 요소에 사용 (기본값) |
| `.clear` | 미디어가 풍부한 배경 위에서 더 투명하게 |
| `.identity` | 효과 없음 (조건부로 끌 때 사용) |

### 개념 3: 틴트와 인터랙티브 효과

유리에 색을 입히거나 터치에 반응하는 효과를 줄 수 있어요.

```swift
import SwiftUI

struct GlassTintDemoView: View {
    var body: some View {
        ZStack {
            // 배경
            Image(systemName: "mountain.2.fill")
                .font(.system(size: 200))
                .foregroundStyle(.green.opacity(0.3))
                .frame(maxWidth: .infinity, maxHeight: .infinity)
                .background(Color.blue.gradient)
                .ignoresSafeArea()

            VStack(spacing: 20) {
                // 파란색 틴트 유리
                Button("파란 유리 버튼") { }
                    .font(.headline)
                    .padding()
                    .glassEffect(.regular.tint(.blue))

                // 빨간색 틴트 유리
                Button("빨간 유리 버튼") { }
                    .font(.headline)
                    .padding()
                    .glassEffect(.regular.tint(.red))

                // 인터랙티브 유리: 탭하면 반짝이고 튀어오름
                Button("인터랙티브 유리") { }
                    .font(.headline)
                    .padding()
                    .glassEffect(.regular.interactive())

                // 틴트 + 인터랙티브 조합
                Button("조합 효과") { }
                    .font(.headline)
                    .padding()
                    .glassEffect(
                        .regular
                            .tint(.purple.opacity(0.8))
                            .interactive()
                    )
            }
        }
    }
}

#Preview {
    GlassTintDemoView()
}
```

> 🔥 **실무 팁**: `.tint()`는 **장식이 아닌 의미 전달**을 위해 사용하세요. 빨간색은 삭제/위험, 파란색은 주요 액션 같은 방식으로요. 무작정 화려하게 만들면 오히려 혼란스러워집니다.

### 개념 4: .glass 버튼 스타일

iOS 26에서는 버튼에 전용 유리 스타일을 적용할 수 있어요.

```swift
import SwiftUI

struct GlassButtonDemoView: View {
    var body: some View {
        ZStack {
            Color.indigo.gradient.ignoresSafeArea()

            VStack(spacing: 20) {
                // 일반 유리 버튼 (보조 액션)
                Button("보조 액션") { }
                    .buttonStyle(.glass)

                // 강조 유리 버튼 (주요 액션)
                Button("주요 액션") { }
                    .buttonStyle(.glassProminent)

                // 아이콘 + 텍스트 유리 버튼
                Button("저장", systemImage: "square.and.arrow.down") { }
                    .buttonStyle(.glass)

                // 아이콘만 있는 유리 버튼
                Button("", systemImage: "plus") { }
                    .buttonStyle(.glass)
            }
            .font(.headline)
        }
    }
}

#Preview {
    GlassButtonDemoView()
}
```

| 스타일 | 외관 | 용도 |
|--------|------|------|
| `.glass` | 반투명, 배경이 비침 | 보조 액션 |
| `.glassProminent` | 불투명, 배경이 보이지 않음 | 주요 액션 |

### 개념 5: GlassEffectContainer — 유리 요소 그룹화

> 💡 **비유**: GlassEffectContainer는 **물방울들이 모이는 그릇**이에요. 가까이 있는 물방울이 하나로 합쳐지듯, 유리 요소들이 서로 가까우면 하나의 유리판으로 합쳐지는 효과가 나타납니다.

```swift
import SwiftUI

struct GlassContainerDemoView: View {
    var body: some View {
        ZStack {
            LinearGradient(
                colors: [.orange, .pink, .purple],
                startPoint: .top,
                endPoint: .bottom
            )
            .ignoresSafeArea()

            // 컨테이너 안의 유리 요소들이 서로 블렌딩됨
            GlassEffectContainer {
                HStack(spacing: 8) {
                    Button("홈", systemImage: "house.fill") { }
                        .glassEffect()

                    Button("검색", systemImage: "magnifyingglass") { }
                        .glassEffect()

                    Button("설정", systemImage: "gear") { }
                        .glassEffect()
                }
                .font(.headline)
                .labelStyle(.iconOnly)
                .padding()
            }
        }
    }
}

#Preview {
    GlassContainerDemoView()
}
```

### 개념 6: 플로팅 액션 버튼 패턴

Liquid Glass와 잘 어울리는 **플로팅 액션 버튼** 패턴을 만들어볼까요?

```swift
import SwiftUI

struct FloatingButtonView: View {
    var body: some View {
        NavigationStack {
            ZStack(alignment: .bottomTrailing) {
                // 메인 콘텐츠
                List(1...30, id: \.self) { item in
                    Text("항목 \(item)")
                }
                .navigationTitle("메모")

                // 플로팅 액션 버튼
                Button {
                    print("새 메모 작성")
                } label: {
                    Label("새 메모", systemImage: "plus")
                        .bold()
                        .labelStyle(.iconOnly)
                        .padding()
                }
                // 인터랙티브 유리 효과 + 원형
                .glassEffect(.regular.interactive(), in: .circle)
                .padding([.bottom, .trailing], 16)
            }
        }
    }
}

#Preview {
    FloatingButtonView()
}
```

## 실습: 직접 해보기

Liquid Glass 스타일의 미니 음악 플레이어를 만들어봅시다.

```swift
import SwiftUI

struct MiniPlayerView: View {
    @State private var isPlaying = false

    var body: some View {
        ZStack {
            // 앨범 아트 배경
            LinearGradient(
                colors: [.blue, .indigo, .purple],
                startPoint: .topLeading,
                endPoint: .bottomTrailing
            )
            .ignoresSafeArea()

            VStack {
                Spacer()

                // 미니 플레이어 (Liquid Glass)
                GlassEffectContainer {
                    HStack(spacing: 16) {
                        // 앨범 아트
                        RoundedRectangle(cornerRadius: 8)
                            .fill(.white.opacity(0.3))
                            .frame(width: 50, height: 50)
                            .overlay {
                                Image(systemName: "music.note")
                                    .font(.title2)
                            }

                        // 곡 정보
                        VStack(alignment: .leading) {
                            Text("좋은 노래")
                                .font(.headline)
                            Text("멋진 아티스트")
                                .font(.caption)
                                .foregroundStyle(.secondary)
                        }

                        Spacer()

                        // 컨트롤 버튼들
                        Button("이전", systemImage: "backward.fill") { }
                            .labelStyle(.iconOnly)
                            .glassEffect()

                        Button(
                            isPlaying ? "일시정지" : "재생",
                            systemImage: isPlaying
                                ? "pause.fill"
                                : "play.fill"
                        ) {
                            isPlaying.toggle()
                        }
                        .labelStyle(.iconOnly)
                        .contentTransition(.symbolEffect(.replace))
                        .glassEffect(.regular.interactive())

                        Button("다음", systemImage: "forward.fill") { }
                            .labelStyle(.iconOnly)
                            .glassEffect()
                    }
                    .padding()
                    .glassEffect(
                        .regular,
                        in: RoundedRectangle(cornerRadius: 20)
                    )
                }
                .padding()
            }
        }
    }
}

#Preview {
    MiniPlayerView()
}
```

## 더 깊이 알아보기

### Liquid Glass의 탄생

Liquid Glass는 **WWDC 2025**에서 "Meet Liquid Glass" (Session 219) 세션을 통해 발표되었습니다. Apple은 이렇게 설명했어요: *"렌싱(빛의 굴절)은 자연 세계 어디에서나 일어나는 현상이고, 우리는 모두 투명한 물체가 빛을 왜곡하고 굴절시키는 것을 직관적으로 이해합니다."*

이전의 머티리얼(블러 효과)이 빛을 **산란**시켰다면, Liquid Glass는 빛을 **굴절**시킵니다. 이것은 시각적 디자인에서 근본적인 차이예요 — 마치 진짜 유리가 앱 위에 올려져 있는 것 같은 물리적 존재감을 만들어냅니다. Liquid Glass 요소는 나타날 때 빛의 굴절이 점진적으로 변화하면서 마치 물에서 솟아오르듯 자연스럽게 나타나고, 상호작용할 때 젤리처럼 탄력적으로 반응합니다.

### Liquid Glass의 핵심 원칙

Apple은 Liquid Glass를 사용할 때 지켜야 할 원칙을 제시했어요:

**콘텐츠 레이어 vs 네비게이션 레이어**
- **콘텐츠 레이어** (아래): 리스트, 텍스트, 이미지 등 주요 콘텐츠 → 유리 효과 **적용하지 않음**
- **네비게이션 레이어** (위): 툴바, 탭 바, 플로팅 버튼 등 컨트롤 → 유리 효과 **적용**

이 구분을 무시하고 리스트의 모든 행에 유리를 적용하면 매우 어색한 인터페이스가 됩니다!

## 흔한 오해와 팁

> ⚠️ **흔한 오해**: "모든 뷰에 `.glassEffect()`를 넣으면 예쁘겠지?" — 절대 아닙니다! Liquid Glass는 **네비게이션 레이어**에만 적용해야 합니다. 콘텐츠(리스트, 텍스트 등)에 적용하면 읽기 어렵고 시각적으로 혼란스러워요.

> 🔥 **실무 팁**: iOS 26 이전 버전도 지원해야 한다면, `@available(iOS 26, *)`로 분기 처리하세요. 유리 효과는 iOS 26 전용이라서 이전 버전에서는 기존 머티리얼(`.ultraThinMaterial` 등)을 사용하면 됩니다.

> 💡 **알고 계셨나요?**: Liquid Glass는 접근성 설정에 **자동 대응**합니다. "투명도 줄이기"를 켜면 유리가 더 불투명해지고, "대비 높이기"를 켜면 테두리가 강조되며, "모션 줄이기"를 켜면 애니메이션이 줄어들어요. 개발자가 별도로 처리할 필요가 없습니다!

## 핵심 정리

| 개념 | 설명 |
|------|------|
| Liquid Glass | iOS 26의 새 디자인 언어, 빛을 굴절시키는 유리 소재 |
| .glassEffect() | 뷰에 유리 효과를 적용하는 수정자 |
| Glass.regular | 기본 유리 (대부분의 경우) |
| Glass.clear | 더 투명한 유리 (미디어 배경 위에서) |
| .tint() | 유리에 의미적 색상 부여 |
| .interactive() | 터치 시 반짝임/바운스 효과 |
| GlassEffectContainer | 여러 유리 요소를 그룹화하여 블렌딩 |
| .buttonStyle(.glass) | 보조 유리 버튼 |
| .buttonStyle(.glassProminent) | 주요 유리 버튼 |

## 다음 섹션 미리보기

Ch4를 완료했습니다! 화면 구성과 네비게이션의 기초를 모두 배웠어요. 다음 챕터에서는 SwiftUI의 핵심인 **상태 관리**를 배웁니다. @State, @Binding, @Observable — 데이터가 변하면 화면이 자동으로 업데이트되는 SwiftUI의 마법을 알아볼까요? [Ch5. 상태 관리](../05-state-management/01-state-binding.md)에서 만나요!

## 참고 자료

- [Meet Liquid Glass - WWDC25](https://developer.apple.com/videos/play/wwdc2025/219/) - Liquid Glass의 철학과 핵심 원리
- [Build a SwiftUI app with the new design - WWDC25](https://developer.apple.com/videos/play/wwdc2025/323/) - SwiftUI에서 Liquid Glass 적용 실전 가이드
- [Applying Liquid Glass to custom views - Apple Developer](https://developer.apple.com/documentation/SwiftUI/Applying-Liquid-Glass-to-custom-views) - 커스텀 뷰 유리 효과 공식 문서
- [Adopting Liquid Glass - Apple Developer](https://developer.apple.com/documentation/technologyoverviews/adopting-liquid-glass) - Liquid Glass 전체 도입 가이드
