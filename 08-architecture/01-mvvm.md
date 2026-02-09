# MVVM 패턴

> Model-View-ViewModel, SwiftUI에서의 MVVM 구현

## 개요

앱이 커지면 View 안에 네트워크 호출, 데이터 변환, 비즈니스 로직이 뒤섞이기 시작합니다. 이러면 코드를 읽기도, 테스트하기도 어려워지죠. MVVM 패턴은 이 문제를 **역할 분리**로 해결합니다. SwiftUI의 선언적 특성과 만나면 UIKit 시절보다 훨씬 자연스러운 구현이 가능해요.

**선수 지식**: [Ch7. 실전 API 프로젝트](../07-networking/05-api-project.md)에서 배운 네트워크 앱 구현, [@Observable 매크로](../05-state-management/02-observable.md)
**학습 목표**:
- MVVM의 세 구성 요소(Model, View, ViewModel)의 역할 이해
- SwiftUI에서 @Observable로 ViewModel 구현하기
- View와 ViewModel 분리의 실질적 이점 체감하기
- MVVM에 대한 다양한 관점과 적절한 사용 시점 판단하기

## 왜 알아야 할까?

100줄짜리 View를 상상해보세요. API를 호출하고, 응답을 파싱하고, 날짜를 포맷팅하고, 에러를 처리하고, UI까지 그립니다. 여기서 "가격 계산 로직이 맞는지" 테스트하려면 어떻게 해야 할까요? View를 통째로 띄워야 합니다. 앱을 빌드하고, 시뮬레이터에서 화면을 열고, 버튼을 눌러봐야 하죠.

MVVM을 적용하면 이야기가 달라집니다. 비즈니스 로직을 ViewModel에 분리하면, **UI 없이도 순수한 Swift 코드로 테스트**할 수 있거든요. 또한 디자이너가 UI를 변경해도 로직은 건드릴 필요가 없고, 반대로 API가 바뀌어도 View는 그대로입니다.

## 핵심 개념

### 개념 1: MVVM의 세 가지 역할

> 💡 **비유**: MVVM은 **레스토랑 운영**과 같습니다. Model은 식재료 창고(원본 데이터), ViewModel은 주방장(데이터를 가공하고 비즈니스 로직을 처리), View는 서빙 직원(손님에게 보여주는 역할)입니다. 손님(사용자)은 주방에 직접 들어가지 않고, 서빙 직원을 통해 주문하고 음식을 받죠.

| 구성 요소 | 역할 | SwiftUI에서 |
|-----------|------|-------------|
| **Model** | 데이터 구조와 비즈니스 규칙 | `struct`, `@Model`, Codable 타입 |
| **View** | UI 렌더링과 사용자 입력 | `View` 프로토콜을 따르는 구조체 |
| **ViewModel** | View와 Model 사이의 중재자 | `@Observable` 클래스 |

데이터 흐름 방향을 정리하면 이렇습니다:

**사용자 액션** → **View** → **ViewModel** → **Model** → **ViewModel**(가공) → **View**(업데이트)

### 개념 2: SwiftUI에서 ViewModel 만들기

SwiftUI에서 ViewModel은 `@Observable` 매크로를 사용해 만듭니다. 이전의 `ObservableObject` + `@Published` 방식보다 훨씬 깔끔해졌어요.

```swift
import SwiftUI

// MARK: - Model
// 서버에서 받는 원본 데이터 구조
struct Article: Identifiable, Codable {
    let id: Int
    let title: String
    let body: String
    let userId: Int
}

// MARK: - ViewModel
// View에 필요한 데이터와 로직을 관리합니다
@Observable
@MainActor
class ArticleListViewModel {
    // View가 읽는 상태들
    var articles: [Article] = []
    var isLoading = false
    var errorMessage: String?

    // 비즈니스 로직: 기사 목록 가져오기
    func fetchArticles() async {
        isLoading = true
        errorMessage = nil

        do {
            let url = URL(string: "https://jsonplaceholder.typicode.com/posts")!
            let (data, _) = try await URLSession.shared.data(from: url)
            articles = try JSONDecoder().decode([Article].self, from: data)
        } catch {
            errorMessage = "기사를 불러올 수 없습니다: \(error.localizedDescription)"
        }

        isLoading = false
    }

    // 표시용 데이터 가공
    var articleCount: String {
        "\(articles.count)개의 기사"
    }
}
```

> ⚠️ **흔한 오해**: "ViewModel에 `@MainActor`를 붙이면 성능이 떨어지지 않나요?" — ViewModel은 UI 상태를 관리하므로 메인 스레드에서 동작해야 합니다. `async` 메서드 안의 네트워크 호출은 자동으로 백그라운드에서 실행되고, 결과만 메인 스레드로 돌아오기 때문에 성능 문제가 없습니다.

