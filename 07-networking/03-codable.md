# Codable과 JSON 파싱

> JSON 디코딩/인코딩, CodingKeys, 중첩 구조 처리

## 개요

서버 API에서 받아오는 데이터는 대부분 **JSON** 형식입니다. 이 JSON을 Swift의 구조체나 클래스로 변환하는 작업을 "디코딩"이라고 하는데요, Swift의 `Codable` 프로토콜은 이 과정을 놀랍도록 간단하게 만들어줍니다. 복잡한 파싱 코드 없이, 구조체를 선언하기만 하면 자동으로 JSON 변환이 이루어지거든요.

**선수 지식**: [URLSession과 REST API](./02-urlsession.md)에서 배운 네트워크 요청 방법
**학습 목표**:
- `Codable` 프로토콜로 JSON을 Swift 모델로 변환하는 방법
- `CodingKeys`로 서버의 키 이름과 Swift 프로퍼티를 매핑하는 방법
- 중첩된 JSON 구조와 날짜/특수 타입을 처리하는 전략

## 왜 알아야 할까?

서버 API가 보내주는 데이터는 이렇게 생겼습니다:

```swift
// 서버에서 온 JSON 데이터 (문자열)
let json = """
{"user_name": "김스위프트", "created_at": "2025-01-15T09:30:00Z"}
"""
```

이걸 Swift에서 `user.userName`처럼 편하게 쓰려면 변환 과정이 필요합니다. 예전에는 `JSONSerialization`으로 딕셔너리를 만들고 일일이 타입 캐스팅을 해야 했는데, `Codable`이 등장한 후로는 구조체 하나만 선언하면 끝입니다. iOS 개발에서 가장 많이 쓰는 프로토콜 중 하나이니, 반드시 익혀두세요.

## 핵심 개념

### 개념 1: Codable이란? — 자동 번역기

> 💡 **비유**: `Codable`은 **자동 번역기**와 같습니다. JSON이라는 "외국어"를 Swift라는 "모국어"로 자동 번역해주죠. 구조체의 프로퍼티 이름과 타입만 맞춰주면, 번역기가 알아서 매핑합니다.

`Codable`은 사실 두 프로토콜의 조합입니다:

- **Decodable**: JSON → Swift (디코딩, 서버 응답 파싱)
- **Encodable**: Swift → JSON (인코딩, 서버에 데이터 전송)

```run:swift
// Codable = Decodable & Encodable
struct User: Codable {
    let id: Int
    let name: String
    let email: String
    let isActive: Bool
}

// JSON → Swift (디코딩)
let jsonData = """
{"id": 1, "name": "김스위프트", "email": "swift@example.com", "isActive": true}
""".data(using: .utf8)!

let user = try JSONDecoder().decode(User.self, from: jsonData)
print(user.name)  // "김스위프트"

// Swift → JSON (인코딩)
let encoded = try JSONEncoder().encode(user)
let jsonString = String(data: encoded, encoding: .utf8)!
print(jsonString)  // {"id":1,"name":"김스위프트","email":"swift@example.com","isActive":true}
```

```output
김스위프트
{"id":1,"name":"김스위프트","email":"swift@example.com","isActive":true}
```

> 💡 **알고 계셨나요?**: `Codable`은 Swift 4(2017년)에서 **SE-0166** 프로포절을 통해 도입되었습니다. Itai Ferber, Michael LeHew, Tony Parker가 설계했는데, 이들은 "Swift 네이티브한 직렬화 솔루션"을 목표로 했습니다. 그 전에는 Objective-C의 `NSCoding`이나 수동 JSON 파싱에 의존해야 했죠. `Codable` 덕분에 Swift는 JSON 처리가 가장 편한 언어 중 하나가 되었습니다.

### 개념 2: URLSession + Codable — 황금 조합

실제로 API에서 데이터를 가져와 디코딩하는 전체 흐름을 살펴봅시다:

```swift
// API 응답 모델
struct Post: Codable, Identifiable {
    let id: Int
    let title: String
    let body: String
    let userId: Int
}

// URLSession + Codable로 데이터 가져오기
func fetchPosts() async throws -> [Post] {
    let url = URL(string: "https://jsonplaceholder.typicode.com/posts?_limit=5")!

    // 1. 네트워크 요청
    let (data, response) = try await URLSession.shared.data(from: url)

    // 2. 상태 코드 확인
    guard let httpResponse = response as? HTTPURLResponse,
          httpResponse.statusCode == 200 else {
        throw URLError(.badServerResponse)
    }

    // 3. JSON 디코딩 — 이 한 줄이 핵심!
    let posts = try JSONDecoder().decode([Post].self, from: data)
    return posts
}
```

