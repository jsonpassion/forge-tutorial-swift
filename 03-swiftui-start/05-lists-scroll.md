# 리스트와 스크롤

> List, ForEach, ScrollView, LazyVStack

## 개요

앱에서 가장 많이 보이는 화면이 뭘까요? 바로 **목록**입니다. 설정 앱, 메일 앱, 메시지 앱, 음악 앱 — 모두 스크롤 가능한 리스트가 핵심이죠. 이 섹션에서는 SwiftUI의 List, ForEach, ScrollView를 배워서 다양한 형태의 목록 화면을 만들어봅니다.

**선수 지식**: [04. 레이아웃 시스템](./04-layout.md)에서 배운 Stack, Spacer, padding, frame
**학습 목표**:
- List와 ForEach로 동적 목록을 만들기
- Identifiable 프로토콜의 역할 이해하기
- ScrollView와 LazyVStack/LazyHStack으로 커스텀 스크롤 뷰 만들기
- 스와이프, 당겨서 새로고침, 검색 같은 리스트 기능 구현하기

## 왜 알아야 할까?

iPhone 사용 시간의 절반 이상은 **스크롤**하는 데 쓴다고 해도 과언이 아닙니다. 연락처 목록, 쇼핑 상품 목록, 채팅 메시지 목록... 앱의 핵심 데이터를 사용자에게 보여주는 가장 기본적인 방법이 리스트예요. 여기서 제대로 배워두면 어떤 앱이든 데이터 목록을 자신있게 구현할 수 있습니다.

## 핵심 개념

### 개념 1: ForEach — 반복적으로 뷰 생성하기

> 💡 **비유**: ForEach는 **복사기**입니다. 원본 템플릿을 하나 만들어놓으면, 데이터 개수만큼 자동으로 복사해서 화면에 나열해줍니다. 데이터가 10개면 10개, 100개면 100개의 뷰를 만들어요.

ForEach는 컬렉션의 각 항목에 대해 뷰를 생성합니다. [Ch1에서 배운 배열](../01-swift-basics/03-collections.md)의 각 요소를 뷰로 변환하는 역할이죠.

```swift
import SwiftUI

struct ForEachDemoView: View {
    // 과일 목록 데이터
    let fruits = ["사과", "바나나", "딸기", "포도", "오렌지"]

    var body: some View {
        VStack(alignment: .leading, spacing: 12) {
            Text("과일 목록")
                .font(.title2)
                .fontWeight(.bold)

            // ForEach로 배열의 각 요소를 뷰로 변환
            ForEach(fruits, id: \.self) { fruit in
                HStack {
                    Image(systemName: "leaf.fill")
                        .foregroundStyle(.green)
                    Text(fruit)
                        .font(.body)
                }
            }
        }
        .padding()
    }
}

#Preview {
    ForEachDemoView()
}
```

여기서 `id: \.self`가 보이죠? 이것은 "각 항목을 구별하는 기준으로 값 자체를 사용해"라는 뜻입니다. SwiftUI는 목록이 변경될 때 **어떤 항목이 추가/삭제/이동되었는지** 알아야 하거든요. 그래서 고유 식별자(ID)가 필요합니다.

### 개념 2: Identifiable 프로토콜

> 💡 **비유**: Identifiable은 **주민등록번호**와 같습니다. 이름이 같은 사람이 있어도 주민번호로 구분할 수 있듯, 데이터 모델에 고유한 `id`를 부여해서 SwiftUI가 각 항목을 정확히 구별할 수 있게 해줍니다.

문자열 같은 단순 타입은 `id: \.self`로 충분하지만, 커스텀 타입은 `Identifiable` 프로토콜을 따르는 것이 좋습니다.

```swift
// Identifiable 프로토콜: id 프로퍼티를 가져야 함
struct TodoItem: Identifiable {
    let id = UUID()     // 자동으로 고유한 ID 생성
    var title: String
    var isCompleted: Bool
}

struct TodoListView: View {
    @State private var todos = [
        TodoItem(title: "SwiftUI 공부하기", isCompleted: true),
        TodoItem(title: "프로젝트 만들기", isCompleted: false),
        TodoItem(title: "앱스토어 출시하기", isCompleted: false),
        TodoItem(title: "코드 리뷰하기", isCompleted: false),
        TodoItem(title: "블로그 글 쓰기", isCompleted: true)
    ]

    var body: some View {
        VStack(alignment: .leading, spacing: 8) {
            Text("할 일 목록")
                .font(.title2)
                .fontWeight(.bold)
                .padding(.horizontal)

            // Identifiable을 따르면 id: 파라미터 생략 가능!
            ForEach(todos) { todo in
                HStack {
                    Image(systemName: todo.isCompleted
                          ? "checkmark.circle.fill"
                          : "circle")
                        .foregroundStyle(todo.isCompleted ? .green : .gray)

                    Text(todo.title)
                        .strikethrough(todo.isCompleted)
                        .foregroundStyle(todo.isCompleted ? .secondary : .primary)
                }
                .padding(.horizontal)
                .padding(.vertical, 4)
            }
        }
        .padding(.vertical)
    }
}

#Preview {
    TodoListView()
}
```