### 개념 3: View에서 ViewModel 연결하기

```swift
// MARK: - View
// UI만 담당합니다 — 로직은 ViewModel에 위임
struct ArticleListView: View {
    // ViewModel을 @State로 소유합니다
    @State private var viewModel = ArticleListViewModel()

    var body: some View {
        NavigationStack {
            Group {
                if viewModel.isLoading {
                    ProgressView("로딩 중...")
                } else if let error = viewModel.errorMessage {
                    ContentUnavailableView(
                        "오류 발생",
                        systemImage: "exclamationmark.triangle",
                        description: Text(error)
                    )
                } else {
                    articleList
                }
            }
            .navigationTitle("뉴스")
            .toolbar {
                ToolbarItem(placement: .topBarTrailing) {
                    Text(viewModel.articleCount)
                        .font(.caption)
                        .foregroundStyle(.secondary)
                }
            }
            .task {
                await viewModel.fetchArticles()
            }
        }
    }

    // 리스트 UI를 분리하여 body를 깔끔하게 유지
    private var articleList: some View {
        List(viewModel.articles) { article in
            VStack(alignment: .leading, spacing: 8) {
                Text(article.title)
                    .font(.headline)
                    .lineLimit(2)
                Text(article.body)
                    .font(.subheadline)
                    .foregroundStyle(.secondary)
                    .lineLimit(3)
            }
            .padding(.vertical, 4)
        }
    }
}

#Preview {
    ArticleListView()
}
```

핵심 포인트를 정리하면:

- **`@State private var viewModel`**: View가 ViewModel의 생명주기를 소유합니다
- **`.task { }`**: View가 나타날 때 ViewModel의 비즈니스 로직을 호출합니다
- **View는 "어떻게 보여줄지"만 결정**: 데이터 가져오기, 에러 처리 등은 모두 ViewModel에 위임합니다

### 개념 4: 테스트 가능성 — MVVM의 진짜 가치

MVVM의 핵심 이점은 **ViewModel을 독립적으로 테스트**할 수 있다는 것입니다.

```swift
import Testing

@Suite("ArticleListViewModel 테스트")
struct ArticleListViewModelTests {

    @Test("초기 상태 확인")
    @MainActor
    func initialState() {
        let vm = ArticleListViewModel()

        #expect(vm.articles.isEmpty)
        #expect(vm.isLoading == false)
        #expect(vm.errorMessage == nil)
        #expect(vm.articleCount == "0개의 기사")
    }
}
```

View를 띄우지 않고도 로직을 검증할 수 있죠. 이것이 MVVM이 존재하는 가장 큰 이유입니다.

## 실습: 직접 해보기

검색 기능이 있는 ViewModel을 만들어봅시다.

```swift
import SwiftUI

// MARK: - Model
struct User: Identifiable, Codable {
    let id: Int
    let name: String
    let email: String
    let phone: String
}

// MARK: - ViewModel
@Observable
@MainActor
class UserSearchViewModel {
    var users: [User] = []
    var searchText = ""
    var isLoading = false

    // 검색어에 따라 필터링된 결과를 반환
    var filteredUsers: [User] {
        guard !searchText.isEmpty else { return users }
        return users.filter { user in
            user.name.localizedCaseInsensitiveContains(searchText) ||
            user.email.localizedCaseInsensitiveContains(searchText)
        }
    }

    // 검색 결과 요약
    var resultSummary: String {
        if searchText.isEmpty {
            return "전체 \(users.count)명"
        }
        return "\(filteredUsers.count)명 검색됨"
    }

    func loadUsers() async {
        isLoading = true
        do {
            let url = URL(string: "https://jsonplaceholder.typicode.com/users")!
            let (data, _) = try await URLSession.shared.data(from: url)
            users = try JSONDecoder().decode([User].self, from: data)
        } catch {
            print("사용자 로드 실패: \(error)")
        }
        isLoading = false
    }
}

// MARK: - View
struct UserSearchView: View {
    @State private var viewModel = UserSearchViewModel()

    var body: some View {
        NavigationStack {
            List(viewModel.filteredUsers) { user in
                VStack(alignment: .leading) {
                    Text(user.name)
                        .font(.headline)
                    Text(user.email)
                        .font(.caption)
                        .foregroundStyle(.secondary)
                }
            }
            .searchable(text: $viewModel.searchText, prompt: "이름 또는 이메일 검색")
            .navigationTitle("사용자 검색")
            .toolbar {
                ToolbarItem(placement: .topBarTrailing) {
                    Text(viewModel.resultSummary)
                        .font(.caption)
                }
            }
            .overlay {
                if viewModel.isLoading {
                    ProgressView()
                }
            }
            .task {
                await viewModel.loadUsers()
            }
        }
    }
}

#Preview {
    UserSearchView()
}
```

