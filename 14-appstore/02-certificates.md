# 인증서와 프로비저닝

> 개발자 계정, Certificate, Provisioning Profile, Code Signing

## 개요

앱을 실제 iPhone에 설치하려면 "이 앱은 신뢰할 수 있는 개발자가 만들었습니다"라는 **디지털 서명**이 필요합니다. 인증서(Certificate), 프로비저닝 프로파일(Provisioning Profile), 코드 서명(Code Signing)은 이 신뢰 체계의 핵심 요소인데요, 처음에는 복잡하게 느껴지지만 한번 이해하면 간단합니다.

**선수 지식**: [앱 아이콘과 스크린샷](./01-app-icon-assets.md)
**학습 목표**:
- Apple Developer Program의 구성과 가입 과정을 이해할 수 있다
- 인증서와 프로비저닝 프로파일의 관계를 설명할 수 있다
- Xcode 자동 서명으로 코드 서명을 설정할 수 있다

## 왜 알아야 할까?

"No matching provisioning profile found" — iOS 개발자라면 한 번쯤 만나는 에러입니다. 이 에러의 의미를 모르면 구글링에 시간만 낭비하게 되죠. 인증서와 프로비저닝의 구조를 이해하면 이런 문제를 5분 안에 해결할 수 있고, 팀 개발 환경에서도 자신 있게 설정할 수 있습니다.

## 핵심 개념

### 개념 1: Apple Developer Program — 출발점

> 💡 **비유**: Apple Developer Program은 **운전면허**입니다. 면허가 없으면 도로(App Store)에 나갈 수 없죠. 연간 비용을 내고 갱신해야 하는 것도 비슷합니다.

| 항목 | 내용 |
|------|------|
| 연간 비용 | $99 (한화 약 13만원) |
| 개인 vs 조직 | 개인은 본인 이름, 조직은 D-U-N-S 번호 필요 |
| 포함 사항 | App Store 배포, TestFlight, App Store Connect, Xcode Cloud (25시간/월) |
| 무료 계정 | 실제 기기 테스트 가능, App Store 배포 불가 |

> 🔥 **실무 팁**: 개인 개발자도 회사 이름으로 출시하고 싶다면 **조직(Organization)** 계정을 만드세요. 나중에 개인에서 조직으로 변경하는 것은 불가능하니, 처음부터 결정하는 게 좋습니다.

### 개념 2: 인증서(Certificate) — 디지털 신분증

> 💡 **비유**: 인증서는 **공인인증서**와 같습니다. 공개키(은행에 등록)와 개인키(내 컴퓨터에 보관)의 쌍으로 "이 사람이 진짜 이 개발자입니다"를 증명하죠.

인증서는 **공개키/개인키 쌍**으로 작동합니다.

| 인증서 종류 | 용도 | 유효 기간 |
|-------------|------|-----------|
| Development | 개발 중 실기기 테스트 | 1년 |
| Distribution | App Store 배포용 | 1년 |
| Developer ID | Mac 앱 직접 배포 | 5년 |

**인증서 생성 과정 (자동 서명 시):**

1. Xcode가 Keychain Access에서 **개인키**를 생성
2. 개인키로 **CSR(Certificate Signing Request)** 생성
3. CSR을 Apple Developer Portal에 전송
4. Apple이 **인증서** 발급 (공개키 포함)
5. Xcode가 자동으로 다운로드 및 설치

> ⚠️ **흔한 오해**: "인증서는 여러 Mac에서 공유할 수 없다" — 가능합니다! Keychain Access에서 인증서와 개인키를 `.p12` 파일로 내보내면 다른 Mac에서도 사용할 수 있어요. 팀 개발 시 필수 지식입니다.

### 개념 3: 프로비저닝 프로파일(Provisioning Profile) — 통행증

> 💡 **비유**: 프로비저닝 프로파일은 **건물 출입증**입니다. "이 사람(인증서)이 이 건물(앱)에 이 문(기기)으로 들어갈 수 있다"는 세 가지 정보를 묶어놓은 거죠.

프로비저닝 프로파일은 **3가지 정보**를 연결합니다.

- **누가**: 인증서 (개발자 신원)
- **무엇을**: App ID + Bundle Identifier (어떤 앱)
- **어디서**: 등록된 기기 UDID (어떤 기기, Development/Ad Hoc만 해당)

| 프로파일 종류 | 기기 제한 | 용도 |
|--------------|----------|------|
| Development | 등록된 기기만 (100대/종류) | 개발 중 테스트 |
| Ad Hoc | 등록된 기기만 (100대/종류) | 사내 배포, 제한적 테스트 |
| App Store | 제한 없음 | App Store 배포 (심사 통과 필요) |
| Enterprise | 제한 없음 | 기업 내부 배포 ($299/년 별도) |

### 개념 4: 코드 서명(Code Signing) — 자동이 답입니다

Xcode의 **Automatically manage signing** 옵션을 켜면, 위의 모든 과정을 Xcode가 알아서 처리합니다.