배열 디코딩도 간단합니다. `[Post].self`처럼 배열 타입을 전달하면 JSON 배열을 자동으로 파싱합니다.

### 개념 3: CodingKeys — 이름이 다를 때의 통역사

서버의 JSON 키 이름이 Swift 컨벤션과 다른 경우가 많습니다. 서버는 `snake_case`를, Swift는 `camelCase`를 쓰니까요:

```swift
// 서버 JSON: {"user_name": "김스위프트", "profile_image_url": "https://...", "created_at": "2025-01-15"}

// 방법 1: CodingKeys enum으로 직접 매핑
struct UserProfile: Codable {
    let userName: String
    let profileImageUrl: String
    let createdAt: String

    // JSON 키와 Swift 프로퍼티 이름 매핑
    enum CodingKeys: String, CodingKey {
        case userName = "user_name"
        case profileImageUrl = "profile_image_url"
        case createdAt = "created_at"
    }
}

// 방법 2: keyDecodingStrategy로 자동 변환 (더 간편!)
let decoder = JSONDecoder()
decoder.keyDecodingStrategy = .convertFromSnakeCase

// 이러면 CodingKeys 없이도 user_name → userName 자동 매핑
struct UserProfileSimple: Codable {
    let userName: String
    let profileImageUrl: String
    let createdAt: String
}

let user = try decoder.decode(UserProfileSimple.self, from: jsonData)
```

> 🔥 **실무 팁**: 서버가 일관되게 `snake_case`를 사용한다면, `CodingKeys`를 일일이 쓰는 것보다 `keyDecodingStrategy = .convertFromSnakeCase`가 훨씬 간편합니다. 하지만 서버의 키 이름이 불규칙하거나 특수한 경우(`ID`, `URL`, `iOS` 등)에는 `CodingKeys`를 직접 정의하는 게 안전합니다.

### 개념 4: 옵셔널과 기본값 — 없는 필드 처리

서버 응답에 특정 필드가 없거나 `null`인 경우를 처리하는 방법:

```run:swift
struct Article: Codable {
    let id: Int
    let title: String
    let subtitle: String?       // JSON에 없을 수 있는 필드 → 옵셔널
    let thumbnailUrl: String?   // null이 올 수 있는 필드 → 옵셔널
    let viewCount: Int          // 항상 있어야 하는 필드
}

// 이 JSON으로 디코딩 가능 (subtitle, thumbnailUrl이 없어도 OK)
let json = """
{"id": 1, "title": "Swift 6 새로운 기능", "viewCount": 1500}
""".data(using: .utf8)!

let article = try JSONDecoder().decode(Article.self, from: json)
print(article.subtitle)      // nil
print(article.thumbnailUrl)  // nil
print(article.viewCount)     // 1500
```

```output
nil
nil
1500
```

> ⚠️ **흔한 오해**: "JSON에 필드가 없으면 항상 에러가 난다" — 아닙니다! Swift 프로퍼티를 **옵셔널(`?`)**로 선언하면, JSON에 해당 키가 없거나 `null`이어도 정상적으로 디코딩됩니다. 반대로 옵셔널이 아닌 프로퍼티의 키가 JSON에 없으면 `DecodingError.keyNotFound` 에러가 발생합니다.

### 개념 5: 중첩 JSON 구조 처리

실제 API는 종종 중첩된 JSON을 반환합니다:

```swift
// 서버 응답 JSON
// {
//   "results": [
//     {"trackName": "Bad Guy", "artistName": "Billie Eilish", "artworkUrl100": "https://..."},
//     ...
//   ],
//   "resultCount": 50
// }

// 최상위 응답 래퍼
struct SearchResponse: Codable {
    let resultCount: Int
    let results: [Track]
}

// 개별 항목
struct Track: Codable, Identifiable {
    let trackId: Int
    let trackName: String
    let artistName: String
    let artworkUrl100: String
    let collectionName: String?  // 앨범명 — 없을 수 있음
    let previewUrl: String?      // 미리듣기 URL — 없을 수 있음

    var id: Int { trackId }
}

// 사용법
func searchMusic(term: String) async throws -> [Track] {
    var components = URLComponents(string: "https://itunes.apple.com/search")!
    components.queryItems = [
        URLQueryItem(name: "term", value: term),
        URLQueryItem(name: "media", value: "music"),
        URLQueryItem(name: "limit", value: "25")
    ]

    let (data, _) = try await URLSession.shared.data(from: components.url!)
    let response = try JSONDecoder().decode(SearchResponse.self, from: data)
    return response.results  // 중첩된 배열만 반환
}
```

### 개념 6: 날짜 디코딩 전략

JSON에서 날짜를 처리하는 것은 의외로 까다롭습니다. 서버마다 날짜 형식이 다르기 때문이죠:

