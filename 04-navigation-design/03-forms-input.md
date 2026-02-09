# 폼과 사용자 입력

> TextField, SecureField, Toggle, Picker, Slider, Form

## 개요

앱이 사용자와 소통하려면 **입력**을 받아야 합니다. 이름을 적고, 비밀번호를 넣고, 옵션을 선택하고, 값을 조절하는 것 — 이 모든 것이 폼(Form)이에요. 이 섹션에서는 SwiftUI가 제공하는 다양한 입력 컨트롤을 배우고, 설정 앱 같은 깔끔한 폼을 만들어봅니다.

**선수 지식**: [02. TabView와 모달](./02-tab-modal.md)에서 배운 sheet, 화면 구성
**학습 목표**:
- Form으로 설정 화면 스타일의 폼 만들기
- TextField, SecureField로 텍스트 입력 받기
- Toggle, Picker, Slider, Stepper로 값 선택하기
- @FocusState로 키보드 포커스 관리하기

## 왜 알아야 할까?

설정 앱을 열어보세요. Wi-Fi 켜기/끄기(Toggle), 화면 밝기(Slider), 언어 선택(Picker), 이름 입력(TextField)... 이 모든 컨트롤이 `Form` 안에 깔끔하게 정리되어 있죠. 회원가입, 프로필 편집, 앱 설정 등 사용자 입력이 필요한 거의 모든 화면에서 이 패턴을 사용합니다.

## 핵심 개념

### 개념 1: Form — 입력 컨트롤의 컨테이너

> 💡 **비유**: Form은 **신청서 양식**입니다. 여러 항목이 섹션별로 깔끔하게 정리되어 있고, 각 항목에 맞는 입력 방식(텍스트, 체크박스, 드롭다운 등)이 준비되어 있죠.

```swift
import SwiftUI

struct SettingsFormView: View {
    @State private var username = ""
    @State private var isNotificationOn = true
    @State private var fontSize = 14.0

    var body: some View {
        NavigationStack {
            // Form은 자동으로 그룹화된 리스트 스타일을 적용
            Form {
                // Section으로 항목을 그룹화
                Section("계정") {
                    TextField("사용자 이름", text: $username)
                }

                Section("알림") {
                    Toggle("푸시 알림", isOn: $isNotificationOn)
                }

                Section("화면") {
                    Slider(value: $fontSize, in: 10...30, step: 1) {
                        Text("글자 크기")
                    }
                    Text("현재 크기: \(Int(fontSize))pt")
                }
            }
            .navigationTitle("설정")
        }
    }
}

#Preview {
    SettingsFormView()
}
```

### 개념 2: TextField — 텍스트 입력

> 💡 **비유**: TextField는 **빈 칸이 있는 양식**이에요. "이름: ______" 처럼 빈 칸을 채워넣는 거죠.

```swift
import SwiftUI

struct TextFieldDemoView: View {
    @State private var name = ""
    @State private var bio = ""
    @State private var age: Int?

    var body: some View {
        Form {
            Section("기본 정보") {
                // 기본 텍스트 필드
                TextField("이름", text: $name)

                // 숫자 전용 입력 (format 파라미터)
                TextField("나이", value: $age, format: .number)
                    .keyboardType(.numberPad)

                // 여러 줄 입력 (axis: .vertical)
                TextField("자기소개", text: $bio, axis: .vertical)
                    .lineLimit(3...6)
            }

            Section("입력 타입별 키보드") {
                TextField("이메일", text: .constant(""))
                    // 이메일 키보드
                    .keyboardType(.emailAddress)
                    // 자동 대문자 끄기
                    .textInputAutocapitalization(.never)
                    // 자동완성 힌트
                    .textContentType(.emailAddress)

                TextField("웹사이트", text: .constant(""))
                    .keyboardType(.URL)
                    .textInputAutocapitalization(.never)
            }
        }
    }
}

#Preview {
    TextFieldDemoView()
}
```

### 개념 3: SecureField — 비밀번호 입력

비밀번호처럼 가려야 하는 텍스트를 입력받을 때 사용합니다. 입력한 문자가 점(●)으로 표시돼요.

```swift
import SwiftUI

struct LoginFormView: View {
    @State private var email = ""
    @State private var password = ""

    var body: some View {
        Form {
            Section("로그인") {
                TextField("이메일", text: $email)
                    .keyboardType(.emailAddress)
                    .textContentType(.emailAddress)
                    .textInputAutocapitalization(.never)

                // 비밀번호 필드: 입력 내용이 가려짐
                SecureField("비밀번호", text: $password)
                    .textContentType(.password)
            }

            Section {
                Button("로그인") {
                    print("로그인 시도: \(email)")
                }
                .frame(maxWidth: .infinity)
                .disabled(email.isEmpty || password.isEmpty)
            }
        }
    }
}

#Preview {
    LoginFormView()
}
```

### 개념 4: Toggle, Picker, Slider, Stepper

폼에서 자주 쓰는 선택/조절 컨트롤들을 한 번에 살펴볼까요?