> ⚠️ **흔한 오해**: "배열의 인덱스를 id로 쓰면 된다" — 절대 하지 마세요! 항목이 추가/삭제/재정렬되면 인덱스가 바뀌어서 SwiftUI가 혼란에 빠집니다. 애니메이션이 이상하게 작동하고, 잘못된 항목이 업데이트될 수 있어요. 항상 **고유한 식별자**를 사용하세요.

### 개념 3: List — 시스템 스타일 목록

> 💡 **비유**: List는 **설정 앱**의 화면과 같습니다. 행 사이에 구분선이 있고, 섹션으로 그룹화할 수 있고, 스크롤도 자동이에요. Apple이 미리 디자인한 깔끔한 목록 양식을 그대로 쓸 수 있습니다.

List는 ForEach에 스크롤, 구분선, 섹션 헤더 등의 기능을 추가한 고급 컴포넌트입니다.

```swift
struct ContactListView: View {
    let contacts = [
        Contact(name: "김민수", phone: "010-1234-5678", group: "가족"),
        Contact(name: "이영희", phone: "010-2345-6789", group: "가족"),
        Contact(name: "박철수", phone: "010-3456-7890", group: "친구"),
        Contact(name: "정미영", phone: "010-4567-8901", group: "친구"),
        Contact(name: "최동욱", phone: "010-5678-9012", group: "직장")
    ]

    var body: some View {
        List {
            // Section으로 그룹화
            ForEach(groupedContacts.keys.sorted(), id: \.self) { group in
                Section(group) {
                    ForEach(groupedContacts[group] ?? []) { contact in
                        HStack {
                            // 프로필 아이콘
                            Image(systemName: "person.circle.fill")
                                .font(.title2)
                                .foregroundStyle(.blue)

                            VStack(alignment: .leading, spacing: 2) {
                                Text(contact.name)
                                    .font(.headline)
                                Text(contact.phone)
                                    .font(.caption)
                                    .foregroundStyle(.secondary)
                            }
                        }
                    }
                }
            }
        }
    }

    // 그룹별로 분류
    var groupedContacts: [String: [Contact]] {
        Dictionary(grouping: contacts) { $0.group }
    }
}

struct Contact: Identifiable {
    let id = UUID()
    let name: String
    let phone: String
    let group: String
}

#Preview {
    ContactListView()
}
```

### 개념 4: List 스타일과 기능

List는 다양한 스타일과 인터랙션 기능을 제공합니다.

```swift
struct ListFeaturesView: View {
    @State private var items = ["항목 1", "항목 2", "항목 3", "항목 4", "항목 5"]
    @State private var searchText = ""

    var body: some View {
        NavigationStack {
            List {
                ForEach(filteredItems, id: \.self) { item in
                    Text(item)
                        // 스와이프 액션
                        .swipeActions(edge: .trailing) {
                            Button(role: .destructive) {
                                if let index = items.firstIndex(of: item) {
                                    items.remove(at: index)
                                }
                            } label: {
                                Label("삭제", systemImage: "trash")
                            }
                        }
                        .swipeActions(edge: .leading) {
                            Button {
                                // 즐겨찾기 로직
                            } label: {
                                Label("즐겨찾기", systemImage: "star")
                            }
                            .tint(.yellow)
                        }
                }
                // 삭제와 이동 지원
                .onDelete { indexSet in
                    items.remove(atOffsets: indexSet)
                }
                .onMove { source, destination in
                    items.move(fromOffsets: source, toOffset: destination)
                }
            }
            .navigationTitle("내 목록")
            // 검색 기능
            .searchable(text: $searchText, prompt: "검색")
            // 당겨서 새로고침
            .refreshable {
                // 데이터를 새로 불러오는 로직
                try? await Task.sleep(for: .seconds(1))
            }
            .toolbar {
                EditButton()  // 편집 모드 토글
            }
        }
    }

    var filteredItems: [String] {
        if searchText.isEmpty {
            return items
        }
        return items.filter { $0.localizedCaseInsensitiveContains(searchText) }
    }
}

#Preview {
    ListFeaturesView()
}
```