```swift
struct Event: Codable {
    let name: String
    let startDate: Date
    let endDate: Date
}

// ISO 8601 형식 (가장 흔함): "2025-01-15T09:30:00Z"
let decoder = JSONDecoder()
decoder.dateDecodingStrategy = .iso8601

// 커스텀 형식: "2025/01/15"
let customDecoder = JSONDecoder()
let formatter = DateFormatter()
formatter.dateFormat = "yyyy/MM/dd"
customDecoder.dateDecodingStrategy = .formatted(formatter)

// Unix 타임스탬프: 1736928600
let timestampDecoder = JSONDecoder()
timestampDecoder.dateDecodingStrategy = .secondsSince1970
```

## 실습: iTunes 음악 검색기

URLSession + Codable을 활용한 음악 검색 앱을 만들어봅시다:

```swift
import SwiftUI

// API 응답 모델
struct ITunesResponse: Codable {
    let resultCount: Int
    let results: [Song]
}

struct Song: Codable, Identifiable {
    let trackId: Int
    let trackName: String
    let artistName: String
    let artworkUrl100: String
    let collectionName: String?

    var id: Int { trackId }
}

// 뷰 모델
@MainActor
@Observable
class MusicSearchViewModel {
    var songs: [Song] = []
    var isLoading = false
    var errorMessage: String?

    func search(term: String) async {
        guard !term.isEmpty else {
            songs = []
            return
        }

        isLoading = true
        errorMessage = nil
        defer { isLoading = false }

        do {
            // URL 구성
            var components = URLComponents(string: "https://itunes.apple.com/search")!
            components.queryItems = [
                URLQueryItem(name: "term", value: term),
                URLQueryItem(name: "media", value: "music"),
                URLQueryItem(name: "country", value: "kr"),
                URLQueryItem(name: "limit", value: "20")
            ]

            guard let url = components.url else { return }

            // 네트워크 요청 + JSON 디코딩
            let (data, _) = try await URLSession.shared.data(from: url)
            let response = try JSONDecoder().decode(ITunesResponse.self, from: data)
            songs = response.results
        } catch {
            errorMessage = "검색에 실패했습니다"
        }
    }
}

// 뷰
struct MusicSearchView: View {
    @State private var viewModel = MusicSearchViewModel()
    @State private var searchText = ""

    var body: some View {
        NavigationStack {
            List(viewModel.songs) { song in
                HStack(spacing: 12) {
                    // 앨범 아트 (AsyncImage로 비동기 로드)
                    AsyncImage(url: URL(string: song.artworkUrl100)) { image in
                        image
                            .resizable()
                            .aspectRatio(contentMode: .fill)
                    } placeholder: {
                        Color.gray.opacity(0.3)
                    }
                    .frame(width: 50, height: 50)
                    .clipShape(RoundedRectangle(cornerRadius: 8))

                    // 곡 정보
                    VStack(alignment: .leading, spacing: 2) {
                        Text(song.trackName)
                            .font(.headline)
                            .lineLimit(1)
                        Text(song.artistName)
                            .font(.subheadline)
                            .foregroundStyle(.secondary)
                        if let album = song.collectionName {
                            Text(album)
                                .font(.caption)
                                .foregroundStyle(.tertiary)
                        }
                    }
                }
            }
            .overlay {
                if viewModel.isLoading {
                    ProgressView()
                } else if viewModel.songs.isEmpty && !searchText.isEmpty {
                    ContentUnavailableView.search(text: searchText)
                }
            }
            .navigationTitle("음악 검색")
            .searchable(text: $searchText, prompt: "아티스트 또는 곡 검색")
        }
        .task(id: searchText) {
            // searchText가 바뀔 때마다 자동 재실행 + 이전 작업 취소
            try? await Task.sleep(for: .milliseconds(500))  // 디바운스
            guard !Task.isCancelled else { return }
            await viewModel.search(term: searchText)
        }
    }
}

#Preview {
    MusicSearchView()
}
```

## 더 깊이 알아보기

### Codable의 진화

`Codable`은 Swift 4(2017) 이후로 꾸준히 발전해왔습니다:

- **Swift 4.0** (2017): `Codable` 도입 (SE-0166)
- **Swift 5.5** (2021): enum의 associated values에 대한 자동 Codable 합성 (SE-0295)
- **Swift 5.6** (2022): `CodingKeyRepresentable` 프로토콜 (SE-0320) — Dictionary 키 처리 개선
- **Swift 6.0** (2024): Foundation의 `JSONDecoder`가 순수 Swift로 재구현되어 20~40% 성능 향상

Swift 6에서는 `JSONDecoder`가 완전히 Swift로 다시 작성되었는데, 이로 인해 대용량 JSON 파싱 시 메모리 할당과 문자열 처리 속도가 크게 개선되었습니다.

