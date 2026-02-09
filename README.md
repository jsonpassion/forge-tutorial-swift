# Swift 마스터 가이드

> Swift 기초부터 앱스토어 런칭까지 완전 정복

**16챕터 73섹션**으로 구성된 A-to-Z Swift & SwiftUI 튜토리얼입니다.
iOS 26+ / SwiftUI / Liquid Glass / Swift 6 기준으로 작성되었습니다.

## 튜토리얼 현황

| 항목 | 수량 | 상태 |
|------|------|------|
| 총 챕터 | 16개 | ✅ 완료 (16/16) |
| 총 섹션 | 73개 | ✅ 완료 (73/73, 100%) |
| Part 1: Swift 언어 기초 | 11개 섹션 | ✅ 완료 |
| Part 2: SwiftUI 기본 | 14개 섹션 | ✅ 완료 |
| Part 3: 실전 앱 개발 | 14개 섹션 | ✅ 완료 |
| Part 4: 사용자 경험 | 13개 섹션 | ✅ 완료 |
| Part 5: 출시와 확장 | 21개 섹션 | ✅ 완료 |

---

## Part 1: Swift 언어 기초 (입문)

### Ch1. Swift 시작하기
- [01. 개발 환경 설정](01-swift-basics/01-introduction.md) — Xcode 설치, Playground, 첫 번째 프로그램 실행
- [02. 변수와 상수](01-swift-basics/02-variables-constants.md) — var, let, 데이터 타입, Type Safety와 Type Inference
- [03. 컬렉션 타입](01-swift-basics/03-collections.md) — Array, Dictionary, Set의 활용
- [04. 조건문과 반복문](01-swift-basics/04-control-flow.md) — if, guard, switch, for-in, while 제어 흐름
- [05. 함수와 클로저](01-swift-basics/05-functions-closures.md) — func 정의, 클로저 표현식, map/filter/reduce
- [06. 옵셔널](01-swift-basics/06-optionals.md) — Optional, 바인딩, 체이닝, nil 안전성의 철학

### Ch2. Swift 타입 시스템
- [01. 구조체와 클래스](02-swift-types/01-struct-class.md) — struct vs class, 값 타입 vs 참조 타입
- [02. 프로퍼티와 메서드](02-swift-types/02-properties-methods.md) — 저장/연산 프로퍼티, 인스턴스/타입 메서드
- [03. 프로토콜과 익스텐션](02-swift-types/03-protocols-extensions.md) — protocol, extension, 프로토콜 지향 프로그래밍
- [04. 열거형과 패턴 매칭](02-swift-types/04-enums-pattern.md) — enum, associated values, Swift의 강력한 switch
- [05. 제네릭과 에러 처리](02-swift-types/05-generics-errors.md) — Generic 타입, Result, throw/try/catch

---

## Part 2: SwiftUI 기본 (초급)

### Ch3. SwiftUI 첫걸음
- [01. Hello, SwiftUI!](03-swiftui-start/01-hello-swiftui.md) — View 프로토콜, body, #Preview, 프로젝트 구조
- [02. 텍스트와 이미지](03-swiftui-start/02-text-image.md) — Text, Image, 수정자(Modifier) 체이닝
- [03. 버튼과 인터랙션](03-swiftui-start/03-button-interaction.md) — Button, Label, 액션 처리, buttonStyle
- [04. 레이아웃 시스템](03-swiftui-start/04-layout.md) — VStack, HStack, ZStack, Spacer, padding, frame
- [05. 리스트와 스크롤](03-swiftui-start/05-lists-scroll.md) — List, ForEach, ScrollView, LazyVStack

### Ch4. 화면 구성과 네비게이션
- [01. NavigationStack](04-navigation-design/01-navigation-stack.md) — 화면 전환, NavigationLink, toolbar, navigationTitle
- [02. TabView와 모달](04-navigation-design/02-tab-modal.md) — 탭 네비게이션, sheet, fullScreenCover, alert
- [03. 폼과 사용자 입력](04-navigation-design/03-forms-input.md) — TextField, SecureField, Toggle, Picker, Slider, Form
- [04. SF Symbols과 에셋 관리](04-navigation-design/04-sf-symbols.md) — SF Symbols 6, Asset Catalog, Color Set, 다크모드 대응
- [05. iOS 26 Liquid Glass 디자인](04-navigation-design/05-liquid-glass.md) — glassEffect, GlassEffectContainer, 머티리얼, 새로운 디자인 언어

### Ch5. 상태 관리
- [01. @State와 @Binding](05-state-management/01-state-binding.md) — 뷰 내부 상태, 부모-자식 간 양방향 바인딩
- [02. @Observable 매크로](05-state-management/02-observable.md) — Observation 프레임워크, 뷰 모델과의 연결
- [03. @Environment와 앱 전역 상태](05-state-management/03-environment.md) — @Environment, custom EnvironmentKey, 앱 설정 관리
- [04. 데이터 흐름 설계](05-state-management/04-data-flow.md) — 단방향 데이터 흐름, Source of Truth, 상태 디버깅

---

## Part 3: 실전 앱 개발 (중급)