```swift
import SwiftUI

struct ControlsDemoView: View {
    @State private var isDarkMode = false
    @State private var selectedColor = "파랑"
    @State private var volume = 50.0
    @State private var quantity = 1
    @State private var birthday = Date()

    let colors = ["빨강", "파랑", "초록", "노랑"]

    var body: some View {
        NavigationStack {
            Form {
                // Toggle: 켜기/끄기
                Section("디스플레이") {
                    Toggle("다크 모드", isOn: $isDarkMode)

                    // SF Symbol 포함 토글
                    Toggle("자동 밝기", systemImage: "sun.max.fill",
                           isOn: .constant(true))
                }

                // Picker: 여러 옵션 중 선택
                Section("테마 색상") {
                    // 메뉴 스타일 (기본)
                    Picker("색상", selection: $selectedColor) {
                        ForEach(colors, id: \.self) { color in
                            Text(color).tag(color)
                        }
                    }

                    // 세그먼트 스타일
                    Picker("색상 (세그먼트)", selection: $selectedColor) {
                        ForEach(colors, id: \.self) { color in
                            Text(color).tag(color)
                        }
                    }
                    .pickerStyle(.segmented)
                }

                // Slider: 연속 값 조절
                Section("소리") {
                    Slider(value: $volume, in: 0...100) {
                        Text("볼륨")
                    } minimumValueLabel: {
                        Image(systemName: "speaker.fill")
                    } maximumValueLabel: {
                        Image(systemName: "speaker.wave.3.fill")
                    }
                    Text("볼륨: \(Int(volume))%")
                }

                // Stepper: 정수 값 증감
                Section("수량") {
                    Stepper("수량: \(quantity)개", value: $quantity, in: 1...99)
                }

                // DatePicker: 날짜 선택
                Section("생일") {
                    DatePicker("생년월일", selection: $birthday,
                               displayedComponents: [.date])
                }
            }
            .navigationTitle("컨트롤 모음")
        }
    }
}

#Preview {
    ControlsDemoView()
}
```

### 개념 5: @FocusState — 키보드 포커스 관리

> 💡 **비유**: @FocusState는 **손전등**이에요. 여러 입력 필드 중에서 지금 빛을 비추고 있는(포커스된) 필드가 어디인지를 추적하고, 원하는 곳으로 빛을 옮길 수 있습니다.

여러 텍스트 필드가 있을 때, 리턴 키를 누르면 다음 필드로 자동 이동하게 하거나, "완료" 버튼으로 키보드를 닫을 수 있어요.

```swift
import SwiftUI

// 포커스할 필드를 열거형으로 정의
enum FormField: Hashable {
    case firstName, lastName, email
}

struct FocusDemoView: View {
    @State private var firstName = ""
    @State private var lastName = ""
    @State private var email = ""

    // 현재 포커스된 필드를 추적
    @FocusState private var focusedField: FormField?

    var body: some View {
        NavigationStack {
            Form {
                TextField("이름", text: $firstName)
                    .focused($focusedField, equals: .firstName)
                    // 리턴 키 라벨 변경
                    .submitLabel(.next)
                    // 리턴 키 누르면 다음 필드로
                    .onSubmit { focusedField = .lastName }

                TextField("성", text: $lastName)
                    .focused($focusedField, equals: .lastName)
                    .submitLabel(.next)
                    .onSubmit { focusedField = .email }

                TextField("이메일", text: $email)
                    .focused($focusedField, equals: .email)
                    .submitLabel(.done)
                    .onSubmit { focusedField = nil }
                    .keyboardType(.emailAddress)
            }
            .navigationTitle("회원 가입")
            // 키보드 위에 "완료" 버튼 추가
            .toolbar {
                ToolbarItemGroup(placement: .keyboard) {
                    Spacer()
                    Button("완료") {
                        focusedField = nil
                    }
                }
            }
            // 화면이 나타나면 첫 번째 필드에 포커스
            .onAppear {
                focusedField = .firstName
            }
        }
    }
}

#Preview {
    FocusDemoView()
}
```

> 🔥 **실무 팁**: `@FocusState`의 초기값을 직접 설정해도 작동하지 않습니다. 반드시 `.onAppear`에서 값을 설정해야 포커스가 올바르게 적용돼요.

## 실습: 직접 해보기

프로필 편집 폼을 만들어봅시다.