### 수동 디코딩이 필요한 경우

자동 합성이 안 되는 복잡한 JSON이라면 `init(from:)`을 직접 구현합니다:

```swift
// 서버 응답이 이런 형태라면:
// { "data": { "user": { "full_name": "김스위프트", "age": 25 } }, "status": "ok" }

struct UserInfo: Decodable {
    let fullName: String
    let age: Int

    enum OuterKeys: String, CodingKey {
        case data
    }

    enum DataKeys: String, CodingKey {
        case user
    }

    enum UserKeys: String, CodingKey {
        case fullName = "full_name"
        case age
    }

    // 중첩 구조를 평탄화하는 커스텀 디코딩
    init(from decoder: any Decoder) throws {
        let outer = try decoder.container(keyedBy: OuterKeys.self)
        let data = try outer.nestedContainer(keyedBy: DataKeys.self, forKey: .data)
        let user = try data.nestedContainer(keyedBy: UserKeys.self, forKey: .user)

        fullName = try user.decode(String.self, forKey: .fullName)
        age = try user.decode(Int.self, forKey: .age)
    }
}
```

## 흔한 오해와 팁

> ⚠️ **흔한 오해**: "`Codable`을 쓰면 JSON 키와 프로퍼티 이름이 반드시 같아야 한다" — 아닙니다! `CodingKeys` enum이나 `keyDecodingStrategy`로 자유롭게 매핑할 수 있습니다. 서버의 `snake_case`를 Swift의 `camelCase`로 변환하는 것은 아주 흔한 패턴이에요.

> 🔥 **실무 팁**: `JSONDecoder`와 `JSONEncoder`는 생성 비용이 있으므로, 매 요청마다 새로 만들기보다는 **재사용**하는 것이 좋습니다. 공통 설정을 적용한 디코더를 프로퍼티로 선언해두세요.

```swift
// 앱 전체에서 재사용하는 디코더
extension JSONDecoder {
    static let apiDecoder: JSONDecoder = {
        let decoder = JSONDecoder()
        decoder.keyDecodingStrategy = .convertFromSnakeCase
        decoder.dateDecodingStrategy = .iso8601
        return decoder
    }()
}

// 사용: try JSONDecoder.apiDecoder.decode(User.self, from: data)
```

> 💡 **알고 계셨나요?**: `Codable`은 JSON만을 위한 것이 아닙니다. `PropertyListDecoder`, `XMLDecoder`(서드파티) 등 다양한 인코더/디코더와 함께 사용할 수 있어요. `Codable`은 "직렬화 프로토콜"이고, `JSONDecoder`는 그중 하나의 구현일 뿐입니다.

## 핵심 정리

| 개념 | 설명 |
|------|------|
| `Codable` | `Encodable & Decodable`의 조합. 직렬화/역직렬화 지원 |
| `JSONDecoder` | JSON 데이터 → Swift 타입으로 변환 |
| `JSONEncoder` | Swift 타입 → JSON 데이터로 변환 |
| `CodingKeys` | JSON 키와 Swift 프로퍼티 이름 매핑 |
| `.convertFromSnakeCase` | `snake_case` → `camelCase` 자동 변환 |
| 옵셔널 프로퍼티 | JSON에 해당 키가 없어도 에러 없이 `nil` 할당 |
| `.iso8601` | ISO 8601 형식의 날짜 문자열을 `Date`로 변환 |
| `nestedContainer` | 중첩 JSON 구조를 평탄화 |

## 다음 섹션 미리보기

서버와 통신할 때 항상 성공만 하면 좋겠지만, 현실은 그렇지 않죠. 인터넷이 끊기고, 서버가 에러를 반환하고, JSON 형식이 예상과 다르기도 합니다. 다음 [04. 네트워크 에러 핸들링](./04-error-handling.md)에서 이런 상황을 우아하게 처리하는 방법을 배워봅시다.

## 참고 자료

- [Apple 공식 문서 - Codable](https://developer.apple.com/documentation/swift/codable) - Codable 프로토콜 레퍼런스
- [Apple 공식 문서 - JSONDecoder](https://developer.apple.com/documentation/foundation/jsondecoder) - JSONDecoder 전체 옵션
- [SE-0166: Swift Archival & Serialization](https://github.com/swiftlang/swift-evolution/blob/main/proposals/0166-swift-archival-serialization.md) - Codable 최초 프로포절
- [SE-0295: Codable synthesis for enums with associated values](https://github.com/swiftlang/swift-evolution/blob/main/proposals/0295-codable-synthesis-for-enums-with-associated-values.md) - enum Codable 확장
- [Hacking with Swift - Codable](https://www.hackingwithswift.com/articles/119/codable-cheat-sheet) - Codable 치트시트