### Ch6. SwiftData
- [01. SwiftData 시작하기](06-swiftdata/01-swiftdata-intro.md) — @Model, ModelContainer, ModelContext 기초
- [02. CRUD 구현](06-swiftdata/02-crud.md) — 데이터 생성, 읽기(@Query), 수정, 삭제
- [03. 관계와 고급 쿼리](06-swiftdata/03-relationships-query.md) — @Relationship, Predicate, SortDescriptor, 필터링
- [04. 마이그레이션과 버전 관리](06-swiftdata/04-migration.md) — VersionedSchema, SchemaMigrationPlan
- [05. CloudKit 동기화](06-swiftdata/05-cloudkit.md) — iCloud 자동 동기화, 충돌 해결, 오프라인 대응

### Ch7. 네트워킹과 동시성
- [01. async/await 기초](07-networking/01-async-await.md) — Swift Concurrency, Task, 구조적 동시성
- [02. URLSession과 REST API](07-networking/02-urlsession.md) — GET/POST 요청, HTTP 상태 코드, 헤더 설정
- [03. Codable과 JSON 파싱](07-networking/03-codable.md) — JSON 디코딩/인코딩, CodingKeys, 중첩 구조 처리
- [04. 네트워크 에러 핸들링](07-networking/04-error-handling.md) — 에러 타입 설계, 사용자 피드백, 재시도 로직
- [05. 실전 API 프로젝트](07-networking/05-api-project.md) — 공개 API로 완성하는 네트워크 앱 (검색, 페이징, 캐싱)

### Ch8. 아키텍처 패턴
- [01. MVVM 패턴](08-architecture/01-mvvm.md) — Model-View-ViewModel, SwiftUI에서의 MVVM 구현
- [02. Repository 패턴](08-architecture/02-repository.md) — 데이터 소스 추상화, 프로토콜 기반 설계
- [03. 네비게이션 설계](08-architecture/03-navigation-design.md) — Router 패턴, NavigationPath, 딥링크 대응
- [04. 의존성 주입](08-architecture/04-dependency-injection.md) — DI 원칙, Environment 활용, 테스트 용이성 확보

---

## Part 4: 사용자 경험 (중상급)

### Ch9. 애니메이션과 인터랙션
- [01. 기본 애니메이션](09-animation/01-basic-animation.md) — withAnimation, 암시적/명시적 애니메이션, transition
- [02. 고급 애니메이션](09-animation/02-advanced-animation.md) — spring, keyframe, PhaseAnimator, 타이밍 커브
- [03. 제스처](09-animation/03-gestures.md) — TapGesture, DragGesture, MagnifyGesture, 제스처 조합
- [04. Shape과 Canvas](09-animation/04-shapes-canvas.md) — Path, 커스텀 Shape, Canvas 2D 렌더링
- [05. 전환 효과와 매칭](09-animation/05-transitions.md) — matchedGeometryEffect, scrollTransition, 화면 전환

### Ch10. 시스템 프레임워크 활용
- [01. 이미지와 카메라](10-system-frameworks/01-image-camera.md) — AsyncImage, PhotosPicker, 카메라 캡처
- [02. 지도와 위치](10-system-frameworks/02-mapkit-location.md) — Map 뷰, CoreLocation, 위치 권한 처리
- [03. 알림](10-system-frameworks/03-notifications.md) — 로컬 알림, 예약 알림, 알림 액션과 카테고리
- [04. 공유와 딥링크](10-system-frameworks/04-sharing-deeplink.md) — ShareLink, Universal Links, URL Scheme, Transferable

### Ch11. 고급 SwiftUI
- [01. Custom Layout](11-advanced-swiftui/01-custom-layout.md) — Layout 프로토콜, 플로우 레이아웃, 적응형 배치
- [02. ViewBuilder와 제네릭 뷰](11-advanced-swiftui/02-viewbuilder.md) — @ViewBuilder, 커스텀 컨테이너, Result Builder
- [03. PreferenceKey와 GeometryReader](11-advanced-swiftui/03-preference-geometry.md) — 자식→부모 데이터 전달, 레이아웃 측정, 스크롤 감지
- [04. UIKit 브릿지](11-advanced-swiftui/04-uikit-bridge.md) — UIViewRepresentable, UIViewControllerRepresentable, Coordinator

---

## Part 5: 출시와 확장 (실무~전문가)

### Ch12. 테스트와 품질
- [01. Unit Test](12-testing-quality/01-unit-test.md) — XCTest 기초, 테스트 작성법, Mocking
- [02. Swift Testing 프레임워크](12-testing-quality/02-swift-testing.md) — #expect, @Test, @Suite, 차세대 테스트
- [03. UI Test](12-testing-quality/03-ui-test.md) — XCUITest, 자동화 시나리오, 접근성 ID 활용
- [04. 접근성과 국제화](12-testing-quality/04-accessibility.md) — VoiceOver, Dynamic Type, String Catalogs, 현지화