List의 스타일도 바꿀 수 있습니다:

| 스타일 | 설명 | 사용 예시 |
|--------|------|----------|
| `.automatic` | 기본 (플랫폼 자동 선택) | 대부분의 경우 |
| `.plain` | 구분선만 있는 깔끔한 스타일 | 채팅 목록 |
| `.insetGrouped` | 둥근 그룹 스타일 | 설정 화면 |
| `.sidebar` | 사이드바 스타일 | iPad/Mac 사이드바 |

### 개념 5: ScrollView와 LazyVStack

> 💡 **비유**: List가 **정해진 양식의 보고서**라면, ScrollView + LazyVStack은 **자유 형식의 스크랩북**입니다. List의 기본 스타일에 얽매이지 않고 완전히 자유로운 스크롤 화면을 만들 수 있어요.

List는 편리하지만 기본 스타일(구분선, 배경 등)이 강제됩니다. 완전히 커스텀한 스크롤 화면이 필요하면 ScrollView를 사용하세요.

```swift
struct ScrollViewDemoView: View {
    let items = Array(1...50)

    var body: some View {
        ScrollView {
            // LazyVStack: 화면에 보이는 뷰만 생성 (성능 최적화)
            LazyVStack(spacing: 12) {
                ForEach(items, id: \.self) { number in
                    HStack {
                        // 번호 뱃지
                        Text("\(number)")
                            .font(.headline)
                            .foregroundStyle(.white)
                            .frame(width: 40, height: 40)
                            .background(.blue)
                            .clipShape(Circle())

                        VStack(alignment: .leading, spacing: 2) {
                            Text("항목 #\(number)")
                                .font(.headline)
                            Text("LazyVStack으로 효율적으로 로드됩니다")
                                .font(.caption)
                                .foregroundStyle(.secondary)
                        }

                        Spacer()
                    }
                    .padding(.horizontal)
                    .padding(.vertical, 8)
                }
            }
        }
    }
}

#Preview {
    ScrollViewDemoView()
}
```

> 🔥 **실무 팁**: ScrollView 안에서는 반드시 **LazyVStack** 또는 **LazyHStack**을 사용하세요! 일반 VStack은 모든 뷰를 한 번에 생성하지만, Lazy 버전은 화면에 보이는 것만 생성합니다. 아이템이 수십 개 이상이면 성능 차이가 확연합니다.

### 개념 6: 가로 스크롤과 그리드

```swift
struct HorizontalScrollView: View {
    let categories = ["추천", "인기", "신규", "세일", "이벤트", "베스트", "한정판"]

    var body: some View {
        VStack(alignment: .leading, spacing: 20) {
            // 가로 스크롤
            Text("카테고리")
                .font(.headline)
                .padding(.horizontal)

            ScrollView(.horizontal, showsIndicators: false) {
                LazyHStack(spacing: 12) {
                    ForEach(categories, id: \.self) { category in
                        Text(category)
                            .font(.subheadline)
                            .padding(.horizontal, 16)
                            .padding(.vertical, 8)
                            .background(.blue.opacity(0.1))
                            .clipShape(Capsule())
                    }
                }
                .padding(.horizontal)
            }

            // 그리드 레이아웃
            Text("상품 목록")
                .font(.headline)
                .padding(.horizontal)

            ScrollView {
                // LazyVGrid: 세로 스크롤 그리드
                LazyVGrid(
                    columns: [
                        GridItem(.flexible()),  // 균등 분할
                        GridItem(.flexible())   // 2열 그리드
                    ],
                    spacing: 16
                ) {
                    ForEach(1...10, id: \.self) { index in
                        VStack(spacing: 8) {
                            // 상품 이미지 (SF Symbol로 대체)
                            Image(systemName: "bag.fill")
                                .font(.system(size: 40))
                                .foregroundStyle(.blue)
                                .frame(height: 100)
                                .frame(maxWidth: .infinity)
                                .background(.gray.opacity(0.1))
                                .clipShape(RoundedRectangle(cornerRadius: 12))

                            Text("상품 #\(index)")
                                .font(.subheadline)
                                .fontWeight(.medium)

                            Text("₩\(index * 10000)")
                                .font(.caption)
                                .foregroundStyle(.secondary)
                        }
                    }
                }
                .padding(.horizontal)
            }
        }
    }
}

#Preview {
    HorizontalScrollView()
}
```

## 실습: 메모 앱 만들기