```swift
// Xcode 프로젝트 설정에서 확인할 내용
// 1. Signing & Capabilities 탭 열기
// 2. "Automatically manage signing" 체크
// 3. Team 선택 (Apple Developer 계정)
// 4. Bundle Identifier 설정

// Bundle Identifier 예시:
// com.yourcompany.yourapp
// - 역도메인 표기법 사용
// - 한번 설정하면 변경 불가
// - 앱의 고유 식별자로 영원히 사용됨
```

**Capabilities(기능) 추가 시:**

앱에서 Push Notifications, Sign in with Apple, HealthKit 등을 사용하려면 **Capabilities** 탭에서 해당 기능을 추가해야 합니다. 이때 App ID에 해당 Entitlement가 자동으로 등록되고, 프로비저닝 프로파일이 갱신됩니다.

### 개념 5: 흔한 문제와 해결법

| 에러 메시지 | 원인 | 해결 방법 |
|------------|------|-----------|
| No matching provisioning profile | 프로파일 만료 또는 누락 | Xcode에서 자동 서명 재설정 |
| Certificate not found | 다른 Mac에서 작업 시 개인키 없음 | `.p12` 파일 가져오기 또는 인증서 재생성 |
| Untrusted Developer | 기기에서 개발자 신뢰 안 함 | 설정 → 일반 → VPN 및 기기 관리에서 신뢰 |
| Provisioning profile expired | 1년 유효기간 만료 | Developer Portal에서 갱신 후 다운로드 |

## 실습: 직접 해보기

Xcode에서 자동 서명을 설정하는 단계별 가이드입니다.

**자동 서명 설정 체크리스트:**

- [ ] Apple Developer Program 가입 확인 ($99/년)
- [ ] Xcode → Preferences → Accounts에서 Apple ID 추가
- [ ] 프로젝트 → Signing & Capabilities → Team 선택
- [ ] "Automatically manage signing" 체크 확인
- [ ] Bundle Identifier를 고유값으로 설정 (예: `com.yourname.appname`)
- [ ] 실제 기기 연결 후 빌드 → 신뢰 설정 확인

## 더 깊이 알아보기

Apple이 코드 서명을 요구하는 이유는 **보안 모델** 때문입니다. 2008년 App Store가 처음 열렸을 때, Apple은 "모든 앱은 신뢰할 수 있는 개발자가 만들어야 한다"는 원칙을 세웠어요. 이 원칙 덕분에 iOS는 데스크톱 OS에 비해 악성코드 문제가 현저히 적습니다.

초기에는 인증서와 프로비저닝 프로파일을 모두 수동으로 관리해야 했습니다. 개발자 포털에서 CSR 파일을 업로드하고, 기기 UDID를 하나하나 등록하고, 프로파일을 다운로드해서 Xcode에 설치하는 과정이 필요했죠. 이 복잡한 과정은 "provisioning hell"이라 불릴 정도였습니다. Xcode 8(2016년)부터 자동 서명이 도입되면서 이 고통이 크게 줄었고, 지금은 대부분의 경우 체크박스 하나로 해결됩니다.

## 흔한 오해와 팁

> ⚠️ **흔한 오해**: "무료 Apple ID로는 실기기 테스트가 안 된다" — 가능합니다! 무료 계정으로도 실기기에 앱을 설치할 수 있어요. 다만 7일마다 재설치해야 하고, 사용할 수 있는 Capability가 제한됩니다. App Store 배포만 유료 계정이 필요합니다.

> 💡 **알고 계셨나요?**: Xcode Cloud를 사용하면 인증서와 프로비저닝 프로파일을 클라우드에서 자동 관리합니다. CI/CD 파이프라인에서 "인증서를 어디에 보관할까"라는 고민을 해결해주죠.

## 핵심 정리

| 개념 | 설명 |
|------|------|
| Apple Developer Program | 연 $99, App Store 배포에 필수 |
| Certificate | 공개키/개인키 쌍으로 개발자 신원 증명 |
| Provisioning Profile | 인증서 + App ID + 기기 UDID를 연결한 통행증 |
| Code Signing | 앱 바이너리에 디지털 서명을 붙이는 과정 |
| Bundle Identifier | 앱의 고유 식별자, 한번 설정하면 변경 불가 |
| Automatic Signing | Xcode가 인증서/프로파일을 자동 관리 |

## 다음 섹션 미리보기

인증서와 프로비저닝으로 앱에 서명을 했으니, 이제 App Store에 앱을 등록할 차례입니다. [App Store Connect](./03-app-store-connect.md)에서 앱 메타데이터 작성, 가격 설정, TestFlight 베타 테스트까지 배워봅시다.

## 참고 자료

- [Apple Developer Program 가입](https://developer.apple.com/programs/) - 개발자 프로그램 공식 페이지
- [Code Signing Guide - Apple Developer](https://developer.apple.com/support/code-signing/) - 코드 서명 공식 가이드
- [Xcode Cloud Overview](https://developer.apple.com/xcode-cloud/) - CI/CD 인증서 자동 관리
- [iOS Code Signing Explained - Updraft](https://getupdraft.com/blog/ios-code-signing-development-and-distribution-prov) - 인증서/프로비저닝 심층 해설