### Ch13. 성능과 최적화
- [01. Swift Concurrency 심화](13-performance/01-concurrency-deep.md) — Actor, Sendable, TaskGroup, AsyncStream
- [02. SwiftUI 렌더링 최적화](13-performance/02-swiftui-optimization.md) — 불필요한 뷰 업데이트 방지, body 최적화, Equatable
- [03. 메모리 관리와 ARC](13-performance/03-memory-arc.md) — ARC 동작 원리, 순환 참조, weak/unowned, 메모리 릭 탐지
- [04. Instruments 프로파일링](13-performance/04-instruments.md) — Time Profiler, Allocations, Leaks, 앱 시작 시간 분석

### Ch14. 앱스토어 출시
- [01. 앱 아이콘과 스크린샷](14-appstore/01-app-icon-assets.md) — Icon Composer, 앱 미리보기, 스크린샷 제작
- [02. 인증서와 프로비저닝](14-appstore/02-certificates.md) — 개발자 계정, Certificate, Provisioning Profile, Code Signing
- [03. App Store Connect](14-appstore/03-app-store-connect.md) — 앱 등록, 메타데이터, 가격 및 지역 설정
- [04. 심사 가이드라인](14-appstore/04-review-guidelines.md) — 흔한 리젝 사유, 심사 대응 전략, 이의 제기
- [05. 출시 후 운영](14-appstore/05-post-launch.md) — App Analytics, 크래시 리포트, ASO, 업데이트 전략

### Ch15. 앱 확장 기능
- [01. WidgetKit](15-app-extensions/01-widgetkit.md) — 홈 화면 위젯, Timeline Provider, 인터랙티브 위젯
- [02. Live Activities와 Dynamic Island](15-app-extensions/02-live-activities.md) — 실시간 정보 표시, ActivityKit, 잠금화면 업데이트
- [03. App Intents와 Shortcuts](15-app-extensions/03-app-intents.md) — Siri 통합, 단축어, 음성 명령 지원
- [04. In-App Purchase](15-app-extensions/04-storekit.md) — StoreKit 2, 구독 모델, 결제 처리와 영수증 검증

### Ch16. 최신 기술과 트렌드
- [01. Swift 6와 Concurrency 안전](16-trends/01-swift6.md) — Strict Concurrency, Sendable 준수, 마이그레이션 전략
- [02. visionOS와 공간 컴퓨팅](16-trends/02-visionos.md) — RealityKit, 3D 콘텐츠, 공간 UI 기초
- [03. AI와 머신러닝 통합](16-trends/03-ai-ml.md) — Core ML, Create ML, Foundation Models, Apple Intelligence
- [04. Swift 생태계 전망](16-trends/04-ecosystem.md) — SPM, 서버 사이드 Swift, 멀티플랫폼, Swift Evolution

---

## 부록

### Resources
- [Swift 공식 문서와 WWDC 세션](resources/essential-papers.md) — 필수 공식 자료와 WWDC 세션 목록
- [챕터별 예제 프로젝트](resources/datasets.md) — 실습용 프로젝트 모음
- [개발 도구와 라이브러리](resources/tools.md) — 유용한 서드파티 도구

---

## 학습 로드맵

**Part 1** Swift 언어 기초 (Ch1-2, 입문) → **Part 2** SwiftUI 기본 (Ch3-5, 초급) → **Part 3** 실전 앱 개발 (Ch6-8, 중급) → **Part 4** 사용자 경험 (Ch9-11, 중상급) → **Part 5** 출시와 확장 (Ch12-16, 실무~전문가)

| 단계 | 챕터 | 핵심 목표 | 난이도 |
|------|------|----------|--------|
| **Part 1** | Ch1-2 | Swift 언어 문법과 타입 시스템 익히기 | 입문 |
| **Part 2** | Ch3-5 | SwiftUI로 화면 만들고 상태 관리하기 | 초급 |
| **Part 3** | Ch6-8 | 데이터 저장, 네트워킹, 아키텍처 설계 | 중급 |
| **Part 4** | Ch9-11 | 애니메이션, 시스템 프레임워크, 고급 SwiftUI | 중상급 |
| **Part 5** | Ch12-13 | 테스트와 성능 최적화로 품질 확보 | 실무 |
| **Part 5** | **Ch14** | **앱스토어 출시 (크리티컬 골)** | **실무** |
| **Part 5** | Ch15-16 | 앱 확장 기능과 최신 트렌드 (보너스) | 전문가 |

> **Ch14(앱스토어 출시)가 이 튜토리얼의 최종 목표입니다.**
> Ch1→14가 앱 런칭까지의 크리티컬 패스이며, Ch15-16은 출시 후 앱을 확장하는 보너스 챕터입니다.

## 기술 스택

- **언어**: Swift 6+
- **UI 프레임워크**: SwiftUI (iOS 26+, Liquid Glass)
- **데이터**: SwiftData, CloudKit
- **동시성**: Swift Concurrency (async/await, Actor)
- **IDE**: Xcode 26+
- **테스트**: Swift Testing, XCTest
- **배포**: App Store Connect, Xcode Cloud

## 기여하기

오타, 오류, 개선 사항은 Issue나 PR로 알려주세요.

## 라이선스

MIT License