지금까지 배운 List, ForEach, 스와이프, 검색을 모두 활용해서 간단한 메모 앱을 만들어봅시다.

```swift
import SwiftUI

struct Memo: Identifiable {
    let id = UUID()
    var title: String
    var content: String
    var date: Date
    var isPinned: Bool
}

struct MemoAppView: View {
    @State private var memos = [
        Memo(title: "SwiftUI 학습 계획", content: "Ch3 완료 후 Ch4 시작", date: Date(), isPinned: true),
        Memo(title: "장보기 목록", content: "우유, 빵, 계란, 과일", date: Date().addingTimeInterval(-3600), isPinned: false),
        Memo(title: "운동 루틴", content: "월수금 웨이트, 화목 유산소", date: Date().addingTimeInterval(-7200), isPinned: false),
        Memo(title: "프로젝트 아이디어", content: "날씨 앱, 할일 앱, 가계부 앱", date: Date().addingTimeInterval(-86400), isPinned: true),
        Memo(title: "읽을 책", content: "Clean Code, Swift Programming", date: Date().addingTimeInterval(-172800), isPinned: false)
    ]
    @State private var searchText = ""

    var body: some View {
        NavigationStack {
            List {
                // 고정된 메모 섹션
                if !pinnedMemos.isEmpty {
                    Section("고정된 메모") {
                        ForEach(pinnedMemos) { memo in
                            MemoRowView(memo: memo)
                        }
                    }
                }

                // 일반 메모 섹션
                Section("모든 메모") {
                    ForEach(filteredMemos) { memo in
                        MemoRowView(memo: memo)
                            .swipeActions(edge: .trailing) {
                                Button(role: .destructive) {
                                    deleteMemo(memo)
                                } label: {
                                    Label("삭제", systemImage: "trash")
                                }
                            }
                            .swipeActions(edge: .leading) {
                                Button {
                                    togglePin(memo)
                                } label: {
                                    Label(
                                        memo.isPinned ? "고정 해제" : "고정",
                                        systemImage: memo.isPinned ? "pin.slash" : "pin"
                                    )
                                }
                                .tint(.orange)
                            }
                    }
                }
            }
            .navigationTitle("메모")
            .searchable(text: $searchText, prompt: "메모 검색")
            .overlay {
                if filteredMemos.isEmpty && pinnedMemos.isEmpty {
                    ContentUnavailableView(
                        "메모 없음",
                        systemImage: "note.text",
                        description: Text("검색 결과가 없습니다")
                    )
                }
            }
        }
    }

    // 고정된 메모 필터
    var pinnedMemos: [Memo] {
        memos.filter { $0.isPinned }
            .filter { searchText.isEmpty || $0.title.localizedCaseInsensitiveContains(searchText) }
    }

    // 일반 메모 필터
    var filteredMemos: [Memo] {
        memos.filter { !$0.isPinned }
            .filter { searchText.isEmpty || $0.title.localizedCaseInsensitiveContains(searchText) }
    }

    // 메모 삭제
    func deleteMemo(_ memo: Memo) {
        memos.removeAll { $0.id == memo.id }
    }

    // 고정 토글
    func togglePin(_ memo: Memo) {
        if let index = memos.firstIndex(where: { $0.id == memo.id }) {
            memos[index].isPinned.toggle()
        }
    }
}

// 메모 행 뷰 (별도 분리)
struct MemoRowView: View {
    let memo: Memo

    var body: some View {
        VStack(alignment: .leading, spacing: 4) {
            HStack {
                Text(memo.title)
                    .font(.headline)

                if memo.isPinned {
                    Image(systemName: "pin.fill")
                        .font(.caption2)
                        .foregroundStyle(.orange)
                }
            }

            Text(memo.content)
                .font(.subheadline)
                .foregroundStyle(.secondary)
                .lineLimit(1)

            Text(memo.date, style: .relative)
                .font(.caption2)
                .foregroundStyle(.tertiary)
        }
        .padding(.vertical, 2)
    }
}

#Preview {
    MemoAppView()
}
```

## 더 깊이 알아보기

### List vs ScrollView+LazyVStack, 어떤 걸 써야 할까?

이 질문은 SwiftUI 개발자들 사이에서 가장 자주 나오는 질문 중 하나입니다.

| 기능 | List | ScrollView + LazyVStack |
|------|------|------------------------|
| 구분선 | 자동 | 직접 추가 |
| 스와이프 액션 | `.swipeActions` 지원 | 직접 구현 |
| 당겨서 새로고침 | `.refreshable` 지원 | `.refreshable` 지원 |
| 검색 | `.searchable` 지원 | `.searchable` 지원 |
| 편집 모드 | `.onDelete`, `.onMove` | 직접 구현 |
| 커스텀 스타일 | 제한적 | 완전 자유 |
| 섹션 헤더/푸터 | Section 뷰 | 직접 구현 |
| 배경 제거 | `.listRowBackground` | 기본 투명 |

