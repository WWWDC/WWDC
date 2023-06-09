# Meet TestFlight on Mac

## [Meet TestFlight on Mac](https://developer.apple.com/videos/play/wwdc2021/10170)

\
\


## 🎯 이게뭔데?

TestFlight를 사용하면 앱의 베타 빌드를 테스터에게 배포하고 피드백을 수집할 수 있습니다.

Mac의 TestFlight는 iOS 및 tvOS에서와 동일한 모든 기능을 제공합니다.

* 베타 앱을 설치하고
* 자동 업데이트를 설정하고
* 피드백을 공유할 수 있습니다.

\
\


## 🎯 주요 기능 모음

### 🌀 테스터 모집

이메일 초대 또는 공개 링크를 공유해서 테스터를 모집한 후,

테스터가 위 화면에서 초대를 수락하면 앱을 설치할 수 있게 됩니다.

버전별로 그룹화된 빌드를 중 특정 빌드를 선택하고 설치하여 Mac에서 테스트를 시작할 수 있습니다.

### 🌀 베타 앱 설치

앱을 설치하고나면 Dock, Launchpad 및 Finder와 같은 모든 방법으로 시작할 수 있고,

베타 앱임을 쉽게 인식할 수 있도록 각각 앱 이름 옆에 노란색 점을 표시하여 구분합니다.

테스터는 사용 가능한 최신 빌드가 자동으로 설치되도록 자동 업데이트를 구성할 수 있습니다.

이렇게 하면 앱의 최신 빌드를 테스트할 수 있습니다.

### 🌀 사용자 피드백

테스터는 경험한 문제에 대한 피드백을 보내거나 개선 사항을 제안할 수 있습니다.

스크린샷을 첨부하여 피드백을 보낼 수 있습니다.

### 🌀 충돌 로그 캡쳐

베타 앱이 충돌하는 경우 TestFlight는 자동으로 충돌 로그를 캡처하고 테스터가 추가 설명을 입력할 수 있는 알림창을 제공합니다.

충돌 로그를 다운로드하고 App Store Connect 충돌 피드백 섹션과 Xcode Organizer에서 피드백을 볼 수 있습니다.

\
\


## 🎯 Xcode Cloud 사용 방법

1. Apple Silicon Mac의 기본 Mac 앱 TestFlight용으로 설정
2. iOS 앱 TestFlight용으로 설정
3. 내부 그룹 관리
4. 내장된 Xcode Cloud 기능을 사용하는 방법

에 대해서 알아봅시다!

\
\## 🌀 TestFlight로 네이티브 Mac 앱을 배포하는 방법

먼저 TestFlight와 함께 배포할 프로비저닝 프로필이 필요합니다.

이 서명에 대해 `자동으로 관리하기` or `수동으로 관리하기`중 선택할 수 있습니다.

자동으로 관리하기를 체크하면 앱 빌드시 Xcode에서 앱에 대한 프로비저닝 프로필을 생성하고 포함합니다.

수동 관리시에는 개발자 포털을 참고하여 프로필을 명시적으로 추가하는 방식으로 진행하면 됩니다.

프로비저닝 프로필과 함께 업로드되면 빌드가 macOS 아래에 표시됩니다.

빌드가 완료되면 **App Store Connect**에서 그룹을 만들어 테스터를 관리하고 배포할 수 있습니다.

각 빌드에 대해 초대된 테스터 수, 장치에 설치된 횟수, 지난 7일 동안의 세션, 충돌 및 피드백 수를 볼 수 있습니다.

macOS를 필터링 하여 iOS와 구분되게 충돌 또는 스크린샷 피드백 섹션을 보거나

특정 Mac 장치 또는 MacOS 버전을 선택하여 피드백을 추가로 필터링할 수 있습니다.

\
\


### 🌀 Apple Silicon Mac의 iOS 앱을 배포하는 방법 🥲

각 테스터 그룹에 대해 Apple Silicon Mac에서 iPhone 및 iPad 앱의 TestFlight 가용성을 제어할 수 있습니다.

활성화되면 이 그룹의 iOS 빌드를 Apple Silicon Mac의 TestFlight에서 사용할 수 있습니다.

이것이 비활성화되면 해당 그룹의 모든 iOS 빌드는 Mac에서 더 이상 사용할 수 없습니다.

이렇게 하면 Apple Silicon Mac에서 iOS 앱을 테스트할 수 있는 사람을 더 유연하게 제어할 수 있습니다.

iOS 빌드는 현재와 유사하게 표시되지만 표시되는 개수에는 Apple Silicon Mac의 숫자도 포함됩니다.

마찬가지로 iOS 플랫폼의 App Store Connect에서 충돌 및 스크린샷 피드백을 볼 수 있으며 Apple Silicon Mac에서 제출된 피드백이 포함됩니다.

\


#### → 예 … ? 🥹

이 부분은 해보기 전까지 정확하게 파악할 수 없을 것 같습니다...

\
\## 🌀 여러 내부 그룹 만들기

내부 그룹별로 빌드 배포 및 피드백 수집을 구성할 수 있습니다.

예를 들어 개발팀과 QA팀에 대해 각각 내부 그룹을 만들어 접근을 제한할 수 있습니다.

개발 수명 주기의 일부로 개발 팀에 모든 빌드에 대한 액세스 권한을 부여할 수 있지만 QA 팀에는 특정 안정 빌드에 대한 액세스 권한만 필요할 수 있습니다.

개발 팀의 경우 자동 배포 플래그를 활성화하여 이 그룹에서 모든 현재 및 향후 빌드를 사용할 수 있도록 하는 반면, QA 팀의 경우 특정 빌드를 수동으로 추가하도록 선택합니다.

1. Internal Testing 옆에 있는 더하기 버튼을 클릭
2. 내부 그룹에 이름 지정하고 생성 개발팀 : 자동 배포 활성화 체크 QA 팀 : 자동 배포 활성화 체크 해제
3. QA 팀 그룹에 추가할 빌드를 수동으로 선택
4. 그룹별로 피드백을 활성화하거나 비활성화하는 옵션 설정
5. iOS에서 App Store Connect를 사용하는 경우

\
\## 🌀 TestFlight에 Xcode Cloud 기능 통합

Xcode Cloud는 앱을 자동으로 빌드, 테스트 및 배포할 수 있는 원활한 환경을 제공합니다.

* 첫째, Xcode Cloud는 TestFlight를 통해 베타 빌드 배포를 관리하여 빌드를 특정 베타 그룹에 자동으로 배포하여 원활한 경험을 제공합니다.
* 둘째, 빌드 그룹을 통해 개발 팀이 작업하는 방식으로 빌드가 구성됩니다.

Xcode Cloud 워크플로 및 GIT 분기별로 그룹화된 빌드를 탐색하여 내부 테스터가 빌드를 더 쉽게 찾을 수 있습니다. ( 그룹별 피드백도 필터링 가능 )

\
\


## 🎯 요약하자면

1. Mac의 TestFlight는 네이티브 Mac 앱과 Apple Silicon Mac의 iOS 앱을 모두 지원합니다.
2. 내부 그룹 관리를 개선하여 필요에 따라 여러 그룹을 만듭니다.
3. Xcode Cloud는 App Store Connect 및 TestFlight와 통합되어 빌드 배포를 관리하고 빌드 그룹별로 구성합니다.