## 더 깊이 알아보기

### MVVM의 역사

MVVM은 2005년 Microsoft의 John Gossman이 WPF(Windows Presentation Foundation)를 위해 제안한 패턴입니다. 당시 WPF의 데이터 바인딩 시스템에 최적화된 패턴이었는데, 놀랍게도 15년 후 SwiftUI의 선언적 UI와 만나며 다시 주목받게 됩니다.

사실 Apple은 공식적으로 "MVVM을 쓰라"고 말한 적이 없어요. WWDC 2020의 "Data Essentials in SwiftUI" 세션에서는 "Source of Truth"와 "데이터 흐름"을 강조했을 뿐이죠. SwiftUI 커뮤니티에서는 "MVVM이 SwiftUI에 꼭 필요한가?"라는 논쟁이 계속되고 있습니다.

### "MVVM이 필요없다"는 주장에 대해

일부 개발자들은 SwiftUI 자체가 이미 View와 State를 분리하므로 별도 ViewModel이 필요없다고 주장합니다. 간단한 화면에서는 맞는 말이에요. 하지만 **비즈니스 로직이 복잡하거나, 테스트가 중요하거나, 팀이 크다면** ViewModel의 가치는 분명합니다.

> 🔥 **실무 팁**: 정답은 "항상 MVVM"이 아니라 **"필요할 때 MVVM"**입니다. 단순한 설정 화면에 ViewModel을 만드는 건 과잉 설계예요. 반면 복잡한 검색 화면, 폼 유효성 검증, 다중 API 호출이 필요한 화면에서는 ViewModel이 빛을 발합니다.

## 흔한 오해와 팁

> ⚠️ **흔한 오해**: "ViewModel은 반드시 하나의 View와 1:1로 대응해야 한다" — 꼭 그렇지 않습니다. 간단한 화면은 ViewModel 없이도 되고, 복잡한 화면은 여러 개의 작은 ViewModel로 나눌 수도 있어요.

> 💡 **알고 계셨나요?**: `@Observable`은 WWDC 2023에서 소개된 Observation 프레임워크의 핵심입니다. 이전의 `ObservableObject`와 달리 프로퍼티 하나하나를 세밀하게 추적하기 때문에, 변경되지 않은 프로퍼티를 참조하는 View는 다시 그려지지 않습니다.

> 🔥 **실무 팁**: ViewModel에서 `@Published`나 `ObservableObject`를 쓰고 있다면, `@Observable`로 마이그레이션하세요. 코드가 줄어들고, 성능도 좋아집니다.

## 핵심 정리

| 개념 | 설명 |
|------|------|
| Model | 데이터 구조와 비즈니스 규칙을 정의하는 타입 |
| View | SwiftUI View — UI 렌더링만 담당 |
| ViewModel | `@Observable` 클래스로 View와 Model 사이를 중재 |
| `@MainActor` | ViewModel에 붙여 UI 상태 업데이트의 스레드 안전성 보장 |
| `@State` + ViewModel | View가 ViewModel의 생명주기를 소유하는 패턴 |
| 테스트 가능성 | MVVM의 핵심 이점 — ViewModel을 UI 없이 단위 테스트 가능 |

## 다음 섹션 미리보기

ViewModel이 직접 네트워크를 호출하고 데이터를 관리하면, 데이터 소스가 바뀔 때마다 ViewModel도 수정해야 합니다. 다음 [02. Repository 패턴](./02-repository.md)에서는 데이터 소스를 **추상화**하여 ViewModel과 데이터 레이어를 분리하는 방법을 배웁니다.

## 참고 자료

- [Observation 프레임워크 - Apple 공식 문서](https://developer.apple.com/documentation/observation) — @Observable 매크로의 동작 원리
- [Data Essentials in SwiftUI - WWDC 2020](https://developer.apple.com/videos/play/wwdc2020/10040/) — SwiftUI 데이터 흐름의 근본 원칙
- [Discover Observation in SwiftUI - WWDC 2023](https://developer.apple.com/videos/play/wwdc2023/10149/) — @Observable 소개와 마이그레이션
- [Unit Test the Observation Framework - Jacob Bartlett](https://blog.jacobstechtavern.com/p/unit-test-the-observation-framework) — @Observable ViewModel 테스트 기법