```swift
import SwiftUI

struct ProfileEditView: View {
    @State private var displayName = "홍길동"
    @State private var bio = "Swift를 배우는 중입니다!"
    @State private var isPublicProfile = true
    @State private var theme = "시스템"
    @State private var notificationLevel = 50.0

    let themes = ["라이트", "다크", "시스템"]

    @Environment(\.dismiss) var dismiss

    var body: some View {
        NavigationStack {
            Form {
                Section {
                    HStack {
                        Spacer()
                        Image(systemName: "person.circle.fill")
                            .font(.system(size: 80))
                            .foregroundStyle(.blue)
                        Spacer()
                    }
                    .listRowBackground(Color.clear)
                }

                Section("기본 정보") {
                    TextField("표시 이름", text: $displayName)
                    TextField("자기소개", text: $bio, axis: .vertical)
                        .lineLimit(2...4)
                }

                Section("개인정보") {
                    Toggle("공개 프로필", isOn: $isPublicProfile)
                }

                Section("외관") {
                    Picker("테마", selection: $theme) {
                        ForEach(themes, id: \.self) { t in
                            Text(t).tag(t)
                        }
                    }
                }

                Section("알림") {
                    Slider(value: $notificationLevel, in: 0...100) {
                        Text("알림 빈도")
                    }
                    Text("알림 빈도: \(Int(notificationLevel))%")
                        .font(.caption)
                        .foregroundStyle(.secondary)
                }
            }
            .navigationTitle("프로필 편집")
            .navigationBarTitleDisplayMode(.inline)
            .toolbar {
                ToolbarItem(placement: .cancellationAction) {
                    Button("취소") { dismiss() }
                }
                ToolbarItem(placement: .confirmationAction) {
                    Button("저장") { dismiss() }
                }
            }
        }
    }
}

#Preview {
    ProfileEditView()
}
```

## 더 깊이 알아보기

### iOS 26의 새로운 폼 기능

iOS 26에서는 폼 컨트롤에 몇 가지 흥미로운 변화가 있습니다:

- **TextEditor 리치 텍스트 지원**: `TextEditor`가 `AttributedString`을 바인딩할 수 있게 되어, 볼드, 이탤릭, 색상 등 서식 있는 텍스트 편집이 가능해졌어요. WWDC25에서 별도 세션을 할 만큼 큰 변화입니다.
- **Slider neutralValue**: 슬라이더에 "중립점"을 지정할 수 있게 되었어요. 예를 들어 재생 속도에서 1배속이 중립인 양방향 슬라이더를 만들 수 있습니다.
- **Liquid Glass 자동 적용**: Toggle, Slider, Picker 등 모든 폼 컨트롤이 자동으로 Liquid Glass 디자인을 적용받습니다.

### 폼의 기원

UI 폼의 개념은 종이 양식에서 시작됐습니다. Apple은 iOS 설정 앱에서 `UITableView`의 grouped 스타일을 사용해 폼 패턴을 확립했는데, SwiftUI에서는 이를 `Form`이라는 선언적 컨테이너로 깔끔하게 추상화했죠. `Form`은 플랫폼에 따라 자동으로 다른 스타일을 적용하는 똑똑한 컨테이너입니다 — iOS에서는 grouped list, macOS에서는 정렬된 레이블+필드 레이아웃으로 나타나요.

## 흔한 오해와 팁

> ⚠️ **흔한 오해**: "TextField의 format 파라미터로 실시간 포맷팅이 되나요?" — format은 **포커스를 잃을 때**(commit 시)에만 적용됩니다. 타이핑 중에는 원본 텍스트가 보여요.

> 🔥 **실무 팁**: 폼에서 키보드가 가리는 문제가 있다면, `.scrollDismissesKeyboard(.interactively)`를 추가하세요. 스크롤하면 키보드가 자연스럽게 내려갑니다.

> 💡 **알고 계셨나요?**: Picker에는 6가지 이상의 스타일이 있어요. `.menu`(드롭다운), `.wheel`(휠), `.segmented`(세그먼트), `.inline`(인라인), `.navigationLink`(새 화면) 등 상황에 맞게 선택하세요.

## 핵심 정리

| 개념 | 설명 |
|------|------|
| Form | 입력 컨트롤을 그룹화하는 컨테이너 |
| Section | 폼 내부 항목을 섹션으로 구분 |
| TextField | 텍스트 입력 (format으로 숫자/통화 입력도 가능) |
| SecureField | 비밀번호 등 가려지는 텍스트 입력 |
| Toggle | 켜기/끄기 스위치 |
| Picker | 여러 옵션 중 하나를 선택 |
| Slider | 연속 값을 드래그로 조절 |
| Stepper | 정수 값을 +/- 버튼으로 조절 |
| DatePicker | 날짜/시간 선택 |
| @FocusState | 키보드 포커스 추적 및 제어 |

## 다음 섹션 미리보기

폼의 각 컨트롤 옆에 예쁜 아이콘을 보신 적 있죠? 다음 섹션에서는 Apple이 제공하는 6,000개 이상의 무료 아이콘 **SF Symbols**와 에셋 관리법을 배워봅니다. [04. SF Symbols과 에셋 관리](./04-sf-symbols.md)에서 만나요!

## 참고 자료

- [Form - Apple Developer Documentation](https://developer.apple.com/documentation/swiftui/form) - Form 공식 API 문서
- [TextField - Apple Developer Documentation](https://developer.apple.com/documentation/swiftui/textfield) - TextField의 모든 이니셜라이저와 수정자
- [Cook up a rich text experience in SwiftUI - WWDC25](https://developer.apple.com/videos/play/wwdc2025/280/) - iOS 26 TextEditor 리치 텍스트 세션
- [SwiftUI TextField Advanced - fatbobman](https://fatbobman.com/en/posts/textfield-1/) - TextField format과 검증의 심화 가이드