**결론**: 설정 화면처럼 **시스템 스타일**이 어울리면 List, SNS 피드처럼 **커스텀 디자인**이 필요하면 ScrollView + LazyVStack을 쓰세요.

### ContentUnavailableView — 빈 상태 처리

iOS 17에서 추가된 `ContentUnavailableView`는 목록이 비었을 때 보여주는 표준 화면입니다. Apple의 기본 앱(파일, 사진 등)에서 "결과 없음" 화면에 사용되는 것과 같은 스타일이에요. 검색 결과가 없거나, 데이터가 아직 없을 때 사용하면 전문적인 느낌을 줄 수 있습니다.

## 흔한 오해와 팁

> ⚠️ **흔한 오해**: "VStack으로 스크롤 가능한 목록을 만들 수 있다" — VStack은 스크롤을 지원하지 않습니다! 반드시 `ScrollView` 안에 넣거나 `List`를 사용해야 스크롤이 됩니다. 또한 VStack은 모든 항목을 한 번에 로드하므로, 많은 데이터에는 LazyVStack을 사용하세요.

> 🔥 **실무 팁**: `ForEach`에서 `id: \.self`는 String, Int 같은 단순 타입에만 쓰세요. 커스텀 모델은 반드시 `Identifiable`을 채택하는 것이 안전합니다. UUID()를 id로 쓰면 매번 새로운 ID가 생성되므로, 서버에서 받은 데이터라면 서버의 고유 ID를 사용하세요.

> 💡 **알고 계셨나요?**: `Text(date, style: .relative)`처럼 Text에 Date를 직접 전달하면 "2분 전", "3시간 전"처럼 **실시간으로 업데이트되는 상대 시간**을 표시할 수 있습니다. 타이머를 따로 만들 필요가 없어요!

## 핵심 정리

| 개념 | 설명 |
|------|------|
| ForEach | 컬렉션의 각 요소를 뷰로 변환. `id`로 항목을 구별 |
| Identifiable | `id` 프로퍼티를 요구하는 프로토콜. ForEach에서 `id:` 생략 가능 |
| List | 시스템 스타일 목록. 구분선, 섹션, 스와이프 등 기본 제공 |
| Section | List 안에서 항목을 그룹으로 묶고 헤더/푸터를 표시 |
| .swipeActions | 셀을 스와이프할 때 나오는 버튼 (삭제, 즐겨찾기 등) |
| .searchable | List에 검색 바를 추가 |
| .refreshable | 당겨서 새로고침 기능 추가 (async 필요) |
| ScrollView | 자유 형식의 스크롤 뷰. 가로/세로 모두 가능 |
| LazyVStack | 화면에 보이는 뷰만 생성하는 효율적인 세로 스택 |
| LazyVGrid | 세로 스크롤 그리드 레이아웃. GridItem으로 열 구성 |

## 다음 섹션 미리보기

Ch3에서 SwiftUI의 기본 뷰와 레이아웃을 배웠습니다! 이제 다음 [Ch4. 화면 구성과 네비게이션](../04-navigation-design/01-navigation-stack.md)에서 **여러 화면으로 구성된 앱**을 만드는 법을 배웁니다. NavigationStack으로 화면 전환하고, TabView로 탭 바를 만들고, 모달을 띄우는 방법까지 — 진짜 앱다운 앱을 만들어볼 거예요!

## 참고 자료

- [Apple List 공식 문서](https://developer.apple.com/documentation/swiftui/list) - List 뷰의 모든 API
- [Apple ForEach 공식 문서](https://developer.apple.com/documentation/swiftui/foreach) - ForEach 사용법
- [Apple ScrollView 공식 문서](https://developer.apple.com/documentation/swiftui/scrollview) - ScrollView API 레퍼런스
- [WWDC 2023 - Beyond scroll views](https://developer.apple.com/videos/play/wwdc2023/10159/) - 고급 스크롤 기법
- [WWDC 2022 - SwiftUI on iPad: Organize your interface](https://developer.apple.com/videos/play/wwdc2022/10058/) - List와 네비게이션 패턴
- [Apple Human Interface Guidelines - Lists and tables](https://developer.apple.com/design/human-interface-guidelines/lists-and-tables) - 리스트 디자인 가이드
